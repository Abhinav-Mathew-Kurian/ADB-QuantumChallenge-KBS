# Prototype: sanitised description

This folder describes the Tandem prototype **without its source code**. It covers what the application does, its components and interfaces, and screenshots of the running system. The source is withheld from this public repository and is available to the challenge organisers and judges on request.

## 1. What the user sees

The application has one page, **Joint wipe-out**, laid out as seven chapters. It also has a **Records** area and a printable **report** for every run.

| # | Chapter | What it shows | Screens |
|---|---|---|---|
| 1 | **The ADB problem** | The question. The two bonds and their published terms. Every input is labelled *Published*, *Model assumption*, *Scenario assumption* or *Derived*. Shared earthquakes and shared rainfall link the bonds. | [01](screens/01-problem-question.png), [02](screens/02-problem-shared-events.png) |
| 2 | **Define the scenario** | One question, one fixed 3-year period, the brief's 5 scenarios, 3 supported precisions, seeds for runs A and B, a question summary, and a "Frozen demo build" badge that shows only while the solver fingerprint matches the freeze. | [03](screens/03-scenario-settings.png) |
| 3 | **Estimate** | Run stages with timings. The **circuit-integrity gate** (5 checks). The IQAE estimate with its 95% interval and "≈ 1 in N three-year terms". IQAE rounds and Grover powers. Measured resources, seed and run ID. Labelled "Simulator · Qiskit Aer (ideal) · not quantum hardware". | [04](screens/04-run-started.png), [05](screens/05-estimate-gate.png), [06](screens/06-estimate-result.png), [07](screens/07-estimate-iqae-rounds.png) |
| 4 | **Independent checks** | Exact reference, IQAE run B, Monte Carlo and cross-entropy importance sampling at **the same query budget**, as an interval chart and a table. Then the many-seed equal-budget study, either frozen or re-run live. | [08](screens/08-checks-equal-budget.png), [09](screens/09-checks-many-runs.png) |
| 5 | **DCE record** | Decision (*Ready to use / Rough guide / Not reliable yet*). Confidence, with its factors and the calibration caveat. Evidence: record ID, run ID, input hash, model fingerprint, circuit SHA, seeds, budget, software commit and packages. Actions: **Reproduce from record**, printable report, Markdown report, JSON export. | [10](screens/10-dce-record.png), [11](screens/11-dce-reproduced.png), [14](screens/14-report-printable.png) |
| 6 | **What drives the joint tail** | 1,000 worst-case scenarios drawn by amplitude amplification from the same frozen model. Their composition (sampled vs exact), the cost per scenario, and a statistical check against the exact distribution. Labelled "not a forecast". | [12](screens/12-drivers-worst-cases.png) |
| 7 | **Limitations and next stage** | Current limitations, and the four Phase 2 steps. | [13](screens/13-next-limitations-phase2.png) |

**Records** has four parts:
- workflow runs;
- the DCE ledger, with four-eyes review;
- calibration;
- a check of every headline figure against the frozen run.

## 2. Components

```
 ┌──────────────────────────────┐
 │ Web application              │  React + TypeScript; keyboard navigation; chapter captions;
 │                              │  printable report; accessible at phone width (WCAG 2 AA checks)
 └──────────────┬───────────────┘
                │ HTTPS / JSON, session auth
 ┌──────────────▼───────────────┐      ┌──────────────────────┐
 │ Evidence gateway (Node.js)   │─────►│ MongoDB evidence     │  immutable runs; content-addressed
 │ roles: analyst, reviewer,    │      │ store                │  evidence; DCE ledger; review log
 │ auditor (read-only)          │      └──────────────────────┘
 └──────────────┬───────────────┘
                │ JSON
 ┌──────────────▼───────────────┐
 │ Quantum service (Python,     │  exact model · 13-qubit loader · integrity gate · IQAE / MLAE ·
 │ FastAPI) on Qiskit + Aer     │  classical samplers · DCE scoring · worst-case sampling
 └──────────────────────────────┘
```

| Component | Responsibility |
|---|---|
| **Exact model** | The brief's two-country model (steps 1–8), exact term metrics, the 5 scenarios and 5 targets; the reference for every check |
| **Structured loader** | Builds the 13-qubit state preparation from closed-form per-country tables |
| **Circuit-integrity gate** | 5 checks against the exact model; blocks the run on any failure |
| **Estimators** | IQAE with rigorous intervals; MLAE and a noise-aware variant; certified invariant-plane executor for deep ideal runs |
| **Classical samplers** | Monte Carlo, Sobol, stratified, importance sampling, cross-entropy IS, structured IS |
| **DCE scoring** | Decision, Confidence (precision, reliability, integrity, provenance, corroboration) and the Evidence Object |
| **Worst-case sampler** | Amplitude-amplified sampling of the joint tail, checked against exact conditional fractions |
| **Evidence gateway** | Authentication and roles. Stores every run immutably, keeps the DCE ledger, runs four-eyes review, reproduces runs from records, exports reports, runs background jobs. Waits (with a bound) when the simulator is busy instead of failing the request. |

## 3. Interfaces (shape only)

**Quantum service:**

| Method | Route | Purpose |
|---|---|---|
| GET | `/v1/workflow/programme` | Bond terms, labelled inputs, scenarios, precisions |
| POST | `/v1/workflow/run` | Integrity gate → IQAE → independent checks → DCE record |
| POST | `/v1/workflow/worst-cases` | Amplitude-amplified worst-case sampling on the frozen model |
| POST | `/v1/workflow/budget-study` | Many-seed equal-budget study |

**Evidence gateway:**

| Method | Route | Purpose |
|---|---|---|
| GET | `/api/workflow/programme` | As above, through the gateway |
| POST | `/api/workflow/runs` | Start a run; store the DCE record and an immutable run; queue the worst-case job |
| GET | `/api/workflow/runs`, `/api/workflow/runs/:id` | List and poll runs |
| GET | `/api/workflow/runs/:id/report` | Markdown report generated only from stored evidence |
| POST | `/api/workflow/runs/:id/reproduce` | Re-run from the stored request and compare 8 stored outputs |
| POST | `/api/workflow/budget-study` | Equal-budget study |

**Run request** (example):

```json
{ "scenario": "adb-base", "precision": "standard", "seed": 0, "repeatSeed": 1 }
```

**Run response** (abridged and simplified; the full frozen output is in [`../evidence/sample-results/joint-wipeout-frozen-run.json`](../evidence/sample-results/joint-wipeout-frozen-run.json)):

```json
{
  "integrity": { "passed": true, "checks": 5 },
  "estimate": { "method": "IQAE", "value": 3.1586e-5, "interval": [2.9501e-5, 3.3376e-5], "level": 0.95, "queries": 77370 },
  "comparison": [ "exact", "iqae (repeat seed)", "monte_carlo", "importance_ce" ],
  "dce": { "decision": "ACCEPT", "confidence": 0.80, "evidenceId": "8008ac7f5b8c6173a536" },
  "scope": "Model estimate on an ideal simulator; not a price, rating, forecast or investment advice."
}
```

Every response carries a scope statement. If the integrity gate fails, the response is `blocked`, with the diagnostics and no estimate.

## 4. Running it

The prototype runs as three local services: MongoDB, the quantum service and the gateway, plus the web application. A single stack script starts them all. Before a recording, a pre-recording check runs the frozen request three times through the full stack and checks 18 criteria (see [`../evidence/sample-results/pre-recording-checks.csv`](../evidence/sample-results/pre-recording-checks.csv)). The Python dependencies are pinned in [`../requirements.txt`](../requirements.txt).
