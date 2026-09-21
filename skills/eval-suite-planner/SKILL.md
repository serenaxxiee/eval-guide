---
name: eval-suite-planner
description: 'Plan standalone — populates the Eval Suite Planning & Logging Template from an Agent Vision or plain-English agent description. Asks once for the agent''s instructions so instruction-following eval sets can be planned. Grounded in Practical Guidance on Agent Evaluation v5: Step 1 planning, Steps 2-3 eval-set decomposition, Step 4 gates/improvement targets, Step 5 human inputs, Step 6 grader-validation planning, Step 7 baseline placeholders, Step 8 regression partitioning, and Step 10 reusable-asset candidates. Output is a template-preserving `.xlsx` workbook plus an interactive HTML review page. Use before generating test cases or running evals.'
---

## Purpose

This skill produces the **Plan** artifact of the `/eval-guide` lifecycle: a populated copy of the customer's **Eval Suite Planning & Logging Template** plus an interactive HTML review page. The workbook is the source-of-truth artifact; do not replace it with a scenario table, quality-signal table, generic spreadsheet, default `.docx` report, or HTML-only plan.

The skill aligns to `skills/eval-guide/playbook.md` and `skills/eval-guide/eval-suite-template.md`. Use the 10-step playbook as the methodology spine and the XLSX template as the output shape.

## Core rule

**Copy the blank XLSX template and populate existing cells/rows only. Do not modify the template.**

Do not rename sheets, add sheets, delete sheets, add columns, change headers, rewrite README text, edit `Dropdown Lists`, change styles, change data validation, or convert the template into a different spreadsheet.

If a blank template workbook is available in the session, use it. If not, ask the user to provide the template; do not silently invent a new workbook.

## Question policy

Ask targeted questions only when a workbook field materially affects the plan and cannot be inferred safely:

1. Eval owner / named approver.
2. Lifecycle stage and target deployment decision.
3. Whether the agent is prompt-only, RAG/knowledge-grounded, or agentic with tools/connectors.
4. Regulated/compliance obligations.
5. Authoritative sources and source owners.

If the user wants speed or cannot answer, populate `TBD - confirm before baseline`.

### Ask once for the agent's instructions

Separately from the list above, ask **one** question about the agent's instructions before populating the registry — the agent-specific instruction-following sets cannot be planned without them.

**Skip the question entirely** if the conversation, attachments, Agent Vision, or a supplied description already contain the instruction block. Use what's there and say so.

**Otherwise ask, and wait:**

> *"Want to paste your agent's instructions (the system prompt / instruction block from Copilot Studio)? I'll plan a set per testable instruction, so a failure points at the exact instruction the agent ignored. Without them the plan still covers common capability and trust & safety sets."*
>
> Options: **Paste the instructions** · **Point me at a file or attachment** · **Skip — plan common sets only**

**If supplied:** list the testable instructions you extracted and the ones you dropped as untestable, with one-line reasons, before populating the registry.

**If skipped:** plan Categories 1 and 2, and record the gap in the HTML review page's action list — *"No instruction-following eval sets planned; the agent's instruction block wasn't provided."* Never block on the answer, and never invent instructions the customer didn't write.

## Planning method

When invoked as `/eval-suite-planner <agent description>`:

1. Extract or infer the agent's purpose, users, knowledge sources, capabilities, boundaries, architecture, lifecycle stage, and known risks.
2. Populate **Step 1 — Plan the Eval Effort**:
   - one-sentence eval objective;
   - five-factor risk tier: reach, criticality of error, autonomy/blast radius, regulatory/compliance exposure, data sensitivity;
   - one accountable owner.
