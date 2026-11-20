# Resource profile

This document covers the quantum and classical resources: qubits, circuit depth, two-qubit gates, shots and queries, error mitigation, the fault-tolerant estimate, and the mapping to a hardware modality.

**Conventions.**
- Gate counts are for circuits compiled with Qiskit 2.5.2: basis {rz, sx, x, cx}, optimisation level 1, fixed transpiler seed.
- A = state preparation; Q = Grover iterate = S_χ A† S₀ A.
- One quantum query is one application of A.

## 1. Summary

| Stage | Quantum resource | Classical resource | Status |
|---|---|---|---|
| **Phase 1 prototype** | Ideal state-vector simulation on Qiskit Aer: 13 qubits (demo and benchmark model), 16–24 qubits (n = 3 portfolios), 21 qubits (cascade model); MPS checks up to 35 qubits | One laptop: 6-core AMD Ryzen 7, 9 GB RAM, Linux under WSL2, no GPU | **Measured** |
| **Phase 2** (GPU credits, hardware sandbox) | State vector to about 30–34 qubits on GPU; MPS / tensor-network validation of n-country circuits (269–718 qubits at n = 64); noisy simulation and shallow MLAE on hardware | GPU nodes for simulation; CPU for exact recursions and classical baselines | Planned ([PHASE_2_PLAN.md](PHASE_2_PLAN.md)) |
| **Fault-tolerant execution** | 256–4,952 logical qubits; 0.6–97 M physical qubits (surface code), depending on portfolio, target and modality | — | **Estimated**: no wall-clock advantage at current projections; break-even conditions stated (§5) |

## 2. The demonstration circuit (n = 2, joint wipe-out)

**13 qubits:**
- 3 for the latent shared-earthquake count;
- 3 + 3 for the quarter steps;
- 1 for shared rain;
- 2 for the rain triggers;
- 1 objective.

**Gate counts per benchmark target:**

| Target | A depth | A two-qubit | A one-qubit | Q depth | Q two-qubit | Q one-qubit |
|---|---|---|---|---|---|---|
| Expected loss | 1,754 | 378 | 1,496 | 4,955 | 1,472 | 4,258 |
| Both pay | 1,778 | 378 | 1,520 | 4,998 | 1,472 | 4,301 |
| Loss > USD 80m | 1,778 | 378 | 1,520 | 4,998 | 1,472 | 4,301 |
| Stop-loss | 1,778 | 378 | 1,520 | 5,002 | 1,472 | 4,305 |
| **Both exhausted (joint wipe-out)** | 1,778 | 378 | 1,520 | 4,998 | 1,472 | 4,301 |

**Frozen demo run** (joint wipe-out, standard precision):
- 77,370 queries;
- 9 IQAE rounds of 30 shots;
- maximum Grover power k = 804;
- 38,550 Grover iterations in total.

**IQAE statistics per target** (ADB base case, 95% confidence, 30 shots per round, median of 20 seeds):

| Target | ε (amplitude) | Rounds | Shots | Queries | Max Grover power k | Deepest circuit (layers) | Two-qubit gates executed |
|---|---|---|---|---|---|---|---|
| Expected loss | 5e-4 | 9 | 270 | 19,905 | 209 | 1.03 M | 14.5 M |
| Both pay | 1e-3 | 8 | 240 | 10,290 | 108 | 0.54 M | 7.5 M |
| Loss > USD 80m | 3e-4 | 8 | 240 | 18,450 | 199 | 1.00 M | 13.5 M |
| Stop-loss | 5e-5 | 8.5 | 255 | 42,600 | 456 | 2.28 M | 31.3 M |
| Both exhausted | 3e-6 | 9 | 270 | 74,835 | 794 | 3.97 M | 55.0 M |

**Reading the table.** The deepest circuits are millions of layers deep, so they cannot run on today's noisy devices. That leaves two routes:
- **near term:** depth-capped MLAE with error mitigation (§4), a demonstration path rather than an advantage path;
- **advantage path:** fault-tolerant execution (§5).

