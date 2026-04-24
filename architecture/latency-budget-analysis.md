# Latency Budget Analysis

**Version:** 0.1.0
**Last Updated:** April 2026
**Status:** Draft
**Classification:** Private/Internal

---

## 1. Scope

This document is the depth layer below the Latency Budget overview in [unified-platform-architecture.md §Latency Budget Analysis](./unified-platform-architecture.md). It governs:

- End-to-end round-trip time (RTT) from a biological event to the stimulation of the same subject
- Per-stage targets, 99th percentile ceilings, and variance budgets
- Timing failure modes and their neural-level consequences
- Mitigations (predictive cache, edge fallback, TC substrate relaxation)

Out of scope. Payload and routing decisions are in [data-flow-architecture.md](./data-flow-architecture.md). Security-related timing (kill switch, anomaly detection cadence) is in [security-architecture.md](./security-architecture.md).

---

## 2. The Libet Buffer

Benjamin Libet's experiments (1983, 1985) established that conscious awareness of a voluntary movement trails the onset of the readiness potential by roughly 350 ms, with W-time at about 200 ms before movement. The 350 ms figure is the number that matters for engineering. It sets the ceiling on how long a satellite-mediated processing loop can take before the subject experiences a perceptual break.

This framing is contested. Schurger et al. (2012) argue the readiness potential reflects stochastic accumulation rather than a premade unconscious decision, which narrows the interpretive value of Libet's number but does not change the engineering ceiling. Cited sources live in [bibliography.md](../reference/bibliography.md).

Engineering consequences:

- **Hard ceiling.** RTT must remain below 350 ms for the subject to experience unified agency.
- **Design target.** 50 ms mean RTT provides 7x margin. This is the figure used in [unified-platform-architecture.md](./unified-platform-architecture.md) and [api-contracts.yaml](../integration/api-contracts.yaml) (`performance_sla.total_rtt_ms: target 40, maximum 70`).
- **Variance matters more than mean.** A loop that averages 40 ms but has a 99th percentile of 250 ms will still produce perceptual breaks at the p99 rate. Budget the tails, not the means.

---

## 3. End-to-end RTT Decomposition

Forward path (biological to satellite SNN). Reverse path is the mirror.

| Stage | Target mean | p99 | Jitter σ | Basis |
|---|---|---|---|---|
| CMOS ADC + spike detection | 0.3 ms | 1 ms | 0.1 ms | On-chip, deterministic. From [unified-platform-architecture.md §Latency Breakdown](./unified-platform-architecture.md). |
| Feature extraction + LZ4 encode | 0.5 ms | 1 ms | 0.2 ms | LZ4 is 1 ms latency per [api-contracts.yaml](../integration/api-contracts.yaml), `data_formats.compression`. Feature extraction overlaps. |
| UWB sub-cranial to edge | 1 ms | 2 ms | 0.3 ms | [arkspace-mindtransfer-interface.md](../integration/arkspace-mindtransfer-interface.md) step 3. |
| Neural Firewall encrypt (AES-256-GCM) | 0.5 ms | 1 ms | 0.1 ms | [api-contracts.yaml](../integration/api-contracts.yaml), `security.encryption.real_time_streams.latency_overhead_ms: 1`; [mindtransfer-consciousness-interface.md §Latency Budget](../integration/mindtransfer-consciousness-interface.md). |
| Predictive-coding encode | 1 ms | 2 ms | 0.3 ms | Dream Engine, [unified-platform-architecture.md §Technology Stack](./unified-platform-architecture.md). |
| Edge to ground segment | 3 ms | 5 ms | 1 ms | Fiber or 6G, [arkspace-mindtransfer-interface.md](../integration/arkspace-mindtransfer-interface.md) step 4. |
| Ground to LEO uplink (Ka-band or 1550-nm) | 10 ms | 15 ms | 2 ms | [api-contracts.yaml](../integration/api-contracts.yaml), `performance_sla.latency.ground_to_satellite_ms: target 10, maximum 15`. |
| OISL inter-satellite hop (per hop) | 10 ms | 20 ms | 2 ms | [api-contracts.yaml](../integration/api-contracts.yaml), `inter_satellite_hop_ms`. Most sessions take 0 or 1 hop. |
| SNN inference on neuromorphic payload | 5 ms | 10 ms | 1 ms | [unified-platform-architecture.md §Latency Breakdown](./unified-platform-architecture.md). |
| TC substrate relaxation (when hybrid path is active) | 20 ms | 50 ms | 5 ms | [thermodynamic-consciousness-interface.md §Latency Budget](../integration/thermodynamic-consciousness-interface.md). Adds to the SNN figure only when DTM is used. |
| **Forward total (no TC, no hop)** | **~22 ms** | **~37 ms** | **~3 ms RSS** | Sum of rows above through SNN inference. |
| **Forward total (with 1 OISL hop)** | **~32 ms** | **~57 ms** | n/a | Adds one OISL hop. |
| **Forward total (with TC DTM step)** | **~42 ms** | **~77 ms** | n/a | Adds TC relaxation. |
| Return path | mirror of forward | mirror | n/a | Symmetric unless TC is only on forward path. |
| **RTT (nominal, no TC, no hop)** | **~44 ms** | **~74 ms** | n/a | Mean is 4 ms above the api-contracts target (40 ms). p99 is 4 ms above the api-contracts maximum (70 ms). |
| **RTT (with 1 OISL hop, no TC)** | **~64 ms** | **~114 ms** | n/a | Over the p99 ceiling. Requires edge fallback to keep within Libet margin. |