3. Define eval sets, not scenarios. Use the three categories in `targeted-eval-sets.md`:
   - **Capability eval sets**: one row per capability dimension that must be diagnostic, e.g. accuracy/correctness, faithfulness/groundedness, relevancy, style/tone, reasoning/tool use. Gate the dimensions to the agent's architecture — no tool-use row for a prompt-only agent.
   - **Trust & Safety eval sets**: one row per refusal, boundary, or safety category, e.g. guardrails, out-of-scope handling, sensitive-data handling, prompt injection/jailbreak, compliance-specific behavior.
   - **Agent-specific instruction-following eval sets**: one row per testable instruction from the agent's instruction block, when it was supplied. Apply the catalog's testability filter — observable, has a trigger, has a discriminating negative — and see *Instruction-following rows* below for how to fit them into the template's controlled vocabulary.
4. Apply **Step 4 v5 gates/improvement-target logic**:
   - T&S sets use absolute pass-rate hard gates, usually near 100%.
   - Capability sets usually use a launch floor for first deployment plus regression/direction after baseline, not a standing absolute pass-rate target.
   - High-risk capabilities that function like guardrails keep explicit hard floors.
   - Use the template's existing `Target pass rate`, `Target rationale`, `Gate type`, `Intended use`, `Run cadence`, and `Notes` columns to express this; do not add a new column.
5. Specify **Step 5 human inputs**:
   - grading rubric, ground truth, golden answer, or rubric + ground truth;
   - author/owner;
   - grounding source dependency;
   - whether source changes require review.
6. Plan **Step 6 grader validation** without changing the template:
   - record grader type and validation expectation in the registry row's `Notes`;
   - for LLM-as-judge / Custom rubrics, note that human-labeled hard and borderline cases must validate the judge before baseline scores are trusted;
   - for programmatic checks, note the deterministic check to confirm;
   - for human grading, note reviewer agreement expectations where relevant.
7. Seed **Step 7 baseline placeholders** in `3 . Run Log` only when useful:
   - one placeholder row per eval set;
   - `Run type = Baseline`;
   - result fields blank;
   - `Actionable next step = Validate grader, then run baseline`;
   - `Status = Open`.
8. Apply **Step 8 regression partitioning** in existing registry fields:
   - capability sets usually `Intended use = Both` or `Regression`;
   - most T&S sets are `Gate`; the slim subset likely affected by model/tool/policy changes can be `Both` or `Regression`;
   - set `Run cadence` using existing dropdown values such as `Per-change`, `Nightly`, `Weekly`, or `Milestone-only`.
9. Flag **Step 10 reusable assets** in `4 . Reusable Library`:
   - reusable T&S sets;
   - grading rubrics;
   - failure-pattern templates;
   - production-derived edge-case categories when applicable.

## Workbook population rules

Use `skills/eval-guide/eval-suite-template.md` as the exact tab/column map.

### `README`

Do not edit.

### `1 . Planning`

Populate only existing input cells:

- Agent identity.
- Risk classification (5 factors).
- Owners & roles.
- Deployment gates / sign-off criteria.

For the template's `Min pass rate - Capability` row, reflect v5 Step 4 accurately: use launch floor / high-risk capability floor / regression-governance language, not a generic scenario pass-rate target.

### `2 . Eval Suite Registry`

Populate one row per eval set. Do **not** populate one row per test case or legacy planning artifact.

Required row semantics:

- `Category`: `Capability` or `Trust & Safety`.
- `Dimension tested`: capability dimension or T&S category from the template dropdowns.
- `Purpose / diagnostic signal`: what failure in this set diagnoses.
- `Target pass rate`: absolute gate for T&S; launch floor or `Regression / direction after baseline` for most capability sets.
- `Target rationale`: v5 Step 4 rationale.
- `Gate type`: closest existing dropdown value.
- `Intended use`: `Gate`, `Regression`, or `Both`.
- `Run cadence`: cadence for Step 8.
- `Human input type`, `Human input author`, `Grounding source dependency`, `Source change -> review?`: Step 5.
- `Reusable asset?`, `Reuse tier`, `Set status`: Step 10 and lifecycle status.
- `Notes`: assumptions, open questions, Step 4 nuance, and Step 6 grader-validation plan.

#### Instruction-following rows