**Worst-case sampling** uses the same 13-qubit circuit with k = 139: 279 state preparations per tail scenario. It runs gate by gate on Aer, up to about 2.5 M gates.

## 3. Scaling to n countries

The n-country circuit follows the template of Egger et al. (2021):
- shock registers;
- per-country loaders;
- a QFT adder with a folded comparator;
- a counter.

Target: Pr(at least 2 bonds exhausted).

| n | Regional: qubits / two-qubit / depth | Chain: qubits / two-qubit / depth | Grid: qubits / two-qubit / depth |
|---|---|---|---|
| 2 | 16 / 193 / 602 | 16 / 193 / 602 | 16 / 193 / 602 |
| 4 | 25 / 456 / 1,162 | 33 / 1,256 / 5,012 | 37 / 2,050 / 6,754 |
| 8 | 42 / 1,004 / 2,212 | 66 / 3,404 / 13,762 | 82 / 3,602 / 8,127 |
| 16 | 75 / 2,192 / 4,311 | 131 / 7,792 / 31,261 | 172 / 9,194 / 20,175 |
| 32 | 140 / 4,872 / 8,417 | 260 / 16,872 / 66,167 | 349 / 19,242 / 41,081 |
| 64 | 269 / 10,736 / 16,575 | 517 / 35,536 / 135,945 | 718 / 40,314 / 83,031 |

Qubits and two-qubit gates grow **linearly in n** for every dependence topology. That includes the map-like grid, where the strongest exact classical method (a conditional-independence recursion costing 10^pathwidth) becomes infeasible from n ≈ 8: it needs 1.85 GB at n = 8, 277 GB at n = 12, and about 15 PB at n = 64.

**Cascade circuit** (n = 2):
- 21 qubits;
- per target, A has about 3,660 layers and 774–786 two-qubit gates, and Q has about 12,700 layers and 3,864–3,888 two-qubit gates.

For n countries, cascades add 2n + (number of propagation edges) qubits.

## 4. Error mitigation (measured)

**Method.** Noise-aware MLAE, following Tanaka et al. (2021). It fits P(k) = g + λe^(−κk)(sin²((2k+1)θ) − g), with g = 2^(−objective qubits). This generalises the published g = ½ case.

**Setup:**
- depolarising noise on one- and two-qubit gates, plus readout error;
- gate-level Aer simulation;
- maximum power 16, 100 shots per power, 20 seeds.

| Two-qubit error | Exponential schedule: raw → mitigated relative RMSE | Linear schedule: raw → mitigated |
|---|---|---|
| 0 | 8.3% → 8.4% | 1.6% → 1.6% |
| 0.1% | 17.9% → **10.8%** | 1.5% → 1.3% |
| 0.3% | 26.5% → **11.5%** | 1.9% → 1.5% |
| 1% | 37.0% → **12.8%** | 2.5% → 2.6% |

- Mitigation never hurts materially, and it removes up to **2.9×** of the noise-induced error.
- It does not make deep IQAE circuits feasible on noisy hardware.

## 5. Fault-tolerant estimate and hardware modality

**Method:** Chakrabarti et al. (2021).
- **Oracle calls:** IQAE bound N ≤ (1.4/ε)·ln((2/α)·log₂(π/4ε)), with α = 0.05.
- **Error budget:** 90% to estimation, 10% to rotation synthesis.
- **Rotation synthesis:** Ross–Selinger.
- **Surface code:** the model of Beverland et al. (2022).
- **FT-compiled model:** the same circuit structure costed with standard fault-tolerant primitives (QROM multiplexers, phase-gradient rotations, Gidney adders, AND-tree reflection). It is a model built from published primitive costs, not a compiled circuit.

**Results at 1% relative precision.** Superconducting at p = 1e-3. The classical figure is the best valid classical method measured on our machine; importance sampling counts only when its effective sample size is at least 1%.