Jitter aggregate is root-sum-square over independent stages, not linear sum. Linear addition would massively overstate the tail.

Notes on the table:
- The api-contracts SLA is 40 ms target / 70 ms max for RTT. The straight sum of per-stage means in this decomposition lands at 44 ms, already 4 ms above the target. The 40 ms target is achievable only if the return path runs lighter than the forward path (return has no feature extraction, no predictive-coding encode, and no SNN inference on the return direction, since inference happens on forward). [unified-platform-architecture.md §Latency Breakdown](./unified-platform-architecture.md) uses a return path estimate of ~20 ms on that basis, yielding ~49 ms RTT. The decomposition in this document is deliberately symmetric to expose where the return path has to be shortened to meet the 40 ms target.
- The p99 column aggregates per-stage maxima linearly, which is a worst-case estimate. Jitter RSS (section 4) puts the realistic p99 closer to the 70 ms ceiling than the 74 ms shown here, but the headroom is narrow.
- Two-hop paths (OISL with one relay) sit above the p99 ceiling and are not suitable for the motor loop. They are tolerable for bulk state migration (BULK class in [data-flow-architecture.md §QoS](./data-flow-architecture.md)).
- TC relaxation is on the forward path only when the Dream Engine routes through a DTM step. Most sessions do not; TC is active during predictive-coding inference, not per-spike.

---

## 4. Jitter and Variance Budget

Jitter is the quantity to engineer against. A mean-40 ms path with unbounded tails is worse than a mean-60 ms path with p99 at 70 ms for subjective continuity.

Aggregate jitter budget:

- **Target aggregate σ (one-way).** ≤ 4 ms
- **Target aggregate σ (round-trip).** ≤ 6 ms (RSS, not sum)
- **p99 ceiling (round-trip).** 70 ms, per [api-contracts.yaml](../integration/api-contracts.yaml)
- **Hard Libet ceiling.** 350 ms under any condition

Variance sources ranked by contribution:
1. Ground-to-LEO uplink (atmospheric turbulence, scheduler queueing)
2. OISL hop when one is present (router queueing)
3. TC substrate relaxation when active (stochastic by construction)
4. Predictive-coding encode when the model drifts
5. Everything else combined

This ranking informs where monitoring must be densest. The session status endpoint (`GET /sessions/{session_id}`) exposes per-direction latency and packet loss for this reason ([api-contracts.yaml](../integration/api-contracts.yaml), `get_session_status`).

Continuity under jitter:
- Motor channel. p99 above 100 ms produces perceptible lag on fine motor tasks even though it is well below the Libet ceiling.
- Sensory channel. Tolerant to jitter; the predictive model fills gaps.
- Safety / emergency. CRITICAL class must stay at ≤ 5 ms per hop; emergency stop must complete within 100 ms end to end ([api-contracts.yaml](../integration/api-contracts.yaml), `emergency_stop.response_time_requirement_ms`).

---

## 5. Timing Failure Modes

| Failure | Threshold | Neural-level effect | Source |
|---|---|---|---|
| RTT above 100 ms sustained | p99 breach, not hard ceiling | Perceptible motor lag, degraded agency | Engineering judgment from this doc's aggregate budget |
| RTT above 350 ms sustained | Libet ceiling breach | Loss of unified agency; perceptual split | [bibliography.md](../reference/bibliography.md) (Libet 1985) |
| Stimulation inter-pulse jitter above 5 ms at high frequency | Frequency-dependent | Risk of driving into seizure-induction territory if coupled with a rising amplitude envelope | [bibliography.md](../reference/bibliography.md) (Pycroft 2016); safety limits in [api-contracts.yaml](../integration/api-contracts.yaml), `safety_limits` |
| Packet reordering window above 5 ms | Exceeds reorder buffer | LIF state corruption at L6; spikes dropped at reorder buffer | [data-flow-architecture.md §Failure Modes](./data-flow-architecture.md) |
| Clock drift above 500 µs across nodes | GPS discipline lost | Temporal coding unreliable; rate coding still works | [data-flow-architecture.md §Failure Modes](./data-flow-architecture.md) |
| OISL handover duration above 1 s | Failover SLA breach | Motor loop falls to predictive cache | [api-contracts.yaml](../integration/api-contracts.yaml), `performance_sla.reliability.failover_time_seconds: maximum 1` |

