# Sample results

These files were exported by script from the evidence of the frozen build. Nothing in them was typed by hand. All results are ideal simulation on Qiskit Aer.

| File | What it holds |
|---|---|
| `joint-wipeout-frozen-run.json` | The frozen demo run: the request, the solver fingerprint, the estimate, the independent checks, the decision and confidence, the worst-case composition, the budget study, and the software versions |
| `joint-wipeout-equal-budget-checks.csv` | Exact reference, IQAE runs A and B, Monte Carlo and cross-entropy importance sampling at the same 77,370-query budget |
| `joint-wipeout-budget-study.csv` | Many-seed study at 8,192 and 65,536 queries: IQAE vs Monte Carlo, measured alongside the binomial formula |
| `worst-case-composition.csv` | 1,000 amplitude-amplified worst-case scenarios: sampled vs exact conditional fractions |
| `headline-figures-check.csv` | Each headline figure used in the film, checked against the frozen run, with its exact meaning and denominator |
| `benchmark-25-cells-equal-budget.csv` | 5 scenarios × 5 targets: IQAE relative RMSE and coverage vs six classical methods at equal budget (20 seeds) |
| `dce-calibration-summary.json` | DCE calibration on 1,584 held-out records: Brier skill, results per decision, results per method, and the caveat |
| `pre-recording-checks.csv` | The 18 checks run through the full stack before the film was recorded |

**Target names in the CSVs:**

| Name | Meaning |
|---|---|
| `expected_loss` | expected loss over the term |
| `joint_attachment` | both bonds pay something |
| `loss_exceedance` | combined loss above USD 80m |
| `stop_loss` | expected combined loss above the stop-loss threshold |
| `both_exhausted` | both bonds wiped out (the demo's question) |

**Relative RMSE** is the root-mean-square error divided by the exact value, over seeds.
