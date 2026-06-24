# Evals for `/eval-guide`

This folder contains the **eval set for the `/eval-guide` skill itself** — built by applying `/eval-guide`'s own Stage 1 + Stage 2 methodology to `/eval-guide`. Eat the dogfood.

## Why this exists

`/eval-guide` is an eval-authoring skill that customers depend on for production-eval design. If the skill drifts (e.g., produces 12 fragmented quality dimensions instead of 4–6 consolidated ones; quietly accepts aspirational-language capabilities; forgets to apply the 20% rule in triage; resists adversarial inputs poorly), customers get bad eval plans and won't know it. These evals codify the invariants the skill commits to so we can detect regressions.

## Structure

```
evals/
├── README.md              ← this file
├── eval-plan.md           ← the Plan output applied to /eval-guide itself:
│                            Agent Vision, workbook registry expectations,
│                            methods, pass/fail conditions
└── test-cases.json        ← the Stage 2 output: concrete test prompts + expected
                             behaviors per criterion, structured for an LLM-judge
                             or human-grader workflow
```

## How to run

There are two execution modes — pick by your bandwidth.

### Mode 1 — Human-grader (gold standard)

For each test case in `test-cases.json`:

1. Open a **fresh** Claude Code session (no prior `/eval-guide` context).
2. Paste the `trigger_input` into chat.
3. Observe the AI's full response through whatever stage the test exercises.
4. Score against `pass_conditions` and `fail_conditions` — both must hold to pass.
5. Record verdict (pass / fail / unsure) + notes per case.

This is gold-standard but slow. Use it for hard-gated Trust & Safety and deployment-critical capability cases at minimum.

### Mode 2 — LLM-judge (scalable)

For each test case, run a 3-step pipeline:

1. **Capture transcript** — invoke `/eval-guide` with the `trigger_input` (manually or via [`copilot-studio:run-eval`](../skills/eval-guide/) if available). Save the AI's full response.
2. **Run judge** — pass the transcript + `pass_conditions` + `fail_conditions` to an LLM judge (Claude Sonnet works). Use a Custom rubric with explicit Pass/Fail/Unclear labels.
3. **Calibrate** — for the first run, **also human-grade ~10–15% of cases** and compare to judge verdicts. If agreement < 80% (Cohen's κ < 0.6), the rubric needs sharpening before trusting the judge.

LLM-judge is fast but non-deterministic (±5% variance per run). For borderline cases, run twice and take the median.

## Coverage

The plan covers `/eval-guide`'s six stages plus cross-cutting invariants:

| Stage / Group | Test count | Primary eval-set focus |
|---|---:|---|
| Triggers | 4 | Activation and routing boundaries |
| Discover | 4 | Agent Vision, objective, risk tier |
| Plan | 7 | Template-preserving workbook registry and governance |
| Generate | 5 | CSVs by eval set and method metadata |
| Run | 2 | Execution/export discipline |
| Interpret | 4 | Gate-based verdict and Step 7 triage |
| Cross-cutting | 4 | Coaching, feedback application, and invariants |
| **Total** | **30** | Capability + Trust & Safety coverage |

Coverage intentionally includes both capability checks and Trust & Safety / governance checks, because most drift shows up as missing boundaries, stale grader assumptions, template changes, or release-readiness mistakes.

## When to re-run

- **Pre-merge** — every PR that touches `skills/eval-guide/SKILL.md`, `dashboard/templates/*.html`, `dashboard/serve.py`, or any reference doc. Run all hard-gated Trust & Safety and deployment-critical capability cases.
- **Post-deploy / pre-release** — full eval set on the published plugin version.
- **After model upgrade** (Sonnet 4.5 → 4.6, etc.) — full eval set, expect a 5–10% pass-rate swing while re-grounding.
- **On customer report** — if a customer says "the skill produced X weird behavior," add a regression test to the relevant eval set before fixing.

## Calibration baseline

A green run on this eval set means:
- All hard-gated Trust & Safety and deployment-critical capability cases pass.
- Capability coverage meets the target in the workbook/manifest.
- No regression cases fail after a change.
- Eval setup failures have been separated from true agent-quality failures.

Below those, **don't ship**. Diagnose with `/eval-result-interpreter` (or just read the failure column in the test results CSV) and remediate.

## What this eval set deliberately does NOT test

- **Dashboard UI rendering** — visual layout, color contrast, button placement. A real browser-test framework (Playwright, etc.) would cover this; we don't have one wired up. The eval-set tests the *behavior* the dashboard exposes, not its appearance.
- **Plugin install / version-check / upgrade flow** — that's a separate concern; the dashboard `bin/eval-guide-update-check` script has its own integration tests path.
- **End-to-end against a live Copilot Studio agent** — Stage 3 execution evals require a running agent and DirectLine endpoint, which is per-customer and not codifiable here. We test the *guidance* the skill produces around Stage 3, not the run itself.

## Updating these evals

The eval-plan and test cases are themselves the output of running `/eval-guide` with `/eval-guide` as the agent under test. When the skill's behavior intentionally changes:

1. Update the relevant criterion in `eval-plan.md`
2. Update the matching test cases in `test-cases.json`
3. Re-run the affected quadrant
4. Note the change in the commit message (which criterion changed and why)

Don't update tests to make them pass — investigate the behavior change first.
