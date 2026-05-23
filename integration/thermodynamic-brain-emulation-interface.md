# thermodynamic-core ↔ brain-emulation Interface Specification

**Version**: 0.1.0
**Status**: Draft
**Last Updated**: April 2026

---

## Overview

This document defines the interface between **thermodynamic-core** (Substrate Layer) and **brain-emulation** (Interface Layer). The relationship is primarily scientific. brain-emulation simulates biologically realistic neural dynamics in Brian2 (deterministic ODEs, optionally noisy). thermodynamic-core defines a stochastic physical substrate (Langevin SDE on continuous variables, p-bit thermal fluctuation). The contract specifies how brain-emulation's atlas-based templates map to TC primitives, and how TC outputs feed back into brain-emulation's visualization and benchmark surfaces.

The TC-side view lives at [`thermodynamic-core/docs/integration/with-brain-emulation.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/integration/with-brain-emulation.md). This document is the zae-docs mirror, authoritative for the cross-repo contract.

> **Note**. The Substrate hardware is at TRL 1–2. The translator tool is unbuilt as of April 2026. Everything below describes target contracts for Phase 2 work.

---

## Interface Position in Platform Architecture

```
┌──────────────────────────────────────────────────────────┐
│             brain-emulation (Interface)                   │
│  Brian2 SNNs · Atlas templates · Three.js visualization   │
└─────────────────────┬────────────────────────────────────┘
                      │  Atlas template (JSON)
                      │  Validation traces (spike statistics)
                      ▲
                      │  TC equilibrium samples
                      │  rendered as synthetic spike trains
                      ▼
┌──────────────────────────────────────────────────────────┐
│                 thermodynamic-core (Substrate)            │
│  p-bits · Langevin dynamics · HBSC couplings · DTM        │
└──────────────────────────────────────────────────────────┘
```

The link is offline (training) and visualization (post-hoc), not realtime. No realtime latency budget applies.

---

## 1. Paradigm Comparison

| Aspect | brain-emulation (Brian2) | thermodynamic-core (TC) | Contract implication |
|---|---|---|---|
| Neuron model | Biologically realistic (LIF, HH variants) | Langevin variable $x_i$ | Translator must collapse multi-compartment models to a single Langevin DOF, or mark them as unmappable |
| Dynamics | Deterministic ODEs (optionally noisy) | Stochastic Langevin SDE | Validation metric must compare distributions, not point trajectories |
| Synapse model | Conductance-based, STDP | Energy coupling $J_{ij}$ | Conductance dynamics not preserved; only mean coupling and sign |
| Learning | STDP, homeostatic plasticity | Onsager-Machlup gradient descent | Online STDP not portable; offline OM training only |
| Hardware target | CPU/GPU simulation | TSU (thermodynamic sampling unit) | Both can run on CPU/GPU as software reference for validation |
| Status | v1.0 feature-complete | TRL 2 research | Translator must run on a CPU-only TC reference (per [thermodynamic-core/sims/](https://github.com/Zae-Project/thermodynamic-core/tree/main/sims)) until hardware is available |

The STL neuristor described in [`thermodynamic-core/docs/architecture/neuromorphic-integration.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/architecture/neuromorphic-integration.md) bridges the determinism gap in dedicated hardware. Stochastic mode at input layers, deterministic mode at hidden and output layers. The translator tool described below does not need to know this; it produces $J_{ij}$ and $b_i$ that can run on either mode.

---

## 2. Mapping Contract: Brain Atlas → TC Energy Landscape

### 2.1 Atlas templates in brain-emulation

