# Data Flow Architecture

**Version:** 0.1.0
**Last Updated:** April 2026
**Status:** Draft
**Classification:** Private/Internal

---

## 1. Scope and Non-goals

This document is the depth layer below the Data Flow overview in [unified-platform-architecture.md §Data Flow Architecture](./unified-platform-architecture.md). It governs:

- End-to-end payload composition from biological tissue to the SNN substrate and back
- Encoding, compression, and serialization at each hop
- Transport protocols and link formats per segment
- QoS classes, scheduling, flow control, and failure handling
- The digital/analog boundary where the thermodynamic substrate enters the data plane

Out of scope. Latency budgets are maintained in [latency-budget-analysis.md](./latency-budget-analysis.md). Security properties of the data plane are maintained in [security-architecture.md](./security-architecture.md). Formal message shapes are maintained in [integration/api-contracts.yaml](../integration/api-contracts.yaml) and the per-pair integration specs under [integration/](../integration/).

---

## 2. Layer Model

```
┌────────────────────────────────────────────────────────────────────┐
│                         DATA PLANE LAYERS                          │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  L0  Biological tissue                                             │
│      ▲ action potentials (analog)                                  │
│  L1  CMOS micro-electrode array                                    │
│      ▲ on-chip ADC, spike detection, feature extraction            │
│  L2  Sub-cranial transceiver (UWB hub)                             │
│      ▲ framing, integrity, AES-256 envelope                        │
│  L3  Edge node (Neural Firewall)                                   │
│      ▲ anomaly detection, key rotation, safety filter              │
│  L4  Ground segment (RF / optical uplink)                          │
│      ▲ LEO link layer, forward error correction                    │
│  L5  Space segment (OISL mesh)                                     │
│      ▲ inter-satellite routing, state migration                    │
│  L6  SNN payload (neuromorphic + TC substrate)                     │
│      ▲ LIF inference, predictive coding, Langevin relaxation       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

The return path is the mirror of the forward path. Stimulation commands re-enter at L6 and are expanded at L1 into electrode drive patterns.

---

## 3. Signal Chain per Direction

### 3.1 Forward path (biological to synthetic)

| Hop | Transform | Output format |
|---|---|---|
| L0 → L1 | Analog action potential to digital spike event | `SpikeEvent { neuron_id, timestamp_offset_us, amplitude? }` (see [api-contracts.yaml](../integration/api-contracts.yaml), `spike_event`) |
| L1 → L2 | Feature extraction, LZ4 compression (ratio 2:1 to 5:1) | `SpikeStreamUplink` protobuf |
| L2 → L3 | UWB framing, CRC32 integrity | Encrypted UWB frame |
| L3 → L4 | Neural Firewall encrypt (AES-256-GCM, HKDF from ECDH), sequence numbering | Encrypted WebSocket binary |
| L4 → L5 | RF or optical uplink to LEO, forward error correction | Link layer frames |
| L5 → L6 | OISL-Neural-v1 routing to target payload | `SpikeStreamInterSatellite` protobuf |
| L6 | LIF inference, predictive coding, optional TC relaxation | SNN state update |

### 3.2 Reverse path (synthetic to biological)

| Hop | Transform | Output format |
|---|---|---|
| L6 → L5 | SNN output to stimulation plan, safety signing (HMAC-SHA256) | `StimulationCommandDownlink` protobuf with `SafetyChecksum` |
| L5 → L4 | OISL egress, downlink scheduling | Link layer frames |
| L4 → L3 | Decrypt, verify `safety_signature`, enforce stimulation limits | Validated command stream |
| L3 → L2 | UWB frame, integrity check | Encrypted UWB frame |
| L2 → L1 | Decode `StimulationEvent` to electrode drive (waveform, amplitude, duration) | Electrode command |
| L1 → L0 | Current injection through CMOS array | Biphasic or charge-balanced waveform |

### 3.3 Stage responsibilities

| Stage | Compresses | Encrypts | Encodes | Rewrites |
|---|---|---|---|---|
| L1 CMOS | Yes (feature extraction) | No | Spike event | No |
| L2 UWB hub | No | Envelope | No | No |
| L3 Firewall | No (already compressed) | Yes (AES-256-GCM) | No | Safety filter may drop |
| L4 Ground | No | Transport layer (TLS 1.3 control plane only) | FEC | No |
| L5 OISL | No | No (link stays inside trust zone) | Packet priority classes | Route only |
| L6 SNN | No | No | Spike time to LIF state | Inference rewrites |

---

## 4. Spike Encoding Schemes

Three encodings coexist, chosen per channel:

- **Rate coding.** Per-neuron firing rate over a 1 to 10 ms window. Used for sensory integration channels where precise spike timing is not required. Tolerant to packet loss and jitter.
- **Temporal coding.** Exact spike timestamps at microsecond resolution. Used for motor channels and any path where the 350 ms Libet buffer is still the ceiling but phase information carries load. See [technical-glossary.md](../glossary/technical-glossary.md) for the term anchor.
- **Population coding.** Ensemble activity vector across a defined neuron set. Used for predictive coding residuals and for the digital/analog boundary with the TC substrate (section 10).

Rate coding is the default for OISL inter-satellite transit because it survives per-hop jitter. Temporal coding is preserved end-to-end on motor paths and reconstructed at L1 from timestamp offsets.

---

## 5. Predictive-coding Bandwidth Reduction

The Dream Engine (generative model, resident at L3 and L6) maintains a shared predictive model. Only residuals between the prediction and the observed signal cross the network. Expected reduction, per [unified-platform-architecture.md §Bandwidth Requirements](./unified-platform-architecture.md):

| Channel | Raw rate | After predictive coding |
|---|---|---|
| CMOS to transceiver (spikes) | 2 to 20 Gbps | unchanged (on-chip, no network hop) |
| Transceiver to edge | 100 Mbps to 1 Gbps | 10 to 100 Mbps (10x typical) |
| Edge to satellite | 1 to 10 Gbps | 500 Mbps to 1 Gbps (observed target) |
| Inter-satellite neural state | 60+ Gbps | state deltas dominate steady state |

The predictive model drifts if residuals stop flowing. Heartbeats at 1 Hz (per [api-contracts.yaml](../integration/api-contracts.yaml) `websocket_api`) double as model-sync beacons.

---

## 6. Protocol Stack per Segment

| Segment | Physical | Link | Transport | Session/App |
|---|---|---|---|---|
| L1 intra-array | CMOS metal | On-chip bus | N/A | Vendor DMA |
| L1 to L2 | Bonded serial | Serializer | N/A | Vendor DMA |
| L2 to L3 (UWB) | IEEE 802.15.4z | UWB MAC | UDP-like | AES-256-GCM envelope |
| L3 to L4 (edge to ground) | Fiber or 6G | Ethernet / radio | QUIC or WebSocket over TLS 1.3 | `SpikeStreamUplink` / `StimulationCommandDownlink` |
| L4 to L5 (ground to LEO) | Ka-band RF or 1550-nm optical | CCSDS AOS frames | OISL-Neural-v1 | Neural stream |
| L5 to L5 (OISL) | 1550-nm optical | CCSDS AOS | OISL-Neural-v1 | `SpikeStreamInterSatellite` |
| L5 to L6 (payload bus) | Internal SpaceWire or PCIe-rad | Vendor link | DMA | Vendor driver |
| Control plane | Any IP path | Ethernet / radio | TLS 1.3 `TLS_AES_256_GCM_SHA384` | REST (`https://edge.arkspace.me/api/v1`) |

