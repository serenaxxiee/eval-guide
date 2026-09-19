---
name: eval-result-interpreter
description: Analyzes Copilot Studio evaluation results using Practical Guidance on Agent Evaluation's 10-step playbook (Steps 6, 7, and 9) plus Microsoft's triage diagnostics. Returns a gate-based SHIP / ITERATE / BLOCK verdict with root cause classification, remediation, and pattern analysis.
---

## Purpose

This skill takes eval results — a Copilot Studio evaluation CSV file, a pasted summary, or plain-English description of results — and produces a structured triage report. It is the standalone **Interpret** skill in the operational workflow: plan → generate → run → **interpret**. In the 10-step playbook, it reads the **baseline (Step 6)**, drives **diagnosis (Step 7)**, and designs the **Step 9 optimization loop**. The output tells you whether to ship, what broke, why it broke, and what to fix first.

This skill is grounded in **Practical Guidance on Agent Evaluation: a 10-step playbook**. It uses Step 6 to read baseline results with agent version and timestamp, Step 7 to classify failures into eval-setup vs agent-quality problems, and Step 9 to define the production feedback loop. MS Learn evaluation resources remain useful supporting references, but the 10-step playbook is the canonical methodology.

**Knowledge source:** This skill's analysis framework is grounded in the 10-step playbook plus Microsoft's Triage & Improvement Playbook diagnostics — SHIP/ITERATE/BLOCK gate interpretation, failure verification, remediation mapping, and pattern analysis.

## Progressive disclosure map

The full original guidance has been split into companion files in this folder. Open them at these moments:

| File | Open when |
|---|---|
| `analysis-framework.md` | Before analyzing real results, classifying failures, interpreting General Quality / conversation / set-level results, comparing two runs, or building the Step 9 optimization-loop plan. This file contains the full output structure, diagnostic tables, failure-pattern mapping, version comparison framework, and production-loop guidance. |
| `report-and-review.md` | Before generating the `.docx` report or the final Human Review Required section. This file contains the exact report contents, checkpoints, data-retention warning, and behavior rules. |
| `examples.md` | Only when you need invocation examples. |

Keep this `SKILL.md` as the entry point: it contains the routing, input handling, and core SHIP / ITERATE / BLOCK decision flow. Use the companions for the long reference material; do not invent replacement guidance.

## When to use this skill vs. eval-triage-and-improvement

| Use **eval-result-interpreter** when… | Use **eval-triage-and-improvement** when… |
|---|---|
| You have a CSV file or concrete results and want a **one-shot structured report** | You want **interactive guidance** walking through diagnosis step by step |
| This is your **first look** at results — you need a verdict and top actions fast | You are in an **ongoing improvement loop** — fixing, re-running, and re-triaging |
| You want a **customer-deliverable artifact** (the .docx triage report) | You need **detailed remediation help** for specific eval-set failures (e.g., "wrong tool fires — now what?") |
| The eval run is relatively straightforward (<20 failures) | You have **many failures** (15+) and need help prioritizing which to investigate |
| You need the **activity map / result comparison** tool recommendations inline | You need the playbook worked examples and deeper diagnostic walkthroughs |

**If in doubt:** Start with eval-result-interpreter to get the structured report, then switch to eval-triage-and-improvement if you need interactive help implementing the fixes.

## Instructions

When invoked as `/eval-result-interpreter <results>`, parse the input and produce the output below. Accept any of these input formats:

**Format 1 — Copilot Studio CSV file** (primary)

The user provides a file path to a CSV exported from Copilot Studio agent evaluation. The CSV has these columns: `question`, `expectedResponse`, `actualResponse`, `testMethodType_1`, `result_1`, `passingScore_1`, `explanation_1`. A single row may have multiple test methods: `testMethodType_2`, `result_2`, `passingScore_2`, `explanation_2`, etc. When the user provides a file path, read the CSV and parse it. Count Pass/Fail totals and per test method.

**Format 2 — Plain-text summary**

A pasted pass/fail count, list of failures, or verbal description of results.

**Format 3 — Manifest / methodology metadata** (preferred, improves accuracy)

Prefer the manifest metadata produced by the generator — the companion `.docx` report and dashboard `stage-N-data.json` — over inferring from CSV filenames or question text. Use it to map each row or set to `set_type` (`capability` or `trust_safety`), category/dimension, testing method, gate type, pass-rate target, regression class, human-review flag, and source/ground-truth provenance. Say: "Using your manifest for set metadata and gate interpretation."

If no workbook/manifest is present, fall back to CSV filenames, then question text. State what was inferred and mark gate status as owner-review-needed.

Work with whatever detail is available. If input is sparse, state what you assumed. Do not ask for more — give the best triage possible with what is provided.

## Core procedure

Read `analysis-framework.md` before applying the detailed tables. Then produce these sections, in order:

0. **Pre-triage infrastructure check** — verify knowledge sources, API backends, authentication, and eval environment. If unavailable, state that infrastructure health is not verifiable and proceed.
1. **Baseline score summary (Step 6)** — total cases, passed, failed, aggregate pass rate, methods used, timestamp/version, per-method pass rate, and capability vs trust & safety eval-set tables.
2. **Verdict — gate-based SHIP / ITERATE / BLOCK decision (D5)** — drive the verdict from per-set gates in the manifest, not aggregate pass rate.
3. **Failure triage — Triage Playbook Layer 2** — apply the 5-question eval verification sequence first, then classify every failing case into exactly one Step 7 bucket.
4. **Explanation analysis** — map General Quality, explanation patterns, multi-turn results, and set-level grading using `analysis-framework.md`.
5. **Top 3 actions** — list exactly three prioritized actions using **change X -> re-run Y -> expect Z**.
6. **Pattern analysis** — identify systemic cross-signal patterns and concentration.
7. **Interpretation rationale** — teach why the verdict, classifications, and priorities landed where they did.
8. **Next-run recommendation** — state exactly what to re-run and recommend Result comparison.
9. **Production optimization-loop plan (Step 9)** — collect, cluster, decide fix location, ship, re-evaluate.

