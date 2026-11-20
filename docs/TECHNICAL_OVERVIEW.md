# Technical overview

This document describes **what** the Tandem prototype computes and **how** it computes it. It contains no source code.

## 1. The question and the event

**Question.** What is the probability that both ADB catastrophe bonds are wiped out in the same three-year programme term?

**Event.** Pr(L_K = USD 80m **and** L_T = USD 80m) over the three-year term, where L_K and L_T are the losses of the Kyrgyz and Tajik bonds. A bond is wiped out when its USD 70m earthquake cover pays at 100% **and** its USD 10m precipitation cover pays.

**Inputs.** Every input in the application carries one of four labels:
- *Published*: the prospectus supplements;
- *Model assumption*: the challenge brief's two-country model, §3b;
- *Scenario assumption*: one of the brief's five scenarios;
- *Derived*: computed from the inputs above.

The demonstration fixes one event, one term, the ADB base case, standard precision, and frozen seeds.

## 2. Exact model (the reference)

We transcribe the brief's model, steps 1–8:
- a **25-state continuous-time Markov chain** for earthquake payouts in quarter steps (0, 25, 50, 75, 100% of the earthquake cover) per country;
- **shared earthquake events** that move both countries, plus local events;
- **precipitation**: a shared rainfall trigger S and local triggers O_i, with r_i = S OR O_i;
- the term state over three years.

The model reproduces **all 25 reference values in the brief's Table 5**.

One subtlety matters. In the chain's generator, a transition from a capped state that lands back on the same state must be dropped. Otherwise the generator rows do not sum to zero, and the term metrics come out about 1% too high.

### Two exact identities behind the scalable loader
Both identities are proven and tested to about 1e-16.

1. **Horizon collapse.** Every stream is Poisson and every cap is monotone (add-then-cap, logical OR). So the capped three-year state has the same law as **one period of length three**, and no convolution over years is needed.
2. **Conditional independence with an exact latent variable.** Given the number N of shared earthquake events, the two countries' quarter-step totals are independent. Every shared event adds at least one step to both countries, so N ≥ 4 forces both to the cap. Therefore **E = min(N, 4)** is an exact latent variable.

## 3. Quantum state preparation (13 qubits)

| Register | Qubits | Meaning |
|---|---|---|
| Latent E | 3 | shared-earthquake count, capped at 4 |
| Kyrgyz steps | 3 | quarter steps of the earthquake cover |
| Tajik steps | 3 | quarter steps of the earthquake cover |
| Shared rain S | 1 | one rainfall event affecting both |
| Rain triggers r₁, r₂ | 2 | each country's precipitation cover |
| Objective | 1 | marks "both wiped out" |

- Each register is loaded conditionally on its parents with multiplexed RY rotations (Grover–Rudolph style).
- **Every rotation angle is a closed form of per-country tables**: Poisson probabilities and one 5-state chain per country. The loader never integrates the joint distribution, so the objection of Herbert (2021), that generic Grover–Rudolph loading costs as much as classical integration, does not apply. The circuit grows **linearly** with the number of countries.
- Circuits are compiled to {rz, sx, x, cx} with deterministic transpiler settings.

## 4. Circuit-integrity gate (runs before any estimate)

The exact values come from the Markov chain, independently of the circuit. If any check fails, the estimate is blocked, no record is created, and the diagnostics are kept.

| # | Check | Frozen run |
|---|---|---|
| 1 | The circuit reproduces the target probability (Aer statevector vs exact) | \|Δ\| = 1 × 10⁻¹⁸ |
| 2 | The circuit reproduces the joint wipe-out distribution: P(00), P(01), P(10), P(11) vs the exact per-bond and joint values | max \|Δ\| = 8 × 10⁻¹⁵ |
| 3 | The oracle marks exactly the wipe-out states | 10 marked states, all "both wiped out"; \|Δ\| = 0 |
| 4 | The Grover operator amplifies as theory predicts: gate-level A·Qᵏ for k = 1, 2, 3 vs sin²((2k+1)θ), plus the invariant-plane residual | × 9.0, × 25.0, × 49.0; residual 4 × 10⁻¹³ |
| 5 | After the run: amplitude estimation was genuinely used | k up to 804; 38,550 Grover iterations |