Formal message bindings live in [integration/api-contracts.yaml](../integration/api-contracts.yaml). Version lock is `OISL-Neural-v1`; breaking changes require a major version bump and a 6-month deprecation window.

---

## 7. Data Formats

| Channel | Format | Compression | Integrity | Reference |
|---|---|---|---|---|
| Spike stream uplink | Protobuf (proto3) | LZ4 (realtime) | CRC32 | [arkspace-mindtransfer-interface.md](../integration/arkspace-mindtransfer-interface.md) |
| Stimulation command | Protobuf (proto3) | None | HMAC-SHA256 + CRC32 | same |
| SNN state transfer | Protobuf (proto3) | ZSTD (bulk) | SHA-256 | [arkspace-consciousness-interface.md](../integration/arkspace-consciousness-interface.md) |
| Inter-satellite spikes | Protobuf (proto3) | LZ4 | CRC32 | same |
| Control plane (REST) | JSON | None | TLS record integrity | [api-contracts.yaml](../integration/api-contracts.yaml) |
| Heartbeat / alert | JSON | None | TLS record integrity | same |
| TC parameter delta | Binary (vendor) | delta-only | Cryptographic signature | [thermodynamic-arkspace-interface.md](../integration/thermodynamic-arkspace-interface.md) |

Timestamps are unsigned 64-bit nanoseconds since the GPS epoch at every layer above L1. Sequence numbers are 32-bit and wrap every 2³² packets.