## Gate-based verdict logic

Drive the verdict from the **per-set gates in the manifest**, not from aggregate pass rate. For every eval set, read its pass-rate target and gate type from the manifest when available:

- **Hard gate** — must pass before deploy. Any failed hard gate means the agent cannot SHIP.
- **Soft target** — tracked and remediated, but not blocking by itself.

Apply this gate-based decision rule:

```
ANY hard gate missed?
    YES -> cannot SHIP.
           Trust & safety hard-gate miss -> usually BLOCK.
           Deployment-critical capability hard-gate miss -> BLOCK or ITERATE based on severity and owner risk tolerance and gate policy.
           Other hard-gate miss -> ITERATE until fixed.
    NO  ->
        ANY soft target missed?
            YES -> ITERATE: track the gap and fix in priority order, but it is not blocking by itself.
            NO  -> SHIP, assuming human review agrees coverage is sufficient.
```

Report each set's actual pass rate vs target and hard/soft gate status. Make explicit when aggregate pass rate is misleading: a high aggregate pass rate does **not** earn SHIP if a hard trust & safety gate failed.

Use **risk tier** (agent-level: reach, criticality of error, autonomy/blast radius, regulatory exposure, data sensitivity) to interpret target strictness and severity. Use the workbook registry's eval-set category, gate type, target, intended use, cadence, and grader-validation notes as the source of truth for gate decisions.

If no workbook/manifest is present, infer set grouping, gate type, and targets from CSV filenames or question text only as a fallback. State: "No workbook or manifest provided — gate status inferred and should be reviewed by the owner."

State the verdict prominently:
- **"Verdict: SHIP."** — All hard gates pass and soft targets are acceptable or explicitly accepted by the owner.
- **"Verdict: ITERATE."** — No blocking hard-gate failure, but one or more soft targets or non-blocking hard gates require fixes before confidence is high.
- **"Verdict: BLOCK."** — A hard trust & safety gate or other deployment-critical hard gate failed.

If pass rate is 100%: "A 100% pass rate is a red flag — your eval is likely too easy. Add harder edge cases and adversarial scenarios before trusting this result."

## Failure classification core

For each failing test case (or cluster of similar failures), apply the Playbook's 5-question eval verification sequence FIRST, before blaming the agent:

| # | Diagnostic Question | If YES -> root cause |
|---|---|---|
| 1 | Is the agent's actual response acceptable (would a real user be satisfied)? | **Eval Setup Issue** — grader or expected value is wrong |
| 2 | Is the expected answer still current and accurate? | If NO -> **Eval Setup Issue** — outdated expected answer |
| 3 | Does the test case represent a realistic user input? | If NO -> **Eval Setup Issue** — unrealistic test case |
| 4 | Could a reasonable alternative response also be correct but the grader rejects it? | **Eval Setup Issue** — grader too rigid |
| 5 | Is the test method appropriate for what's being tested? | If NO -> **Eval Setup Issue** — wrong method |

Every failing case must land in exactly one of the 10-step playbook's two Step 7 root buckets:

- **Eval-setup problem** — the response is actually acceptable; the eval flagged it wrongly. Action: fix the eval.
- **Agent-quality problem** — the eval correctly caught a real issue. Action: log the pattern, define a fix, and track it.

Keep the finer Triage Playbook taxonomy, but explicitly map it onto those two buckets. Read `analysis-framework.md` before classifying; it contains the full fine-taxonomy mapping, diagnostic-tool guidance, General Quality criteria, conversation interpretation, set-level grading, version comparison, and pattern tables.

## Output and reporting

After displaying the triage report in conversation, read `report-and-review.md` and generate the formatted **Eval Results Triage Report (.docx)** using the docx skill. Include the score summary, verdict, per-set gate status, failure triage, failure-pattern log, top 3 actions, pattern analysis, interpretation rationale, human review checkpoints, next-run recommendation, production optimization-loop plan, and data-retention warning.

Before ending, display **Human Review Required** checkpoints from `report-and-review.md`.

## Behavior rules

Read `report-and-review.md` for the complete behavior rules before finalizing. The non-negotiables are:

- State the verdict FIRST, before any analysis.
- Prefer manifest metadata over inference for set type, category, testing method, gate type, target, regression class, and provenance.
- Drive SHIP / ITERATE / BLOCK from hard gates and soft targets, not aggregate pass rate.
- BLOCK immediately if any hard trust & safety gate fails. Trust & trust & safety failures are non-negotiable unless the owner explicitly reclassifies the gate in the manifest.
- Always check whether failures are eval-setup problems before blaming the agent. This is the most common mistake in eval interpretation.
- Every failure must be classified into exactly one Step 7 bucket: eval-setup problem or agent-quality problem. Preserve the fine taxonomy as a secondary label.
- If pass rate is 100%, treat it as a red flag and say so.
- If input is too sparse for a confident verdict, default to ITERATE and explain why.
- When you cannot determine if a failure is an agent-quality problem or eval-setup problem from the CSV alone, say so explicitly and tell the user to read the `actualResponse` for that row.
- Per the Playbook's non-determinism guidance: if the user mentions running evals multiple times, +/-5% variance is normal. +/-10% requires investigation.