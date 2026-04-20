# thermodynamic-core ↔ neutral-consciousness-engine Interface Specification

**Version**: 0.1.0  
**Status**: Draft  
**Last Updated**: April 2026

---

## Overview

This document defines the interface between **thermodynamic-core** (Substrate Layer) and **neutral-consciousness-engine** (Engine/Software Layer). Thermodynamic hardware is the proposed physical substrate on which the SNN engine runs; this specification defines how SNN primitives map to TC primitives and how training and inference flow between the two layers.

> **Note**: This interface is speculative at TRL 2. No physical TC hardware is yet available. All specifications below describe target behavior for future hardware integration.

---

## Interface Position in Platform Architecture

```
┌──────────────────────────────────────────────────────────┐
│             neutral-consciousness-engine                  │
│  LIF neurons · STDP · Predictive Coding · Neural Firewall │
└─────────────────────┬────────────────────────────────────┘
                      │  Model weights + activation trajectories
                      │  (Onsager-Machlup training interface)
                      ▼
┌──────────────────────────────────────────────────────────┐
│                 thermodynamic-core                        │
│  p-bits · Langevin dynamics · HBSC · Digital conditioning │
└──────────────────────────────────────────────────────────┘
```

---

## SNN → TC Primitive Mapping

| SNN Primitive (neutral-consciousness-engine) | TC Primitive (thermodynamic-core) | Notes |
|---|---|---|
| LIF membrane potential | Continuous Langevin variable $x_i$ | Continuous analog of discrete spike accumulation |
| Firing threshold | Energy barrier in $V_\theta(x_i)$ | Threshold = local energy maximum |
| Stochastic spiking | p-bit thermal fluctuation | Same mathematical form: $\sigma(\beta V)$ |
| Synaptic weight $w_{ij}$ | Coupling $J_{ij}$ | Excitatory: $J_{ij} < 0$; Inhibitory: $J_{ij} > 0$ |
| Bias current $I_i$ | Bias $b_i$ in $V_\theta$ | Direct mapping |
| STDP weight update | Onsager-Machlup gradient $\Delta J_{ij}$ | Biological plasticity = thermodynamic gradient |
| Predictive coding error | Residual in conditioned energy | $\Delta V = V_\theta - V_{\text{target}}$ |
| Dream Engine (generative) | Denoising Thermodynamic Model (DTM) | DTM replaces neural-net diffusion step |

---

## Training Interface

### SNN-to-TC Parameter Transfer Protocol

1. **Train SNN** in neutral-consciousness-engine simulation (Nengo/PyTorch, GPU)
2. **Record trajectories**: For $N$ training inputs, record activation sequences $\{x^{(0)}, \ldots, x^{(K)}\}$
3. **Onsager-Machlup GD**: Run gradient descent on TC coupling parameters $\{J_{ij}, b_i\}$ to maximize trajectory probability:
   $$\Delta J_{ij} \propto \sum_k \frac{\Delta x_i^{(k)} + \mu\,\partial_i V_\theta \Delta t}{2k_BT} \cdot x_j^{(k)}$$
4. **Validate fidelity**: Check activation cosine similarity ≥ 0.99 (per Whitelam & Casert 2026 benchmark)
5. **Program hardware**: Write learned $\{J_{ij}, b_i\}$ to TSU non-volatile parameter memory

### Fidelity Requirements

| Metric | Threshold |
|---|---|
| Activation cosine similarity | ≥ 0.99 |
| Spike timing correlation | ≥ 0.90 |
| Generative model quality (FID / KL) | Within 5% of GPU baseline |

---

## Runtime Interface

### Inference Flow

```
[Input: biological spike data from brain-emulation]
         ↓
[Digital Conditioning Interface (host processor)]
  → Compute bias vectors b_enc, b_dec
         ↓
[TC Substrate: Langevin relaxation]
  → Output: equilibrium sample ≈ SNN activation
         ↓
[neural-consciousness-engine runtime]
  → Neural Firewall validation
  → Route to downstream SNN layers
```

### Latency Budget

| Stage | Budget |
|---|---|
| Digital conditioning compute | < 5 ms |
| TC Langevin relaxation (per DTM step) | < 20 ms |
| Full DTM inference chain | < 50 ms |
| OISL satellite round-trip | < 50 ms |
| Libet buffer (remaining) | ~200 ms |

---

## Security Extensions

The Neural Firewall in neutral-consciousness-engine must cover the TC substrate:

| Requirement | Implementation |
|---|---|
| Parameter integrity | Cryptographic signing of $\{J_{ij}, b_i\}$ at programming time |
| Inference channel | AES-256 on digital conditioning bus |
| Output anomaly detection | Distribution shift monitoring on TC equilibrium samples |

---

## Technology Gaps

| Gap | Path Forward |
|---|---|
| TC hardware not yet available | GPU simulation (THRML / JAX) as Phase 2 proxy |
| SNN-to-TC training pipeline | Phase 2: Onsager-Machlup on MNIST; Phase 3: full SNN |
| Relaxation latency unknown | Needs physical TSU benchmarking |
| Neural Firewall on stochastic hardware | Security architecture TBD |

---

## Related Documentation

- Detailed TC side: [`thermodynamic-core/docs/integration/with-consciousness-engine.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/integration/with-consciousness-engine.md)
- Langevin training details: [`thermodynamic-core/docs/algorithms/langevin-training.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/algorithms/langevin-training.md)
- neutral-consciousness-engine architecture: [`neutral-consciousness-engine/ARCHITECTURE.md`](https://github.com/Zae-Project/neutral-consciousness-engine/blob/main/ARCHITECTURE.md)