---

## 8. QoS, Prioritization, Flow Control, Backpressure

### 8.1 Priority classes

Matches the `PriorityLevel` enum in [api-contracts.yaml](../integration/api-contracts.yaml).

| Class | Use | Per-hop target | Drop policy |
|---|---|---|---|
| CRITICAL | Safety, kill switch, emergency stop | 5 ms | never drop; preempt lower classes |
| HIGH | Motor, stimulation commands | 10 ms | never drop; retry up to sequence wrap |
| NORMAL | Sensory, predictive coding residuals | 20 ms | head-of-line drop before enqueue tail |
| BULK | State migration, telemetry | best effort | tail drop first under congestion |

### 8.2 Queueing discipline

- Strict priority between classes. Weighted fair queueing within each class.
- Per-class queue depth caps prevent bulk traffic from starving predictive-coding residuals when a state migration coincides with an active session.
- Emergency stop bypasses all queues via a dedicated control channel with a response time requirement of 100 ms (per [api-contracts.yaml](../integration/api-contracts.yaml), `emergency_stop.response_time_requirement_ms`).

### 8.3 Flow control

- End-to-end. The WebSocket client reports back-pressure via control messages. The L3 Firewall enforces ingress rate limits per session.
- Per hop. OISL router uses CCSDS flow control credits. Runaway senders are rate-clamped by the ingress policer.

### 8.4 Backpressure behavior

When the edge cannot transmit at the requested rate, the Dream Engine falls back to local prediction. This preserves the motor loop at degraded fidelity rather than stalling the session. The degradation signal is emitted as a `WARNING` alert (per [api-contracts.yaml](../integration/api-contracts.yaml), `websocket_api.alert`).

---

## 9. Failure Modes

| Failure | Detection | Effect on neural data | Mitigation |
|---|---|---|---|
| Link loss (UWB, ground, OISL) | Heartbeat miss, sequence gap | Session bifurcates from satellite state | Local predictive cache at edge serves the motor loop until failover completes within 1 s (per [api-contracts.yaml](../integration/api-contracts.yaml), `performance_sla.failover_time_seconds`) |
| Satellite handover | Scheduled orbit transition | Brief RTT spike, possible sequence gap | Pre-warm target payload with last-known SNN state migration; predictive cache absorbs the transient |
| Dropped packets | CRC32 mismatch, sequence gap | Lost spikes reconstructed by predictive model | Sequence-window replay only for CRITICAL/HIGH; NORMAL/BULK drop without retry |
| Out-of-order spikes | Timestamp inversion within window | LIF state corruption if replayed | Reorder buffer at L6 with ≤5 ms horizon; drops stale spikes |
| Clock drift | GPS timestamp skew >1 ms | Spike timing distortion | All nodes discipline to GPS; drift alarms at 500 µs |
| Compression failure | Decoder reject | Packet loss | Fall back to uncompressed; raise alert |
| Safety signature mismatch | HMAC verify fail at L3 | Stimulation command dropped | Kill switch triggered if rate exceeds threshold ([api-contracts.yaml](../integration/api-contracts.yaml), `firewall_api.activate_kill_switch`) |

The neural-level meaning of each failure (missed spike, jitter-induced phase slip, seizure risk under stimulation timing errors) is tracked in [latency-budget-analysis.md](./latency-budget-analysis.md) and [security-architecture.md](./security-architecture.md).

---

## 10. Thermodynamic Substrate in the Data Plane

