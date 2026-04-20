# thermodynamic-core ↔ arkspace-core Interface Specification

**Version**: 0.1.0  
**Status**: Draft  
**Last Updated**: April 2026

---

## Overview

This document defines the interface between **thermodynamic-core** (Substrate Layer) and **arkspace-core** (Infrastructure Layer). Thermodynamic computing is the proposed physical hardware paradigm for the Exocortex Constellation satellite payloads — the technology that makes a 100M-neuron orbital node at 50–200W conceivable within the satellite power budget.

> **Note**: Both the TC hardware and the orbital neuromorphic payload are at TRL 1–2. This is a conceptual interface specification.

---

## Why TC for Orbital Infrastructure

The arkspace-core specification targets:
- **100M neurons per satellite node**
- **50–200W per payload**

Scaling current neuromorphic hardware (Intel Loihi 2: ~1M neurons / ~1W terrestrial) to 100M neurons naively requires ~100W minimum — before accounting for space qualification overhead. Thermodynamic computing offers a potential $10^3$–$10^7\times$ energy improvement over digital alternatives, making the orbital power budget achievable in principle.

---

## Interface Position

```
┌──────────────────────────────────────────────────────────────┐
│                   arkspace-core                               │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐   │
│  │                SATELLITE NODE                          │   │
│  │                                                        │   │
│  │  ┌─────────────────────────────────────────────────┐  │   │
│  │  │  TC PAYLOAD (thermodynamic-core)                 │  │   │
│  │  │                                                   │  │   │
│  │  │  [TSU Array] ←──── [Digital Conditioning Proc.]  │  │   │
│  │  │  p-bit sampling       (ARM / RISC-V host)         │  │   │
│  │  │  Langevin dynamics    bias vector injection        │  │   │
│  │  │  HBSC couplings       parameter storage (NVM)      │  │   │
│  │  └─────────────────────────────────────────────────┘  │   │
│  │                         │                              │   │
│  │  ┌──────────────────────▼──────────────────────────┐  │   │
│  │  │  OISL Router + Ground Link (arkspace-core spec)  │  │   │
│  │  └─────────────────────────────────────────────────┘  │   │
│  └───────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## Hardware Interface Contracts

### Power Budget

| Component | Budget | Notes |
|---|---|---|
| TSU array (p-bit sampling cells) | ≤ 100W | Must leave headroom for conditioning processor + thermal control |
| Digital conditioning processor | ≤ 20W | ARM/RISC-V running bias vector computation |
| Non-volatile parameter memory | ≤ 5W | Stores $\{J_{ij}, b_i\}$ |
| Thermal control subsystem | ≤ 30W | Active control to stabilize $k_BT$ |
| **Total payload budget** | **≤ 200W** | Per arkspace-core specification |

### Parameter Update Interface (Uplink)

| Property | Specification |
|---|---|
| Update trigger | Ground-computed $\theta$ update via Ka-band / optical uplink |
| Data format | Compressed $\{J_{ij}, b_i\}$ delta (only changed parameters) |
| Encryption | AES-256 (consistent with Neural Firewall) |
| Storage | Non-volatile memory (embedded MRAM preferred for radiation tolerance) |
| Integrity check | Cryptographic signature verified before programming |

### Thermal Interface

TC hardware efficiency depends on operating temperature $T$ — $k_BT$ sets the fluctuation amplitude:

| Requirement | Value | Rationale |
|---|---|---|
| Operating temperature | 280–320 K (7–47°C) | Stable $k_BT$; avoid LEO -40° to +80° extremes |
| Temperature variation tolerance | ≤ ±5 K during operation | Larger swings alter $\Delta$ (thermal stability factor) of p-bits |
| Active thermal control | Required | Passive thermal management insufficient for ≤±5 K in LEO |

---

## Radiation Environment Interface

### LEO Radiation Threats and TC Response

| Threat | Impact on TC | Mitigation |
|---|---|---|
| Single-Event Upsets (SEU) | Random p-bit state flip — may be tolerable (already stochastic) | Error-tolerant architecture study needed |
| Total Ionizing Dose (TID) | Shifts subthreshold transistor thresholds → $\mu$ and $k_BT_{\text{effective}}$ drift | Periodic $\theta$ recalibration via uplink |
| Displacement Damage | Carrier mobility degradation | Shielding + recalibration |
| Thermal cycling (-40 to +80°C) | $k_BT$ variation → p-bit behavior shifts | Active thermal control (see above) |

### Radiation Hardening Requirements (TBD)

> These requirements are unresolved — no radiation-hardened TC chips exist as of April 2026. This table defines target specifications for future hardware development:

| Parameter | Target | Current Status |
|---|---|---|
| TID tolerance | ≥ 30 krad(Si) for 2-year LEO mission | Not characterized |
| SEU cross-section | TBD | Not characterized |
| Operating TID range | Full spec over 0–30 krad | Not characterized |

---

## Data Interface: TC ↔ OISL Router

The OISL router transmits neural spike data between satellite nodes. TC hardware outputs equilibrium samples, not conventional spike trains:

| TC Output Format | OISL Packet Format | Conversion |
|---|---|---|
| Boltzmann sample (binary vector) | Spike event list | Each $x_i = +1$ → spike event at node $i$ |
| Langevin trajectory (continuous) | Rate-coded packets | $r_i = (x_i + 1)/2 \times r_{\max}$ |
| DTM step output | Streaming sample | One DTM step → one OISL frame |

---

## Technology Gaps

| Gap | Severity | Path |
|---|---|---|
| No radiation-hardened TC chips | **Critical** | Long-term fab R&D; near-term: shielding + recalibration |
| Thermal management spec undefined | High | Phase 3: thermal envelope study |
| TC power scaling to 100M neurons | High | Power model study; multi-chip TSU array design |
| SEU tolerance analysis | High | Formal fault analysis of stochastic hardware in radiation environment |
| OISL ↔ TC data format | Medium | Phase 3: format standardization |

---

## Related Documentation

- Detailed TC side: [`thermodynamic-core/docs/integration/with-arkspace.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/integration/with-arkspace.md)
- arkspace-core satellite node spec: [`arkspace-core/docs/satellite-node.md`](https://github.com/Zae-Project/arkspace-core)
- TC hardware primitives: [`thermodynamic-core/docs/architecture/hardware-primitives.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/architecture/hardware-primitives.md)