Stimulation safety limits that constrain the timing design:
- max amplitude 100 µA
- max frequency 200 Hz
- max charge per phase 50 nC
- max duty cycle 50%
(per [api-contracts.yaml](../integration/api-contracts.yaml), `safety_limits`)

A stimulation plan whose inter-pulse interval is perturbed by jitter can violate the duty cycle constraint even if the nominal plan respects it. The safety filter at L3 recomputes duty cycle against the actual departure times, not the planned times.

---

## 6. Mitigation Strategies

### 6.1 Local predictive cache at edge

The Dream Engine maintains a local copy at L3 (edge node). When the ground-to-LEO path degrades past the p99 ceiling, the cache serves the motor loop at reduced fidelity rather than stalling. Cache freshness is the tradeoff: longer cache windows tolerate longer outages but accumulate prediction error.

- Refresh cadence. 1 Hz heartbeat (per [api-contracts.yaml](../integration/api-contracts.yaml), `websocket_api.heartbeat.frequency_seconds: 1`).
- Useful cache window. Up to 5 s for sensory channels; 500 ms for motor.
- Degraded service signaling. `WARNING` alert on the control channel.

### 6.2 Edge fallback when orbit path degrades

If the uplink fails outright, the session falls to a ground-only predictive path. This is not real substrate transfer; it is a holdover to prevent perceptual break during handover. Hard cutoff at 1 s per the failover SLA.

### 6.3 Generative-model prefetch

The Dream Engine predicts the next window of sensory input and pre-stages the corresponding stimulation plan. Jitter on the forward path is absorbed by this buffer up to the prediction horizon.

### 6.4 SNN state pre-warm during handover

Satellite handover is scheduled, not random. The target payload is pre-warmed with the current SNN state via `PUT /nodes/{node_id}/snn/state` ([api-contracts.yaml](../integration/api-contracts.yaml), `load_snn_state`) within the migration budget of 100 ms target / 200 ms maximum. This keeps handover RTT bounded.

---

## 7. TC Substrate Timing Contribution

Thermodynamic relaxation is nanoseconds to microseconds at the physics layer. The visible latency in the data plane is dominated by:

- Digital conditioning compute at the L6 host (<5 ms per [thermodynamic-consciousness-interface.md](../integration/thermodynamic-consciousness-interface.md))
- Host-to-substrate parameter staging
- Sample decoding back to spike events

The total DTM step budget is <20 ms and a full inference chain is <50 ms. At the aggregate jitter scale (4 ms σ one-way), the intrinsic substrate jitter (µs regime) is below the noise floor. The variability seen by the rest of the system comes from the digital conditioning compute and from stochastic early-stopping in the relaxation loop.

Implication. The TC substrate does not tighten the timing budget; it loosens energy. Latency-critical channels should bypass TC until a hardware profile with bounded DTM variance is available.

---

## 8. Cross-references

| Segment | Authoritative latency contract |
|---|---|
| L2 to L4 (BCI to edge to ground) | [arkspace-mindtransfer-interface.md §Latency](../integration/arkspace-mindtransfer-interface.md) |
| L4 to L6 (ground to SNN payload) | [arkspace-consciousness-interface.md §Latency Budget](../integration/arkspace-consciousness-interface.md) |
| L1 to L3 (BCI to firewall) | [mindtransfer-consciousness-interface.md §Latency Budget](../integration/mindtransfer-consciousness-interface.md) |
| TC substrate timing | [thermodynamic-consciousness-interface.md §Latency Budget](../integration/thermodynamic-consciousness-interface.md) |
| SLA figures | [api-contracts.yaml](../integration/api-contracts.yaml), `performance_sla` |

---

## 9. Open Questions

1. Queueing model. No agreed closed-form model for the edge-to-ground queue. M/M/1 understates tails; G/G/1 requires traffic shape measurements we do not have yet.
2. Handover tail shape. Scheduled OISL handovers produce a known latency spike, but the distribution is unmeasured. The 1 s failover SLA is a ceiling; the p99 inside that second is unknown.
3. Corpus callosum baseline. The biological signal traverses the callosum at roughly 10 to 20 m/s on myelinated fibers. The engineering target of 50 ms RTT is already faster than the longest biological delay on the longest callosal fiber, which is the justification for calling the loop "seamless". This claim needs a formal comparison table.
4. TC relaxation variance under load. <50 ms is a budget, not a measurement. p99 of TC inference under continuous load has not been profiled.
5. GPS holdover. All timing math assumes GPS-locked clocks. What happens during GPS outage (ionospheric scintillation, jamming) is not specified.

---

## 10. Document History

| Version | Date | Changes |
|---|---|---|
| 0.1.0 | 2026-04 | Initial draft |