Catalogued in [`brain-emulation/data/brain_region_maps/manifest.json`](https://github.com/Zae-Project/brain-emulation/blob/main/data/brain_region_maps/manifest.json):

| Template id | Source | Regions |
|---|---|---|
| `Allen_Motor_Cortex` | Allen Human Brain Atlas | BA4 (M1, corticospinal L5B) |
| `Allen_Prefrontal_Cortex` | Allen Human Brain Atlas | BA10, BA46 |
| `Allen_Somatosensory_Cortex` | Allen Human Brain Atlas | BA3b |
| `BrainGlobe_Visual_Cortex` | BrainGlobe MNI152 | BA17, BA18 |
| `Julich_Thalamocortical_Loop` | EBRAINS Julich-Brain | MD thalamus, PFC |

An additional `basal_ganglia_action_selection.json` exists on disk but is not in the manifest registry as of April 2026. The translator should accept either registered or unregistered JSON templates.

### 2.2 Object-level mapping

| Brian2 object | TC primitive | Sign convention |
|---|---|---|
| Excitatory synaptic weight $w_{ij} > 0$ | Coupling $J_{ij} < 0$ | Lowers energy for aligned states |
| Inhibitory synaptic weight $w_{ij} < 0$ | Coupling $J_{ij} > 0$ | Raises energy for aligned states |
| Bias current $I_{\text{bias}, i}$ | Bias term $b_i$ in $V_\theta$ | Direct mapping |
| 80/20 excitatory/inhibitory ratio | Distribution of $J_{ij}$ signs | Preserve at network level |
| Clustered (laminar) connectivity | Local $J_{ij}$ entries | Dense submatrix per cluster |
| Long-range projections | HBSC rank-$k$ bilinear coupling | Avoids $O(D^2)$ wiring (see [`thermodynamic-core/docs/architecture/scaling-interconnects.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/architecture/scaling-interconnects.md)) |
| Neuron type (30+ variants) | Per-neuron $b_i$ and local coupling profile | Best-effort; multi-compartment models collapse to a single DOF |

---

## 3. Translation Pipeline (Phase 2 Target)

### 3.1 Implementation home

The canonical translator lives in `brain-emulation/` (per [`brain-emulation/CLAUDE.md`](https://github.com/Zae-Project/brain-emulation) "Phase 2 integration target"), not in `thermodynamic-core/`. Rationale. Atlas templates are owned by brain-emulation; the translator is a producer of TC-format payloads from atlas inputs. thermodynamic-core consumes the output and does not need to import Brian2.

### 3.2 Tool contract

| Property | Specification |
|---|---|
| Input | Brian2 `Network` object or atlas-template JSON path |
| Output | `(J, b, kT)` triple: $J_{ij}$ sparse matrix, $b_i$ vector, scalar temperature |
| API | `CouplingMatrix.from_brian2_network(network) -> (J, b, kT)` |
| Backend | NumPy reference (must run without TC hardware); JAX path optional |
| Validation criterion | Activation cosine similarity ≥ 0.99 against a Brian2 reference run on the same template (per Whitelam and Casert 2026 benchmark; see [`thermodynamic-core/docs/integration/with-consciousness-engine.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/integration/with-consciousness-engine.md)) |
| Failure mode | If any unmappable model is encountered (multi-compartment, gap junction, custom equations) the translator must raise a typed error with the offending object id, not silently degrade |

### 3.3 Sketch

```python
# Phase 2 target. Implementation lives in brain-emulation/.
from brain_emulation import BrainTemplate
from thermodynamic_core import CouplingMatrix

template = BrainTemplate.load("allen_motor_cortex")
J, b, kT = CouplingMatrix.from_brian2_network(template.network)
# J[i,j] = -w[i,j] if excitatory else +|w[i,j]|
# b[i]   = I_bias[i]
# kT     = temperature consistent with template's noise envelope
```

---

## 4. Visualization Interface

brain-emulation's existing Three.js front-end ([`brain-emulation/js/`](https://github.com/Zae-Project/brain-emulation)) consumes spike events over a WebSocket. TC hardware (and the TC software reference) emits equilibrium samples, not spike trains. Conversion:

| TC output | Conversion | Visualization payload |
|---|---|---|
| $x_i \in [-1, +1]$ (Ising-coded continuous variable) | $r_i = (x_i + 1)/2 \cdot r_{\max}$ | Rate-coded spike train at $r_i$ Hz |
| Binary Boltzmann sample ($x_i \in \{0, 1\}$ or $\{-1, +1\}$) | One spike event per $x_i = +1$ | Direct event |
| Continuous Langevin trajectory | Thresholded zero-crossing | Synthetic spike at each crossing |

The translator tool from §3 owns the conversion direction Brian2 → TC. The visualization owns the inverse direction TC → spike events. Both directions share the same neuron index space.

---

## 5. Shared Validation Benchmarks

| Benchmark | brain-emulation side | thermodynamic-core side | Comparison metric |
|---|---|---|---|
| MNIST digit classification | Brian2 LIF classifier | RBM / DTCA classifier (see [`thermodynamic-core/sims/boltzmann/`](https://github.com/Zae-Project/thermodynamic-core/tree/main/sims/boltzmann)) | Top-1 accuracy parity, target within 5% |
| Allen Motor Cortex activity | Realistic-mode Brian2 simulation of `Allen_Motor_Cortex` template | TC Langevin sampling on $(J, b, kT)$ from translator | Firing-rate distribution KL divergence; pairwise correlation matrix Frobenius distance |
| Split-brain protocol | Two coupled Brian2 hemispheres with corpus-callosum severance | Two coupled TC substrates with the same severance | Hemispheric independence statistic, in agreement with the protocol in [`neutral-consciousness-engine/ros2_ws/.../tests/split_brain_test.py`](https://github.com/Zae-Project/neutral-consciousness-engine) |

The split-brain benchmark is the same one the Engine pillar already runs on the neutral-consciousness-engine side. This contract requires the TC substrate to produce comparable hemispheric-independence statistics, not identical trajectories.

---

## 6. Performance Contract

The interface is not on the realtime brain-loop path. The [latency-budget-analysis.md](../architecture/latency-budget-analysis.md) RTT budget does not constrain this contract.

| Property | Target |
|---|---|
| Translation runtime | < 10 s for any single atlas template on CPU |
| Translator memory ceiling | < 4 GB peak (largest current template is `basal_ganglia_action_selection.json`, ~10k neurons; future templates may grow) |
| Validation run on Brian2 | < 5 min for `Allen_Motor_Cortex` realistic-mode |
| Validation run on TC reference (`thermodynamic-core/sims/`) | comparable wall time on CPU; no hardware required |

---

## 7. Security Extensions

Atlas templates are open scientific data; not sensitive. Derived coupling matrices $(J, b)$ become model parameters. If a $(J, b)$ pair derived from this translator is loaded into a production payload (TC hardware in orbit), it falls under [security-architecture.md](../architecture/security-architecture.md) §10.1 (parameter memory as high-value target) and the parameter-write contract in [thermodynamic-arkspace-interface.md](./thermodynamic-arkspace-interface.md) (AES-256 envelope plus cryptographic signature verified before NVM programming). The translator itself does not handle keys; signing happens downstream.

---

## 8. Technology Gaps

| Gap | Path forward |
|---|---|
| Translator tool not yet built | Phase 2 work in brain-emulation; canonical home per §3.1 |
| Unmappable Brian2 features (multi-compartment, gap junctions) | Define typed error classes; document the unmapped surface in the brain-emulation README once the tool lands |
| Spike-train ↔ TC sample conversion only conceptual | Pin a reference implementation in the visualization layer once the translator emits its first samples |
| Temporal-resolution mismatch | Brian2 default dt is 0.1 ms; TC Langevin timestep is set by the relaxation time of the underlying primitive. Reconcile in the translator output's `kT` and step-count metadata |
| Conductance-based fidelity | TC Boltzmann machines are simpler than HH-style neurons. Track which atlases require fidelity that TC cannot deliver and mark them out-of-scope for orbital deployment |

---

## 9. Open Questions

1. Translator ownership confirmation. Phase 2 brain-emulation/CLAUDE.md names this repo as the implementation home. Verify with the Engine and Substrate maintainers before code lands.
2. Atlas-template JSON schema versioning. The current manifest version is `2024-05-18`. No `schema_version` field exists per template. A versioning field will be needed if the translator output format changes in a way that's not backward compatible.
3. Benchmark scoring policy. The contract specifies firing-rate KL and pairwise-correlation Frobenius distance for the Allen Motor Cortex benchmark. Whether to report mean, p50, or p99 across multiple seeds is unresolved.
4. Translator caching. A translated $(J, b, kT)$ is deterministic given a template. Whether to cache and where (in `brain-emulation/data/translated/`? per-user?) is open.

---

## 10. Related Documentation

- TC-side spec: [`thermodynamic-core/docs/integration/with-brain-emulation.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/integration/with-brain-emulation.md)
- Sister Substrate specs: [thermodynamic-consciousness-interface.md](./thermodynamic-consciousness-interface.md), [thermodynamic-arkspace-interface.md](./thermodynamic-arkspace-interface.md)
- TC hardware primitives: [`thermodynamic-core/docs/architecture/hardware-primitives.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/architecture/hardware-primitives.md)
- TC neuromorphic integration: [`thermodynamic-core/docs/architecture/neuromorphic-integration.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/architecture/neuromorphic-integration.md)
- TC Onsager-Machlup training: [`thermodynamic-core/docs/algorithms/langevin-training.md`](https://github.com/Zae-Project/thermodynamic-core/blob/main/docs/algorithms/langevin-training.md)
- brain-emulation atlas templates: [`brain-emulation/data/brain_region_maps/manifest.json`](https://github.com/Zae-Project/brain-emulation/blob/main/data/brain_region_maps/manifest.json)
- Architecture overview: [unified-platform-architecture.md](../architecture/unified-platform-architecture.md)
- Security contract for derived parameters: [security-architecture.md](../architecture/security-architecture.md) §10.1
- Glossary: [technical-glossary.md](../glossary/technical-glossary.md)
- Bibliography: [bibliography.md](../reference/bibliography.md)