**Tamper tests** confirm that the gate catches bad circuits:
- a circuit that encodes the wrong model fails checks 1 and 2;
- an oracle that marks the wrong qubit fails check 3;
- a failed gate returns `blocked`, with no estimate and no record.

## 5. Amplitude estimation

- **IQAE** (Grinko et al. 2021) with Clopper–Pearson intervals, so the coverage guarantee is rigorous. It uses 30 shots per round. The headline value is the maximum-likelihood estimate over all rounds, restricted to the rigorous interval.
- **Query accounting.** One quantum query is one application of the state preparation A, so a round with Grover power k costs shots × (2k + 1) queries. One classical query is one simulated three-year path.
- **Certified invariant-plane executor (ideal simulation only).** The compiled Grover operator Q is applied on Aer to the good and bad components of A|0⟩. The residual outside their span is measured, and it must be below 1e-9; we observe 1e-13 to 1e-14. Outcomes are then sampled from the 2 × 2 restriction of the compiled circuit (the invariant-plane theorem of Brassard et al. 2002). This makes circuits millions of layers deep tractable to simulate without changing the statistics.
- **MLAE** (Suzuki et al. 2020) and a noise-aware variant (after Tanaka et al. 2021) are implemented for the depth-limited, noisy-hardware path; see [RESOURCE_PROFILE.md](RESOURCE_PROFILE.md) §4.

## 6. Independent checks

All checks use the same target, the same model and **the same disclosed query budget** as the quantum estimate:
- **exact reference**: the Markov chain, the brief's method;
- **IQAE run B** with a fresh seed;
- **Monte Carlo**;
- **cross-entropy importance sampling**, a tuned rare-event sampler.

A many-seed study at fixed budgets (8,192 and 65,536 queries) compares IQAE and Monte Carlo errors. It reports both the binomial formula and the measured value.

## 7. DCE: Decision, Confidence, Evidence

Every run becomes an immutable DCE record.

**Decision.** One of *Ready to use* (ACCEPT), *Rough guide* (REFINE) or *Not reliable yet* (HELD). It says only whether the estimate meets the stated tolerance for this model's target. Whether the model itself is adequate is a separate, human judgement.

**Confidence.** A calibrated probability that the estimate is within tolerance:

```
confidence = P(within tolerance) × integrity × corroboration × (0.8 + 0.2 × provenance)
```

- **Precision** comes from the interval half-width and the method's measured interval coverage.
- **Reliability** is learned from verified past runs of the same method.
- **Integrity** comes from the circuit-integrity gate.
- **Provenance** depends on whether the inputs are exactly as published.
- **Corroboration** counts how many independent estimates agree within the joint 95% band. **The exact value is never an input**, so the confidence would work the same way where no exact answer exists.

**Evidence.** Record ID, run ID, input hash, model fingerprint, circuit SHA-256, seeds, budget, software commit and package versions, and the result of every check.

**Calibration** was measured on held-out seeds: 1,584 verified records, with an expected calibration error of 0.017 and a Brier skill of 53% over the base-rate guess. Of the results marked *Ready to use*, 98.8% were within tolerance. **Caveat:** 88.7% of the held-out estimates were within tolerance anyway, so the test set may be too easy. The calibration is measured on the ADB benchmark targets only.

**Records can be:**
- reproduced: re-run from the stored request and compared output by output;
- reviewed under a four-eyes rule;
- exported as JSON or Markdown;
- printed as a report generated only from the stored evidence.

## 8. What drives the joint tail (worst-case sampling)

