# AI tools disclosure

The challenge brief asks entrants to disclose any AI use in the submission package. This page lists every AI tool used, what it was used for, and how its output was controlled.

## Tools used

| Tool | Provider | Used for |
|---|---|---|
| **Claude Code** (Claude Opus model family) | Anthropic | **Coding:** Python, TypeScript and Node.js code, tests, evidence scripts, and the scripted recording of the demo film, all under the team's direction. **Research:** finding and reading the papers the brief cites and the further literature in our references. **Writing:** drafting status notes and documentation, including this repository's documents. |
| **ElevenLabs** text-to-speech | ElevenLabs | The narration voice in `evidence/Model run video.mp4`. The narration script was written by the team (with drafting help from Claude Code) and timed to the film. The voice is synthetic, not a recording of a team member. |

## What AI is *not*

- **No AI model is part of the delivered method.** The estimator is quantum amplitude estimation (IQAE, MLAE). The baselines are standard classical samplers and exact solvers. DCE scoring is deterministic arithmetic. None of them calls a language model at run time.
- **The film shows the real application.** It was captured frame by frame from the live application running the frozen request. The screen content is not AI-generated.

## How AI output was controlled

1. **Numbers are computed, never typed.**
   - Every figure in the documentation comes from an evidence file written by a script.
   - Each evidence file records the software versions and seeds.
   - The sample results in this repository were exported from those files by script.
2. **Tests guard the science.** The automated tests (443 Python tests at the freeze) cover:
   - exact reproduction of the brief's 25 reference values;
   - agreement between independent exact methods;
   - circuit amplitudes equal to exact model values;
   - interval coverage;
   - the circuit-integrity gate, including tamper tests.
3. **Mistakes were caught by measurement and recorded.** Examples:
   - a Markov-chain self-loop bug;
   - a mis-specified noise model;
   - an importance-sampling weight collapse;
   - a test asserting folklore that turned out false.

   Each was fixed at its cause, with a regression test. See [RESULTS_AND_VALIDATION.md](RESULTS_AND_VALIDATION.md) §4.
4. **Citations were checked against the source** (publisher page or arXiv) before use. Parameters without a published calibration are labelled **illustrative** wherever they appear.
5. **Claims policy.** We make no claim that our own measurements do not support. In particular:
   - no quantum-advantage claim;
   - no hardware-execution claim;
   - no forecast.

## Human responsibility

The team set the scientific direction, chose the methods, decided which claims to make and which not to make, and reviewed every result. Key team decisions:
- comparing against the strongest classical baselines, not only the brief's four;
- reporting the fault-tolerant estimate honestly;
- gating every estimate on circuit integrity;
- wrapping every number in a DCE record.

The team is accountable for the submission.
