<!-- Companion reference for the `eval-guide` skill. Loaded on demand from SKILL.md. -->

## Stage 1: Plan

Using the Agent Vision, produce a structured eval suite plan. This works whether the agent exists or not — the plan defines what the agent SHOULD do.

### What you walk away with

- **A populated copy of the Eval Suite Planning & Logging Template** for stakeholder review and ongoing use.
- **An interactive HTML review page** that summarizes the workbook, filters eval sets, surfaces TBDs, and keeps the chat response short.
- **One row per eval set** in the registry: capability sets, trust & safety sets, targets/gates, intended use, cadence, human inputs, source dependencies, grader-validation notes, and reusable-asset flags.
- **Baseline placeholders and reusable-asset candidates** in the existing template tabs, without adding or changing workbook structure.

### When this stage is wrong for you

- You do not have the Eval Suite Template and need a final customer artifact now. Ask for the blank template first.
- You already have a populated registry and only need test cases. Skip to Generate.
- You only need to triage run results. Skip to Interpret.

### What to do

1. **Copy and preserve the template.**

   Use the attached Eval Suite Template workbook when available. Populate a copy of it only. Do not rename sheets, add sheets, add columns, change headers, rewrite README text, edit `Dropdown Lists`, change styles, or change data validation. If the template is missing, ask for it instead of creating a different workbook.

2. **Populate Step 1 planning.**

   Fill the existing `1 . Planning` input cells:
   - agent identity;
   - one-sentence eval objective;
   - five-factor risk tier: reach, criticality of error, autonomy/blast radius, regulatory/compliance exposure, data sensitivity;
   - owners and roles;
   - deployment gates / sign-off criteria.

3. **Define eval sets, not scenarios.**

   Populate `2 . Eval Suite Registry` with one row per eval set, using the three categories in `skills/eval-guide/targeted-eval-sets.md`:
   - **Capability** sets: accuracy/correctness, faithfulness/groundedness, relevancy, style/tone, reasoning/tool use as applicable. Hallucination stays in faithfulness/groundedness.
   - **Trust & Safety** sets: guardrails, out-of-scope handling, sensitive-data handling, prompt injection/jailbreak resilience, compliance-specific behavior as applicable.
   - **Agent-specific instruction-following** sets: one row per testable instruction, when the Vision carries `agent_instructions`. The template's `Category` dropdown has no third value — map these onto `Capability` (or `Trust & Safety` for routing/refusal obligations), pick the closest `Dimension tested`, quote the instruction verbatim in `Purpose / diagnostic signal`, and record `Set category: Agent-specific instruction-following` in `Notes`. Never edit the dropdowns.

   Do not generate legacy planning-artifact rows in the workbook. The registry is one row per eval set only.

4. **Apply v5 Step 4 gates and improvement targets.**

   Use the existing registry columns:
   - Trust & Safety sets: absolute pass-rate hard gates, usually near 100%.
   - Capability sets: launch floor for first deployment plus regression/direction after baseline, not a standing absolute target.
   - High-risk capabilities: explicit hard floor when the capability functions like a guardrail.
   - Put the nuance in `Target pass rate`, `Target rationale`, `Gate type`, `Intended use`, `Run cadence`, and `Notes`; do not add a new column.

5. **Specify Step 5 human inputs.**

   Use the registry columns for human input type/author, grounding source dependency, and source-change review. Use `TBD - confirm before baseline` where owners or sources are unknown.

6. **Plan Step 6 grader validation.**

   The template has no grader-validation columns. Do not add them. Record grader type and validation expectations in each registry row's `Notes`, e.g. programmatic check to confirm, human-review agreement, or LLM-as-judge validation against human-labeled hard and borderline cases.

7. **Seed Step 7 baseline placeholders only in existing Run Log columns.**

   Add optional baseline placeholder rows in `3 . Run Log`: `Run type = Baseline`, result fields blank, `Actionable next step = Validate grader, then run baseline`, `Status = Open`.

8. **Partition Step 8 regression cadence.**

   Use `Intended use` and `Run cadence` in the registry. Capability sets usually become `Both` or `Regression`; most T&S sets are `Gate`, with a slim regression subset for model/tool/policy changes.

9. **Flag Step 10 reusable assets.**

   Populate `4 . Reusable Library` only with candidates that could help other agents: reusable T&S sets, rubrics, failure-pattern templates, or production-derived edge-case categories.

### Output

Do not display a long eval-set summary in chat. Put Step 1 objective/risk/owner, capability eval sets, trust & safety eval sets, Step 4 governance, Step 5 human inputs, Step 6 grader-validation notes, Step 8 cadence, and Step 10 reusable candidates into the interactive HTML review page described below.

**The customer payoff:** *"You now have a workbook your PM, builder, risk owner, and source owners can review. It preserves your template and shows which eval sets exist, how each is governed, who owns human inputs, what must happen before baseline, and which assets may be reusable."*

**Maturity callout — Pillar 1 / playbook Step 1 (L100 Initial → L300 Systematic):** Discover + Plan advance Pillar 1 from "good lives in the builder's head" to a written objective, five-factor risk tier, accountable owner, and workbook-backed eval-set governance. Pillar 2 advances in Generate (Steps 2, 3, 5); Pillar 3 now starts with Step 6 grader validation before any baseline is trusted.

### Workbook Review Checkpoint

The legacy Plan dashboard is pre-v5 criteria-based and must not be used for the v5 workbook workflow. Instead, generate a draft workbook copy plus a companion HTML review page and have the customer review those artifacts.

1. Generate `eval-suite-<agent-name>-<YYYY-MM-DD>.xlsx` as a populated copy of the user's template.
2. Generate `eval-suite-<agent-name>-<YYYY-MM-DD>-review.html` next to it using `skills/eval-guide/plan-review-page.md`.
3. Ask the customer to review the workbook and the HTML page:
   - `1 . Planning`: objective, risk tier, owners, sign-off criteria.
   - `2 . Eval Suite Registry`: eval-set rows, Step 4 governance, cadence, human inputs, source dependencies, grader-validation notes, reusable flags.
   - `3 . Run Log`: baseline placeholders, if added.
   - `4 . Reusable Library`: reusable candidates.
4. Apply workbook feedback by editing cell values in a new copy of the workbook and regenerating the HTML review page. Do not edit structure, sheet names, headers, styles, README, or `Dropdown Lists`.
5. When the customer confirms the workbook, treat it as the Plan artifact and proceed to Generate from the registry.

**After confirmation, the eval plan deliverable is:**

   **Customer-ready `.xlsx` eval-suite planning workbook** using the `/xlsx` skill, named `eval-suite-<agent-name>-<YYYY-MM-DD>.xlsx`. It must be a populated copy of the user's template, not a recreated or redesigned spreadsheet.

   **Interactive HTML review page** named `eval-suite-<agent-name>-<YYYY-MM-DD>-review.html`. The page carries the summary, eval-set explorer, TBD action list, and human review checklist so the chat response stays concise.

   The workbook is the review checkpoint and the primary Plan artifact. A separate `.docx` narrative is optional only if the user asks for it.

   Tell the customer only where to open the workbook and HTML review page. Do not duplicate the HTML page content in chat.

---

