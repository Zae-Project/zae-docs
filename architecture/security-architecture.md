# Security Architecture

**Version:** 0.1.0
**Last Updated:** April 2026
**Status:** Draft
**Classification:** Private/Internal

---

## 1. Scope

This document is the depth layer below the Security Architecture overview in [unified-platform-architecture.md §Security Architecture](./unified-platform-architecture.md). It governs the neural-security stack across all four pillars:

- Threat model (brainjacking and adjacent classes)
- Trust zones and boundaries
- Attack surfaces per zone
- Cryptographic primitives, key management, protocol bindings
- Defense in depth (encryption, anomaly detection, hardware kill switch, zero trust)
- Role of the thermodynamic substrate in the security posture
- Post-quantum stance and incident response

Out of scope. Orbit-segment specifics (FCC / ITAR, orbital debris, SATCOM literature) are maintained in [arkspace-core/SECURITY.md](https://github.com/Zae-Project/arkspace-core/blob/main/SECURITY.md) and are referenced here rather than duplicated. Latency-sensitive timing of security components (kill-switch response, anomaly detection cadence) is cross-referenced into [latency-budget-analysis.md](./latency-budget-analysis.md).

---

## 2. Threat Model

### 2.1 Brainjacking taxonomy

Pycroft et al. (2016) established the term. Entries extended for the satellite-mediated case. Full citations in [bibliography.md](../reference/bibliography.md).

| Class | Mechanism | Target effect | Primary defense |
|---|---|---|---|
| Induced seizure | High-frequency or high-amplitude stimulation | Clinical seizure | Stimulation safety limits at L3 filter; hardware kill switch |
| Forced motor command | Stimulation targeted at motor cortex electrodes | Involuntary movement | HMAC-SHA256 on every `StimulationCommandDownlink`; kill switch |
| Sensory manipulation | Stimulation of sensory pathways | False percepts | Anomaly detection on output distribution; kill switch |
| Memory or thought injection | Patterned stimulation across associative areas | Implanted associations | Speculative class. No validated defense beyond the above. |
| Cognitive exfiltration | Passive readout of decoded neural state | Privacy breach | End-to-end encryption; zero-trust segmentation |
| Identity spoofing | Impersonating a legitimate session peer | Session takeover | X.509 device certificates, ECDH key exchange, MFA on user plane |
| Replay | Retransmission of captured packets | Repeated stimulation, state desync | Sequence numbers, timestamp-based IV ([api-contracts.yaml](../integration/api-contracts.yaml), `iv_generation`) |
| Supply chain | Hardware or firmware implant during manufacturing | Persistent compromise | Trusted foundry, supply chain audit ([arkspace-core/SECURITY.md §4](https://github.com/Zae-Project/arkspace-core/blob/main/SECURITY.md)) |
| Side channel | EM, thermal, acoustic, timing leakage | Key or state recovery | Masked crypto, physical shielding; TC noise envelope (section 10) |

### 2.2 Threat actors

Adapted from [arkspace-core/SECURITY.md](https://github.com/Zae-Project/arkspace-core/blob/main/SECURITY.md) for the neural-data case.

| Actor | Capability | Target asset |
|---|---|---|
| Nation state | ASAT, cyber, supply chain | Constellation, cryptographic roots |
| Criminal | Ransomware, credential theft | User plane, session hijack |
| Insider | Privileged access | Key material, model parameters, patient data |
| Supply chain | Hardware implants | Neuromorphic payload, TC hardware, implant firmware |
| Signal attacker | RF / optical jamming or dazzling | Uplink, OISL |
| Device attacker (physical) | Access to the implanted transceiver or edge node | Long-lived session keys, HSM |

Scope note. Space debris is a safety hazard tracked in [arkspace-core/SECURITY.md §5](https://github.com/Zae-Project/arkspace-core/blob/main/SECURITY.md). It is not a security threat in the information-security sense and is not reproduced here.

---

## 3. Trust Zones

Ordered from highest trust (closest to tissue) to lowest trust (public internet). A boundary between two zones requires authentication, encryption, and policy enforcement.

| Zone | Contents | Boundary controls to neighbor |
|---|---|---|
| Z0. Biological tissue | Neurons, axons, corpus callosum | Physical: CMOS array. No digital crossing; analog interface only. |
| Z1. Implanted transceiver | CMOS array, ADC, UWB hub, HSM | AES-256-GCM envelope; HSM-rooted identity |
| Z2. Edge node (Neural Firewall) | Firewall, anomaly detection, key cache | ECDH session key rotation every 1 h ([api-contracts.yaml](../integration/api-contracts.yaml), `session_level.key_rotation_hours: 1`); TLS 1.3 to ground |
| Z3. Ground station | Uplink terminal, telemetry, REST API | X.509 device certs; OAuth 2.0 + MFA on user plane |
| Z4. LEO constellation | Satellite bus, OISL router, SNN payload | CCSDS authentication; OISL beam geometry; per-packet auth inside constellation fabric |
| Z5. TC substrate (inside Z4 payload) | p-bit array, conditioning processor, NVM | Substrate is addressable only through conditioning processor; parameter writes require cryptographic signature |
| Z6. Operator control plane | Engineers, admins, support | RBAC roles from [api-contracts.yaml](../integration/api-contracts.yaml), `security.authorization.roles`; break-glass on ADMIN only |
| Z7. Public internet | Everything else | Never directly connected to Z1 or Z2 |

Zero trust applies within every zone, not only between zones (section 8).

---

## 4. Attack Surfaces per Zone

| Zone | Surface | Dominant threat |
|---|---|---|
| Z1 | Physical access to implant; firmware update path; HSM side channels | Device attacker, insider |
| Z2 | Local network, USB, power side channel, firmware | Device attacker, malware |
| Z3 | Internet-facing REST, WebSocket, OAuth provider | Credential theft, protocol flaws, DDoS |
| Z4 | OISL beam intercept, uplink RF interception, command injection | Signal attacker, nation state |
| Z5 | Parameter write path; conditioning processor firmware | Supply chain, insider |
| Z6 | Privileged sessions | Insider, phishing |

Orbit-segment specifics (OISL optical eavesdropping, laser dazzling, link acquisition spoofing, radiation-induced bit flips) live in [arkspace-core/SECURITY.md](https://github.com/Zae-Project/arkspace-core/blob/main/SECURITY.md). They are not reproduced here but are treated as load-bearing in the defense-in-depth layers below.

---

## 5. Cryptographic Primitives

Canonical bindings from [api-contracts.yaml](../integration/api-contracts.yaml), `security` section.

| Primitive | Use | Latency overhead | Notes |
|---|---|---|---|
| AES-256-GCM | Real-time neural stream (L2 to L5) | 0.5 to 1 ms | Per-record IV derived from timestamp, never reused |
| HKDF | Key derivation from ECDH shared secret | <0.1 ms | SHA-256 based |
| ECDH on Curve25519 | Session key exchange | 1 ms on edge node | Perfect forward secrecy; rotated every 1 h |
| TLS 1.3 (`TLS_AES_256_GCM_SHA384`) | Control plane (REST, WebSocket control) | Typical TLS | Handshake amortized per session |
| HMAC-SHA256 | `StimulationCommandDownlink.safety_signature` | <0.1 ms | Verified at L3 before any stimulation is enacted |
| Paillier homomorphic encryption | Identity handshakes and cognitive-privacy-preserving operations | Slow (>100 ms) | Not on the hot path; bounded use |
| Hybrid homomorphic encryption (HHE) | Cognitive-privacy-preserving stream, candidate to replace pure FHE on this path | Symmetric on the wire (sub-ms); HE cost moved server-side | HERA / Rubato symmetric cipher transcrypted into CKKS server-side. Candidate for the section 13 throughput ceiling; not yet adopted |
| CRC32 | Packet integrity on hot-path messages | <0.1 ms | Integrity only; not authenticity |
| SHA-256 | State snapshot integrity | ~1 ms per MB | Used in `StateHeader.checksum_sha256` ([api-contracts.yaml](../integration/api-contracts.yaml)) |

Choices that are not yet fixed. Post-quantum algorithms (section 11), QKD over OISL ([arkspace-core/SECURITY.md §2](https://github.com/Zae-Project/arkspace-core/blob/main/SECURITY.md)).

---

## 6. Key Management Lifecycle

### 6.1 Hardware root of trust

- Implant. HSM or TPM inside the sub-cranial transceiver. Device key is provisioned at implant time and never leaves the chip. Attestation on every session creation.
- Edge node. TPM-backed key cache. Receives session keys from ECDH exchanges; never exposes long-lived keys.
- Satellite. TPM-equivalent in payload controller. Holds OISL link keys and the parameter-write verification key.
- Ground station. HSM backing the CA that issues X.509 device certs ([api-contracts.yaml](../integration/api-contracts.yaml), `device_level.storage: TPM/HSM`).

### 6.2 Lifecycle

| Phase | Action |
|---|---|
| Provision | At implant time. Device key pair generated on HSM. CA signs device cert (validity 365 d). User-side MFA enrolled. |
| Session bootstrap | POST `/sessions` (see [api-contracts.yaml](../integration/api-contracts.yaml), `create_session`). Issues `encryption_key_id` and `encryption_key`, scopes to session duration. |
| Rotation | ECDH rerun every 1 h during active session. Per-hop symmetric keys derived via HKDF. |
| Revocation | Device cert revocation via CRL + short-lived OCSP stapling. Session keys expire on rotation boundary even if revocation lags. |
| Decommission | Device cert revoked. HSM attests wipe of session keys. Long-lived device key destroyed at explant. |

### 6.3 Key escrow stance

None. No third party holds a copy of any key used on the neural data plane. Recovery of a bricked device requires explant and reprovisioning, not key recovery.

---

## 7. Defense in Depth

The four layers extend the outline in [unified-platform-architecture.md §Security Architecture](./unified-platform-architecture.md). Each layer lists coverage in terms of the classes in section 2.1.

### 7.1 Layer 1. Encryption

- AES-256-GCM on every real-time stream between Z1 and Z4.
- TLS 1.3 on the control plane (ground REST, WebSocket control messages).
- Paillier FHE reserved for identity handshakes and privacy-preserving operations; off the hot path.
- Hybrid homomorphic encryption (HHE, HERA / Rubato transcrypted to CKKS) is the candidate for any cognitive-privacy-preserving stream that needs HE semantics without the pure-FHE latency. See section 13, open question 1.
- End to end from implant HSM to payload TPM. No cleartext exposure at any intermediate zone.

Covers: cognitive exfiltration, identity spoofing (with ECDH+certs), replay (with timestamp IV and sequence numbers).

### 7.2 Layer 2. Anomaly detection

- Stimulation-side. L3 recomputes frequency, duty cycle, and charge per phase against the actual emission schedule, not the planned schedule (section references [api-contracts.yaml](../integration/api-contracts.yaml), `safety_limits`). Violations trigger the kill switch.
- SNN output distribution. ML models at L3 and L6 track baseline distributions over motor and sensory channels. Deviations trigger `WARNING` alerts; persistent deviations escalate to `CRITICAL`.
- Seizure-frequency filter. Dedicated filter looks for stimulation patterns above 200 Hz or with rising amplitude envelopes. Zero false negatives is the requirement; false positives are acceptable.
- TC equilibrium sampling. Distribution shift on TC output is a signal that parameter memory has been tampered with or that radiation has drifted the substrate. Referenced in [thermodynamic-consciousness-interface.md §Security Extensions](../integration/thermodynamic-consciousness-interface.md).

Covers: induced seizure, forced motor command, sensory manipulation, supply chain (output drift), side channel (indirectly).

### 7.3 Layer 3. Hardware kill switch

- Physical circuit breaker in the sub-cranial transceiver.
- Voltage clamp that limits per-electrode current to the safety ceiling (100 µA, [api-contracts.yaml](../integration/api-contracts.yaml), `safety_limits.max_amplitude_uA`).
- Fail-safe default. On loss of authenticated heartbeat for 5 s, the implant stops stimulation and holds sensing only.
- Manual override. Subject-triggered (wearable button) and clinician-triggered. Both paths route to the same hardware breaker.
- API trigger. `POST /firewall/kill-switch` must complete within 100 ms ([api-contracts.yaml](../integration/api-contracts.yaml), `activate_kill_switch.response_time_requirement_ms`).

Covers: induced seizure (final stop), forced motor command (final stop), all compromise scenarios (operator override).

### 7.4 Layer 4. Zero trust

- Per-packet authentication on every hop (ECDH session key + HMAC).
- No implicit trust across zones or within them.
- Continuous verification. Every session status check runs a health probe of the upstream chain.
- Least privilege across the RBAC roles ([api-contracts.yaml](../integration/api-contracts.yaml), `security.authorization.roles`).

Covers: identity spoofing, insider abuse, lateral movement after a single-zone compromise.

---

## 8. Anomaly Detection Deployment

| Model | Home zone | Detects | Action |
|---|---|---|---|
| Stim safety filter | Z2 | Safety-limit violations on actual emission | Drop packet; trigger kill switch if rate exceeds threshold |
| Motor-command classifier | Z2 | Commands inconsistent with intent baseline | Hold; raise alert |
| Sensory distribution monitor | Z2, Z4 | Distribution drift on sensory stimulation plans | Raise alert; degrade to predictive fallback |
| SNN output monitor | Z4 | Unusual activity in the SNN payload itself | Raise alert; quarantine payload |
| TC equilibrium monitor | Z5 | Drift in TC sample distribution | Trigger parameter recalibration; alert |

Model updates follow the same signed-delta flow as TC parameter updates (section 6) and are subject to the engineer RBAC role ([api-contracts.yaml](../integration/api-contracts.yaml)).

---

## 9. Incident Response

1. **Detect.** Alerts from section 8 arrive via `GET /firewall/events` ([api-contracts.yaml](../integration/api-contracts.yaml)).
2. **Isolate.** Scope the kill switch to `SESSION`, `NODE`, or `CONSTELLATION` depending on blast radius. Default is `SESSION`.
3. **Stabilize.** Edge predictive cache maintains the motor loop during isolation. See [latency-budget-analysis.md §Mitigation Strategies](./latency-budget-analysis.md).
4. **Forensic capture.** Signed event log with last 60 s of encrypted neural stream ciphertext. Keys archived per retention policy (open question, section 13).
5. **Recover.** Rotate all session keys. If device compromise is suspected, revoke cert and require reprovisioning.
6. **Postmortem.** Update threat model, detection rules, and (if applicable) the safety filter.

The operator-plane incident process (paging, RBAC escalation to ADMIN) is out of scope for this document and lives with the operator runbook.

---

## 10. Thermodynamic Substrate Security Role

The TC substrate touches security in three ways.

### 10.1 Parameter memory as high-value target

The $\{J_{ij}, b_i\}$ values define the SNN the TC substrate embodies. They are signed at programming time and verified before programming ([thermodynamic-arkspace-interface.md §Parameter Update Interface](../integration/thermodynamic-arkspace-interface.md)). Compromise here is equivalent to modifying the SNN weights in a digital payload.

### 10.2 Side-channel stance

Stochastic hardware has a different side-channel profile from deterministic CMOS. The intrinsic thermal noise envelope obscures fine-grained timing and power signatures of the underlying computation. This is a theoretical advantage against side-channel attacks on cryptographic reduction paths that happen to run on TC. It is not a general-purpose side-channel defense, and conventional masked implementations remain required for AES and ECDH.

### 10.3 p-bit entropy as true random source and PUF

Per [thermodynamic-core/docs/architecture/hardware-primitives.md](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/architecture/hardware-primitives.md), p-bit thermal fluctuation is an entropy source. Candidate uses:
- True random number generator for key generation and IVs.
- Physical unclonable function (PUF) from manufacturing variation in p-bit threshold voltages, for device identity.

Both uses require health testing (min-entropy bounds, start-up behavior under radiation) before production. Radiation and TID shift the substrate's statistical profile (see [thermodynamic-arkspace-interface.md §Radiation Environment](../integration/thermodynamic-arkspace-interface.md)), which means a TC-backed PUF needs periodic enrollment refresh.

---

## 11. Post-quantum Stance

Current cryptographic choices (AES-256, ECDH Curve25519, RSA in X.509 cert chains, Paillier) have different quantum exposures:

- AES-256. Safe against Grover with a 128-bit effective security margin. Keep.
- ECDH on Curve25519. Broken by a cryptographically relevant quantum computer (CRQC). Harvest-now-decrypt-later is the live risk, since stored ciphertext from today may be decrypted against future CRQC.
- Paillier. Broken by a CRQC.
- X.509 RSA / ECDSA. Broken by a CRQC.

Migration plan (draft):
1. Short term. Begin hybrid ECDH + Kyber (KEM) once NIST parameter sets finalize and TPM firmware supports them.
2. Medium term. Replace RSA/ECDSA signatures in the CA chain with Dilithium once broadly supported.
3. Long term. Replace Paillier with a post-quantum homomorphic scheme when one matures for the identity-handshake use case.

Signal longevity makes neural data a higher-than-average harvest target. The cognitive privacy guarantee has to outlast the cryptographic lifetime of any given primitive.

---

## 12. Cross-references

| Topic | Authoritative source |
|---|---|
| Orbit-segment threats (OISL intercept, laser dazzle, radiation bit flips, orbital debris, supply chain) | [arkspace-core/SECURITY.md](https://github.com/Zae-Project/arkspace-core/blob/main/SECURITY.md) |
| Crypto and authentication bindings | [integration/api-contracts.yaml](../integration/api-contracts.yaml), `security` section |
| Stimulation safety limits | [integration/api-contracts.yaml](../integration/api-contracts.yaml), `safety_limits`; [mindtransfer-consciousness-interface.md](../integration/mindtransfer-consciousness-interface.md) |
| TC substrate security contract | [integration/thermodynamic-consciousness-interface.md §Security Extensions](../integration/thermodynamic-consciousness-interface.md) |
| TC radiation environment | [integration/thermodynamic-arkspace-interface.md §Radiation Environment Interface](../integration/thermodynamic-arkspace-interface.md) |
| Defense-layer overview | [architecture/unified-platform-architecture.md §Security Architecture](./unified-platform-architecture.md) |
| Primary literature (Pycroft, Libet, Sperry, SATCOM) | [reference/bibliography.md](../reference/bibliography.md) |

---

## 13. Open Questions

1. FHE throughput ceiling. Paillier on the identity-handshake path is acceptable. Pure FHE cannot sustain a 100 Mbps stream at <1 ms latency for realtime cognitive-privacy-preserving inference. The realistic path is hybrid homomorphic encryption (HHE): a lightweight symmetric cipher (HERA, Rubato) on the wire, transcrypted into CKKS server-side, with reported 3 to 5x latency reductions and GPU/FPGA CKKS acceleration (KLSS key-switching ~181x on GPU). Whether HHE closes the gap on this path is the open item.
2. Post-quantum timeline. Waiting on NIST final parameter sets and TPM firmware support. Hybrid transition plan is sketched but not scheduled.
3. TC side-channel budget. The noise-envelope advantage is plausible, not measured. A formal evaluation on a concrete TC hardware profile is needed.
4. PUF enrollment drift under radiation. How often a TC PUF must be re-enrolled in LEO conditions is unknown. Paired with the TID question in [thermodynamic-arkspace-interface.md](../integration/thermodynamic-arkspace-interface.md).
5. Forensic retention. How long to keep encrypted neural stream ciphertext after an incident, given the cognitive-privacy constraint and the eventual CRQC decryption risk. Retention has to respect both the investigation and the subject.
6. Insider controls on parameter writes. Two-person integrity is assumed for $\theta$ updates in ground operations but not formalized in the RBAC roles.
7. Kill-switch adversary model. The hardware breaker is fail-safe open. An attacker who can induce repeated false triggers can still deny service. Detection of such abuse is not yet specified.

---

## 14. Document History

| Version | Date | Changes |
|---|---|---|
| 0.1.0 | 2026-04 | Initial draft |
