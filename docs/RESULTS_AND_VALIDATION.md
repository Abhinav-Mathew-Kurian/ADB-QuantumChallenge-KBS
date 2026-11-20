# Results and validation

All results come from **ideal state-vector simulation on Qiskit Aer**, except the noise study in [RESOURCE_PROFILE.md](RESOURCE_PROFILE.md) §4. Machine-readable versions of the tables are in [`../evidence/sample-results/`](../evidence/sample-results/).

**Claim boundary.**
- The comparison is accuracy **per query**, not wall-clock time.
- For two countries the exact classical calculation is the fastest method.
- No quantum hardware was used, and no quantum advantage is claimed.
- The results are statements about the model, not forecasts.

---

## 1. The demonstration question: joint wipe-out (frozen run)

Request: ADB base case, standard precision, seed 0 (run A), seed 1 (run B). Frozen 2026-11-11, workflow version `BP15-joint-wipeout-1`.

### 1.1 Estimate and independent checks (same model, same target, same budget)

| Role | Method | Estimate | 95% interval | Queries |
|---|---|---|---|---|
| Reference | Exact (Markov chain) | **3.163 × 10⁻⁵** | — | — |
| Estimate | IQAE, run A | **3.159 × 10⁻⁵** | 2.950–3.338 × 10⁻⁵ | 77,370 |
| Repeat | IQAE, run B (fresh seed) | 3.114 × 10⁻⁵ | 2.861–3.409 × 10⁻⁵ | 56,580 |
| Check | Monte Carlo | 0 (none observed) | 0–4.96 × 10⁻⁵ | 77,370 |
| Check | Cross-entropy importance sampling | 3.712 × 10⁻⁵ | 2.170–5.254 × 10⁻⁵ | 77,370 |

- The exact value lies inside run A's interval.
- Run B agrees with run A within 1.4%.
- Seeing no wipe-out at all at this budget happens to Monte Carlo in about 9% of runs. "Not observed" is not "zero risk". The frozen Monte Carlo seed is fixed by formula (11000 + seed A), not picked.
- 1 ÷ estimate ≈ **1 in 31,660 three-year terms**; 1 ÷ exact = 1 in 31,613.

### 1.2 Many seeds at fixed budgets

IQAE stops before exceeding the budget, so it usually uses fewer queries; the mean is shown. Monte Carlo always uses the full budget.

| Budget | IQAE relative RMSE (20 seeds) | IQAE mean queries used | Monte Carlo relative RMSE (measured, 200 runs) | (binomial formula) | Monte Carlo runs with zero wipe-outs (measured) | (formula) |
|---|---|---|---|---|---|---|
| 8,192 | **49.4%** | 3,040 | **189.0%** | 196.4% | 76.5% | 77.2% |
| 65,536 | **7.0%** | 32,828 | **66.9%** | 69.5% | 15.0% | 12.6% |

### 1.3 What drives the joint tail

These results come from 1,000 worst-case scenarios drawn by amplitude amplification on the same frozen model (seed 21), at 279 queries per scenario. Rejection sampling would need about 31,613 tries per scenario.

| Composition of joint wipe-outs | Sampled | Exact |
|---|---|---|
| Shared quake **and** shared rain | 45.5% | 42.9% |
| Shared rain, separate quakes | 44.4% | 45.7% |
| Shared quake, separate rain | 4.5% | 5.5% |
| No shared event (separate disasters) | 5.6% | 5.9% |
| **One rainfall event triggered both rain covers** | **89.9%** | **88.5%** |
| **One earthquake triggered both earthquake covers** | **50.0%** | **48.4%** |

The sampled distribution passes the check against the exact one: total-variation distance 0.0265 (about 0.0195 expected for a perfect sampler of this size), χ² p = 0.43.

### 1.4 DCE record of the frozen run

| Part | Value |
|---|---|
| Decision | **Ready to use** (ACCEPT) |
| Confidence | **0.80** = precision 1.000 × integrity 1.000 × corroboration 0.800 × (0.8 + 0.2 × provenance 1.000) |
| Evidence | record `8008ac7f5b8c6173a536`; circuit SHA-256 `ca26f8af…e2e5c7`; solver fingerprint `992ff345…918e8f75`; seeds; budget; package versions |
| Reproduced | 3 in-process runs identical; "Reproduce from record" gives all 8 stored outputs identical |

---

## 2. The full ADB benchmark: 5 scenarios × 5 targets

Setup:
- equal query budgets, 20 seeds per cell;
- each cell shows the relative RMSE of the **best of six classical methods** divided by IQAE's relative RMSE. The six are the brief's Monte Carlo, Sobol, stratified sampling and importance sampling, plus two stronger samplers we built: cross-entropy IS and structured IS;
- IQAE's 95% interval covered the exact value in **500 of 500** runs.