The TC substrate operates in continuous analog variables. The data plane meets it at a well-defined digital/analog boundary.

```
┌────────────────────────────────────────────────────────────┐
│                   L6 PAYLOAD DETAIL                        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  [Spike packet]                                            │
│        │                                                   │
│        ▼                                                   │
│  ┌──────────────────┐                                      │
│  │ Digital          │  bias vectors b_enc, b_dec           │
│  │ Conditioning     │─────────────────┐                    │
│  │ (ARM / RISC-V)   │                 │                    │
│  └──────────────────┘                 ▼                    │
│                              ┌───────────────────┐         │
│                              │   TC Substrate     │         │
│                              │   p-bit / Langevin │         │
│                              └────────┬──────────┘         │
│                                       │ equilibrium        │
│                                       │ sample             │
│                                       ▼                    │
│  ┌──────────────────┐       ┌───────────────────┐          │
│  │  Spike encoder   │ ◄──── │  Sample decoder    │          │
│  │  (rate or temp)  │       │  (Boltzmann to     │          │
│  └────────┬─────────┘       │   spike events)    │          │
│           │                 └───────────────────┘          │
│           ▼                                                │
│     [OISL egress]                                          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

Contract summary (full detail in [thermodynamic-consciousness-interface.md](../integration/thermodynamic-consciousness-interface.md) and [thermodynamic-arkspace-interface.md](../integration/thermodynamic-arkspace-interface.md)):

| Property | Value |
|---|---|
| Ingress format | Digital bias vectors computed from incoming spikes, <5 ms conditioning compute |
| Substrate state | Continuous Langevin variable $x_i$; no packet semantics |
| Egress format | Boltzmann sample (binary vector) or continuous trajectory, converted to spike events per the [thermodynamic-arkspace-interface.md](../integration/thermodynamic-arkspace-interface.md) conversion table |
| Relaxation time | <20 ms per DTM step; <50 ms per inference chain |
| Parameter updates | Ground-computed $\theta$ delta, AES-256 encrypted, signed, written to NVM |

The substrate is addressable only through the digital conditioning processor. Nothing in the data plane ever talks to analog p-bits directly.

---

## 11. Cross-references

| Segment | Authoritative contract |
|---|---|
| L0 to L2 | [mindtransfer-consciousness-interface.md](../integration/mindtransfer-consciousness-interface.md) |
| L2 to L5 (user plane) | [arkspace-mindtransfer-interface.md](../integration/arkspace-mindtransfer-interface.md) |
| L5 to L6 (SNN control and state) | [arkspace-consciousness-interface.md](../integration/arkspace-consciousness-interface.md) |
| L6 substrate interior (digital to analog) | [thermodynamic-consciousness-interface.md](../integration/thermodynamic-consciousness-interface.md) |
| L6 payload power, thermal, radiation | [thermodynamic-arkspace-interface.md](../integration/thermodynamic-arkspace-interface.md) |
| All message shapes | [api-contracts.yaml](../integration/api-contracts.yaml) |

---

## 12. Open Questions

1. Spike serialization stability. `SpikeStreamUplink` currently makes `amplitude` optional. A decision is needed on whether amplitude is used anywhere downstream of L2; if not it should be removed to save 4 bytes per spike at 10 to 100 kpps.
2. OISL-Neural-v1 versioning. Major bumps require a 6-month deprecation window. No agreed mechanism yet for running v1 and v2 payloads side by side during cutover.
3. Multi-hemisphere fan-out. The current model assumes one satellite hemisphere paired with one biological hemisphere. Multi-payload mirroring (redundancy) would add a fan-out point at L4 or L5 and is not specified.
4. TC egress format. The conversion from Boltzmann sample to spike event list is documented but not profiled. Throughput at the rate encoder under sustained 100 Mbps sample rates is unmeasured.
5. Clock discipline for TC. GPS-locked timestamps are defined for digital layers only. Whether Langevin relaxation needs its own reference clock, or whether conditioning-clock quantization is sufficient, is unresolved.

---

## 13. Document History

| Version | Date | Changes |
|---|---|---|
| 0.1.0 | 2026-04 | Initial draft |
