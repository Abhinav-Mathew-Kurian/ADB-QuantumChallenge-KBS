# Phase 2 plan

Phase 1 delivered a working two-country use case and a reproducible quantum-estimation and independent-check workflow. Phase 2 turns that into a validated prototype along four lines, the same four shown in the closing chapter of the demo:

| Step | What | Why |
|---|---|---|
| **1. Hardware execution** | Run the same 13-qubit circuits on quantum hardware under a declared noise model, with error mitigation. Report the gap to the ideal result. | Every Phase 1 number is ideal simulation. |
| **2. Data calibration** | Replace brief-level hazard inputs with event-level data (World Bank event-loss tables), and calibrate the shared-event structure. | The results are conditional on the brief's model. |
| **3. Larger portfolios** | Validate the multi-country circuit beyond two countries, where exact enumeration stops being feasible. | The scaling argument is a hypothesis until it is measured at scale. |
| **4. User evaluation** | Test the workflow and the DCE record with ADB risk and treasury staff. | The point of the product is a decision-ready record, so its users have to judge it. |

**Timing.** The plan is written in relative weeks (W1–W12) and will be aligned with the official Phase 2 schedule.

## 1. Measurable objectives

| # | Objective | Success measure |
|---|---|---|
| O1 | **Hardware run** of the 13-qubit circuits through the provided sandbox, with provider job IDs | Error vs the exact value, with and without mitigation; honest report of the ideal-vs-hardware gap |
| O2 | **Scale the evidence:** end-to-end amplitude estimation at n = 3–5 by GPU state vector; amplitude validation to n = 64 by MPS | IQAE relative RMSE vs the best classical method at equal budget, per n and topology; amplitude error ≤ 1e-8 |
| O3 | **Cascades at portfolio scale:** an n-country cascade model (propagation along river and glacier graphs), its exact recursion and its circuit | Two exact methods agree; circuit amplitude equals the exact value; frontier width vs n measured |
| O4 | **The break-even question:** cost a heavier, realistic per-scenario oracle (hazard footprint + vulnerability) on both the classical and the quantum side | A break-even surface (logical step time × classical cost per scenario) with measured points |
| O5 | **Compile the fault-tolerant model:** QROM loaders, Gidney adders and phase-gradient rotations as real circuits | FT-compiled counts reproduced from circuits within ±20% |
| O6 | **Noisy-hardware path:** noisy simulation of the 13-qubit circuit; shallow MLAE with readout mitigation and zero-noise extrapolation | Error vs noise level, with and without mitigation |
| O7 | **DCE in use:** every estimate (API, application, report) arrives as an Evidence Object with learned reliability and calibration; harder held-out sets | Brier skill > 0 per method; *Ready to use* results within tolerance ≥ 95% on a test set with a lower base rate |
| O8 | **Calibrate the illustrative parameters** (cascade probabilities, shared-event structure) from public inventories and event-loss data | Parameters with uncertainty bands; provenance raised from "illustrative" |

## 2. Work packages

| WP | Weeks | Content | Output |
|---|---|---|---|
| WP1 Benchmark parity | Done in Phase 1 | Reproduced the organisers' public benchmark repository: 25/25 reference values match to ≤ 1.6 × 10⁻¹³; ran both entries in the official evaluation harness on private hold-out suites | Parity report |
| WP2 Platform | Done in Phase 1 | Quantum service, evidence gateway with immutable store, the seven-chapter web application, DCE records, printable reports, freeze and reproduce-from-record | Working prototype |
| WP3 Hardware execution | W2–W8 | O1, O6: noisy simulation first, then hardware runs, mitigation, and the gap report | Hardware evidence |
| WP4 Data calibration | W2–W8 | O8: event-level data, shared-event calibration, a data note with provenance in every DCE record | Calibrated model v2 |
| WP5 GPU scale-out | W3–W8 | O2, O3: GPU state vector to about 34 qubits, MPS to n = 64, strong classical baselines at every n | Scaling evidence |
| WP6 Fault tolerance and break-even | W5–W10 | O4, O5 | Updated resource estimate |
| WP7 User evaluation | W6–W10 | O7: sessions with ADB risk and treasury staff on the workflow and the DCE record; changes from their feedback | Evaluation note and revised workflow |
| WP8 Documentation and business case | W9–W12 | Technical report, updated classical bottleneck analysis, business case, final demonstration | Final Phase 2 package |

## 3. Business case (outline)

**Product.** A joint-tail risk engine for multi-country parametric programmes. It does exact, quantum and classical estimation behind one interface, and every number arrives as a reviewable DCE record that a risk committee can audit.

**Users:**
- development banks' disaster-risk-finance teams;
- sovereign risk pools;
- catastrophe-bond structurers and reinsurers;
- ratings and modelling firms.

**Value now (classical mode):**
- auditable confidence on rare-tail numbers;
- independent checks at equal budget;
- the "do triggers pay when real losses occur" (basis-risk) analysis;
- scenario sweeps.

None of this waits for quantum hardware.

**Value later (quantum mode).** The same interface switches to amplitude estimation once fault-tolerant hardware crosses the break-even quantified in [RESOURCE_PROFILE.md](RESOURCE_PROFILE.md) §5. The DCE record already carries the per-method query comparison that shows when that happens.

## 4. Risks and mitigations

| Risk | Mitigation |
|---|---|
| No wall-clock advantage at realistic hardware projections | Already quantified. Phase 2 targets the break-even lever (a heavier oracle) and reports honestly either way. The product keeps standalone value in classical mode. |
| Deep circuits exceed what noisy hardware can run | Shallow MLAE schedules with mitigation for the hardware demonstration; deep runs stay in simulation and are labelled as such |
| GPU memory limits state-vector size | MPS for the structured circuits; the certified-plane executor needs only state-vector passes |
| Hazard parameters cannot be calibrated tightly | Report scenario bands with DCE provenance; the sensitivity sweeps are already built |
| DCE calibration test set too easy (base rate 88.7%) | Build harder held-out sets (lower budgets, new scenarios), and report calibration per difficulty band |

## 5. Resources requested

- **Compute:** 1–8 GPUs for simulation at scale; access to the hardware sandbox for the 13-qubit runs.
- **Data:** event-loss tables and public hazard inventories (landslides, glacial-lake outburst floods).
- **People:** the team, plus time with ADB risk and treasury staff for the user evaluation.