The `Dropdown Lists` tab has no `Instruction-following` value and must not be edited. Map instruction-following sets onto the existing vocabulary:

- `Category`: `Capability`, unless the instruction is a refusal or routing obligation (*"hand off anything payroll-related"*) — then `Trust & Safety`.
- `Dimension tested`: the closest existing dropdown value (citation instructions → `Faithfulness / groundedness`; format instructions → `Style & tone`; clarifying-question instructions → `Relevancy`; handoff instructions → `Out-of-scope handling`).
- `Eval Set Name`: lead with the instruction, e.g. `Instruction: cite the policy section used`.
- `Purpose / diagnostic signal`: quote the instruction verbatim and name what a failure diagnoses.
- `Intended use`: `Both` or `Regression` — instruction-following is the first thing a prompt or model change breaks.
- `Notes`: record `Set category: Agent-specific instruction-following` plus the source instruction, so `/eval-generator` can recover the category from a registry that has no column for it.

Do not add a column, a sheet, or a dropdown value to represent this category.

### `3 . Run Log`

Use this for Step 7 baseline/iteration logging. During planning, add placeholder baseline rows only if useful; keep result fields blank.

### `4 . Reusable Library`

Populate candidate reusable assets only. Do not duplicate every eval set; promote assets that could help other agents.

### `Dropdown Lists`

Do not edit.

## Output

Create `eval-suite-<agent-name>-<YYYY-MM-DD>.xlsx` as a populated copy of the template.

Then create `eval-suite-<agent-name>-<YYYY-MM-DD>-review.html` next to the workbook using `skills/eval-guide/plan-review-page.md`.

Do not paste the summary, eval-set table, or checklist into chat. The HTML page carries that content. The final chat response should be only the workbook path, the HTML review page path, and any blocker/manual action.

## Human review checkpoints

Include these in the HTML review page checklist instead of displaying them in chat:

| # | Checkpoint | What to verify |
|---|---|---|
| 1 | Objective, risk tier, owner | The objective is decision-oriented, the five-factor risk tier is right, and a named owner can sign off. |
| 2 | Eval-set decomposition | Capability sets isolate one diagnostic capability each; T&S sets remain separate from capability; instruction-following sets quote a real instruction and aren't duplicated under another category. |
| 3 | Step 4 bars | T&S has absolute hard gates; capability uses launch floors / regression-direction unless high-risk. |
| 4 | Human inputs | Rubrics, ground truths, golden answers, and source dependencies have owners. |
| 5 | Grader validation | Each set has a plausible grader type and validation plan before baseline. |
| 6 | Regression partition | Capability and slim T&S regression sets have cadence; gate-only T&S sets run at milestones. |
| 7 | Template integrity | No sheets, columns, headers, dropdowns, README text, or formatting were changed — including for instruction-following rows. |
| 8 | Instruction coverage | Every enforceable instruction has a registry row and the source instruction is quoted verbatim in `Purpose / diagnostic signal`. If instructions weren't supplied, the gap is listed as an action. |

## Behavior rules

- Do not generate scenario-plan tables as the Plan artifact.
- Do not generate quality-signal sheets or quality-signal grouping as the Plan artifact.
- Do not add columns to support missing concepts; use existing fields, especially `Notes`.
- Do not invent agent instructions. Plan instruction-following sets only from instructions the customer actually supplied.
- Do not create a `.docx` unless the user explicitly asks for a narrative report.
- Do not produce long narrative chat output after artifact generation; use the HTML review page for the interactive summary and checkpoints.
- Be specific to the described agent, but at eval-set granularity.

## Companion skills

- **`/eval-generator`** — Generate test cases from the populated workbook registry.
- **`/eval-result-interpreter`** — Interpret baseline / iteration results using Step 6-7 and gate status.
- **`/eval-triage-and-improvement`** — Diagnose failures and feed the Step 9 optimization loop.
- **`/eval-library-promoter`** — Promote Step 10 reusable assets.
- **`/eval-guide`** — Orchestrated workflow with dashboard review checkpoints.