| Portfolio, target | Amplitude | Oracle calls | Logical qubits | Code distance | Physical qubits | Quantum time | Best classical | Quantum ÷ classical |
|---|---|---|---|---|---|---|---|---|
| n = 2, both exhausted | 3.2e-5 | 3.3e7 | 276 | 27 | 3.8 M | 3.8e5 s | 2.2 s | 1.7e5× |
| n = 2, loss > threshold | 1.2e-2 | 8.1e4 | 256 | 23 | 4.1 M | 670 s | 0.05 s | 1.3e4× |
| n = 8 regional, both exhausted | 5.6e-4 | 1.8e6 | 412 | 25 | 10.8 M | 1.8e4 s | 2.1 s | 8.4e3× |
| n = 16 grid, both exhausted | 1.1e-3 | 9.3e5 | 1,192 | 27 | 23.5 M | 2.6e4 s | 33 s | 776× |
| n = 64 regional, both exhausted | 1.5e-2 | 6.6e4 | 2,228 | 23 | 78.9 M | 590 s | 6.2 s | 96× |
| n = 64 grid, both exhausted | 9.8e-3 | 9.9e4 | 4,952 | 25 | 96.6 M | 2,550 s | 14 s | 177× |

**Sensitivity to the modality** (n = 64 regional, both exhausted):

| Modality | Physical qubits | Quantum time | Quantum ÷ classical |
|---|---|---|---|
| Superconducting, p = 1e-3 | 78.9 M | 590 s | 96× |
| **Superconducting, p = 1e-4** | **16.2 M** | **282 s** | **46×** |
| Trapped ions / neutral atoms, p = 1e-3 | 38.1 M | 8.9e5 s | 1.4e5× |
| Trapped ions / neutral atoms, p = 1e-4 | 11.1 M | 4.2e5 s | 6.9e4× |

**Recommended modality:** superconducting, or another fast-cycle platform, at p ≤ 1e-4 with a surface code.
- Amplitude estimation is a long, sequential, depth-dominated computation, so logical cycle time dominates its cost.
- Slower-cycle platforms lose by roughly the ratio of their cycle times, even with better fidelity and connectivity.

**Break-even** (n = 64, both exhausted). Quantum matches the best classical method if **either**:
1. the logical step is 50–100 ns, about 100–200× faster than current projections; **or**
2. one classical scenario costs at least 0.2–1 ms. A full physical catastrophe model (hazard footprints, vulnerability curves) can cost that much per scenario. Its quantum oracle would be heavier too, and costing that is a Phase 2 task.

The gap closes by about **1,800×** from n = 2 to n = 64.

## 6. Classical compute (measured, same machine)

| Method | Cost |
|---|---|
| Brief's exact method (n = 2) | ≈ 0.2 ms |
| Exact cascade model (n = 2), two methods | ≈ 30 ms |
| Full Markov-chain enumeration, n countries | 10ⁿ states (practical to n ≈ 5) |
| Exact conditional-independence recursion, n = 64 | regional 0.18 s; chain 1.3 s; grid infeasible from n = 8 |
| Monte Carlo per three-year path | 0.14 µs (n = 2) to 2.4–3.3 µs (n = 64) |
| Structured importance sampling per path | ≈ 0.24 µs (n = 2); the product form collapses at large n |

## 7. Environment

| Item | Value |
|---|---|
| Machine | AMD Ryzen 7 7445HS (6 cores), 9 GB RAM, Windows 11 / WSL2 (Linux 6.6); no GPU |
| Python | 3.12.3 |
| Quantum stack | qiskit 2.5.2, qiskit-aer 0.17.2, numpy 2.5.3, scipy 1.18.1 (see [`../requirements.txt`](../requirements.txt)) |
| Service stack | FastAPI 0.142.2, pydantic 2.13.5, uvicorn 0.54.0; Node.js gateway with MongoDB; React + TypeScript front end |
| Simulators | Aer state vector (gate-level, noise models); Aer MPS (large-circuit validation); certified invariant-plane executor for deep ideal runs |
| Frozen demo run | about 65 s for the full freeze (three identical runs, worst cases, budget study) on the laptop |