The application uses the same frozen model, conditioned on "both wiped out".
- It applies amplitude amplification, A·Qᵏ with k = 139 (chosen from the run's own IQAE estimate), runs the circuit gate by gate on Aer, and measures all 13 qubits.
- It draws 1,000 wipe-out scenarios at **279 queries each**. Rejection sampling would need about **31,613 tries each**, so this is 113× fewer per scenario drawn. The one-off IQAE estimate is extra, and it is already paid for in the run.
- Sampled fractions are checked against the exact conditional fractions read from the loader statevector. The total-variation distance is 0.0265; a perfect sampler of this size would give about 0.0195. The χ² test gives p = 0.43.

This is model-conditioned evidence about the model's joint tail, not a forecast of a specific disaster.

## 9. Architecture

```
 Browser (React + TypeScript)
   │  one page, seven chapters; Records; printable report
   ▼
 Evidence gateway (Node.js) ──► MongoDB evidence store
   │  auth and roles (analyst, reviewer, auditor), immutable runs, DCE ledger,
   │  four-eyes review, reproduce-from-record, report export, background jobs
   ▼
 Quantum service (Python, FastAPI)
      exact model · loader · integrity gate · IQAE · classical samplers ·
      DCE scoring · worst-case sampling      on Qiskit 2.5 / Qiskit Aer 0.17
```

Details of the components and their interfaces: [../prototype/README.md](../prototype/README.md).

## 10. Freeze and reproducibility

- **Freeze.** The frozen request runs three times in-process, and the outputs must be bit-identical. The freeze then records:
  - the SHA-256 fingerprint of the 11 solver files;
  - the request;
  - the expected outputs.

  Every later run is compared with the freeze. The application shows a "Frozen demo build" badge only while the fingerprint matches.
- **Reproduce from record.** The stored request is re-run and 8 stored outputs are compared: integrity, record ID, decision, every estimate and every interval.
- **Same request, same seed.** Repeating a run stores the evidence once (content-addressed) but gives a new run ID each time.

## 11. Beyond the demonstration (built and measured in Phase 1)

The demo answers one question. The underlying engine also covers:
- the full benchmark: 5 scenarios × 5 targets, against six classical methods;
- an **n-country** generalisation with a strong exact classical baseline: a conditional-independence recursion whose cost grows as 10^pathwidth;
- a **cascading multi-hazard** extension: earthquakes triggering landslides and glacial-lake floods, with a 21-qubit circuit;
- noise-aware estimation and a **fault-tolerant resource estimate**.

See [RESULTS_AND_VALIDATION.md](RESULTS_AND_VALIDATION.md) and [RESOURCE_PROFILE.md](RESOURCE_PROFILE.md). Parameters without a published calibration (the cascade probabilities, and countries beyond the two calibrated ones) are labelled **illustrative** wherever they appear.

## References

- Brassard, G., Høyer, P., Mosca, M., Tapp, A. (2002). Quantum amplitude amplification and estimation. *Contemporary Mathematics* 305.
- Chakrabarti, S., et al. (2021). A threshold for quantum advantage in derivative pricing. *Quantum* 5, 463.
- Egger, D. J., García Gutiérrez, R., Cahué Mestre, J., Woerner, S. (2021). Credit risk analysis using quantum computers. *IEEE Trans. Computers* 70(12).
- Grinko, D., Gacon, J., Zoufal, C., Woerner, S. (2021). Iterative quantum amplitude estimation. *npj Quantum Information* 7, 52.
- Grover, L., Rudolph, T. (2002). Creating superpositions that correspond to efficiently integrable probability distributions. arXiv:quant-ph/0208112.
- Herbert, S. (2021). No quantum speedup with Grover–Rudolph state preparation for quantum Monte Carlo integration. *Phys. Rev. E* 103, 063302.
- Marshall, A. W., Olkin, I. (1967). A multivariate exponential distribution. *J. American Statistical Association* 62.
- Montanaro, A. (2015). Quantum speedup of Monte Carlo methods. *Proc. Royal Society A* 471.
- Rubinstein, R. Y., Kroese, D. P. (2004). *The Cross-Entropy Method*. Springer.
- Suzuki, Y., et al. (2020). Amplitude estimation without phase estimation. *Quantum Information Processing* 19, 75.
- Tanaka, T., et al. (2021). Amplitude estimation via maximum likelihood on noisy quantum computer. *Quantum Information Processing* 20, 293.
- Woerner, S., Egger, D. J. (2019). Quantum risk analysis. *npj Quantum Information* 5, 15.