| Scenario | Expected loss | Both pay | Loss > USD 80m | Stop-loss | Both exhausted |
|---|---|---|---|---|---|
| adb-base | 1.45× | 3.45× | 3.70× | 3.66× | 3.09× |
| adb-independent | 1.42× | 4.60× | 3.07× | 4.51× | 1.29× |
| adb-common-shock | 1.23× | 3.60× | 2.96× | 2.66× | 2.27× |
| adb-rain-stress | 2.31× | 5.63× | 3.23× | 2.66× | 5.52× |
| adb-rain-cover | 1.57× | 2.45× | 1.75× | 2.72× | 3.38× |

IQAE is more accurate per query than the best classical method in **all 25 cells**, by 1.2–5.6×. Per-method errors are in [`benchmark-25-cells-equal-budget.csv`](../evidence/sample-results/benchmark-25-cells-equal-budget.csv). In that file, `joint_attachment` means "both pay" and `loss_exceedance` means "loss > USD 80m".

**Three countries** (20-qubit circuit, regional dependence):

| Target | IQAE | Structured IS | Monte Carlo | Ratio |
|---|---|---|---|---|
| Both exhausted | 0.89% | 2.86% | 33.5% | 3.2× |
| ≥ 2 countries pay | 0.34% | 3.71% | 3.76% | 10.8× |

**Cascading multi-hazard extension** (21 qubits, illustrative parameters):
- IQAE is more accurate per query than the best of five classical methods in **18 of 19 cells**.
- On the tail and joint targets the margin is 1.7–7.4×.
- The one cell it does not win is a smooth expected-loss target at a small budget (0.98×).
- Interval coverage was 379 of 380.

---

## 3. DCE calibration (held-out seeds)

Setup:
- **1,584 verified records**: 25 benchmark cells and 19 cascade cells × 3 budgets × 4 seeds × 3 methods (IQAE, Monte Carlo, cross-entropy IS);
- tolerance ±10%;
- seeds never used while designing the confidence formula.

| Measure | Value |
|---|---|
| Brier skill over the base-rate guess | **53%** |
| Expected calibration error | **0.017** |
| *Ready to use* results within tolerance | **98.8%** (1,190 of 1,205) |
| *Rough guide* results within tolerance | 78.8% (175 of 222) |
| *Not reliable yet* results within tolerance | 25.5% (40 of 157) |
| Share of estimates accepted at equal budget | IQAE 95.5%, cross-entropy IS 85.0%, Monte Carlo 47.7% |

**Caveat.** The base rate is 88.7%: most held-out estimates were within tolerance anyway, so this test set may be too easy. The calibration holds for the ADB benchmark targets; it is not a guarantee for other models or data.

---

## 4. Validation

| Check | Result |
|---|---|
| Brief model vs the brief's Table 5 | all 25 values match to printed precision |
| Organisers' public benchmark repository | 25/25 public reference values match to ≤ 1.6 × 10⁻¹³ |
| n-country model at n = 2 vs the brief model | agreement to 5 × 10⁻¹⁴ |
| Two independent exact n-country solvers | agree to ≤ 10⁻¹⁴ |
| Two independent exact cascade solvers | agree to 3.3 × 10⁻¹⁶; with cascades switched off, they equal Table 5 to 10⁻¹² |
| Circuit amplitude vs exact target | ≤ 10⁻¹⁵ relative (brief), ≤ 10⁻¹⁴ (n = 3), ≤ 10⁻⁹ (cascade); MPS up to 35 qubits: 10⁻⁸ |
| Certified Grover-plane residual | 10⁻¹³ to 10⁻¹⁴ |
| IQAE interval coverage | 500/500 (benchmark cells), 379/380 (cascade cells) |
| Circuit-integrity gate tamper tests | wrong model blocked; wrong oracle blocked; blocked runs produce no record |

### Automated tests at the freeze

| Suite | Result |
|---|---|
| Python (model, circuits, estimators, DCE, API) | 443 / 443 |
| Evidence gateway | 19 / 19 |
| UI unit tests | 15 / 15 |
| Browser end-to-end checks (accessibility and phone width included) | 67 / 67 on consecutive runs |
| Live integration check | 18 / 18 |
| Pre-recording check: three frozen runs through the full stack | 18 / 18 ([`pre-recording-checks.csv`](../evidence/sample-results/pre-recording-checks.csv)) |

### Mistakes caught by measurement (and fixed)

Each one was fixed at its cause and covered by a regression test:
- a Markov-chain self-loop bug that inflated term metrics by about 1%;
- a mis-specified noise model: the "good" fraction g = ½ assumed by the literature, where our circuits need g = ¼;
- an importance-sampling weight collapse that made a classical baseline look absurdly fast;
- a test that asserted folklore ("dependence always raises Pr(≥ 2 pay)"), which is false for n ≥ 4;
- a wrong oracle that crashed the plane certification instead of failing the integrity check.

## 5. What the prototype does not do yet

- **No quantum hardware runs.** Everything except the noise study is ideal simulation.
- **No wall-clock advantage.** See the fault-tolerant estimate in [RESOURCE_PROFILE.md](RESOURCE_PROFILE.md) §5.
- **Model-conditioned.** Results depend on the brief's model structure and the published hazard tables.
- **Illustrative parameters** in the extensions (cascade probabilities, countries beyond the two calibrated ones) are labelled as such and are not calibrated to data.
