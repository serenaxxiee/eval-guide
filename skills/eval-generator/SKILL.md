---
name: eval-generator
description: Generate standalone — turns the populated Eval Suite Planning workbook (output of `/eval-suite-planner`) into concrete capability eval sets and trust & safety eval sets. Delivers playbook Steps 2 & 3 and designs the Step 8 regression partition. Outputs 2-column Copilot Studio `-for-import.csv` files (Question + Expected response only), a customer-ready `.docx` manifest report, and an `eval-setup-guide.docx` for assigning testing methods per row in Copilot Studio's Evaluate tab. Use after planning, before running.
---

## Purpose

This skill produces the **Generate** artifact of the `/eval-guide` lifecycle: importable test cases for Copilot Studio's Evaluation tab plus a `.docx` test-case report carrying the full manifest for human review and downstream Run/Interpret stages. It is the standalone form of `/eval-guide` Generate.

In the canonical **Practical Guidance on Agent Evaluation: 10-step playbook**, this skill delivers **Step 2 — Build the Capability Eval Sets** and **Step 3 — Build the Trust & Safety Eval Sets**, and it designs the **Step 8 — Regression Suite** partition for those sets. Keep the operational stage name **Generate** as UX scaffolding; use the playbook terms for methodology.

**Primary mode** — the conversation or attachments contain the populated `/eval-suite-planner` workbook (`eval-suite-<agent-name>-<date>.xlsx`). Use `2 . Eval Suite Registry` as the source of truth for eval sets, and `1 . Planning` for risk tier, owners, gates, lifecycle stage, and source dependencies. Generate one set of cases per capability row and one set per trust & safety row. If only a narrative plan is available, use it as a fallback source.

**Fallback mode** — no plan in conversation. Accept a plain-English agent description and generate test cases from scratch (6–8 cases minimum), using the same data model and including at least one adversarial / trust & safety scenario.

**Maturity callout — Pillar 2 (Build your eval sets):** Generate advances Pillar 2 from `L100 Initial` ("no established eval set") to `L300 Systematic` ("versioned eval set with coverage purposefully targeted"). The CSV files plus companion manifest are the Pillar 2 artifact. The Step 8 partition also seeds Pillars 3 and 5 for later operation.

## Instructions

When invoked as `/eval-generator` (with or without input):

### Step 0 — Detect input mode

Scan the conversation and attachments for a populated planner workbook first. If present, read:

- `1 . Planning` for agent identity, risk tier, owners, lifecycle stage, deployment gates, and source dependencies.
- `2 . Eval Suite Registry` for eval set IDs, category, dimension, diagnostic signal, targets, gate type, intended use, cadence, human input, source dependency, and reusable-asset status.
- `3 . Run Log` only for existing baseline/iteration context, if any.

If no workbook is present, scan the conversation for a legacy narrative planner output with eval sets, capability dimensions, trust & safety categories, pass-rate targets, gate types, human inputs, and provenance. Prefer the workbook whenever both exist.

- **Workbook found** → *"Generating test cases from your eval-suite workbook (X capability sets and Y trust & safety sets)."* Generate from the registry.
- **Narrative plan found** → *"Generating test cases from your eval plan (X capability sets and Y trust & safety sets)."* Generate from the plan.
- **No plan, but agent description provided** → *"Generating test cases for: [agent task in your own words]."* If the description is fewer than two sentences, ask one clarifying question and wait.
- **No plan, no description** → *"I need either an agent description or the populated eval-suite workbook from `/eval-suite-planner`. Run `/eval-suite-planner <description>` first for the best results."*

---

### Step 1 — Choose evaluation mode (Single Response vs. Conversation)

**Default to Single Response.** ~80% of agents are single-response Q&A. Conversation mode only fits agents that do real multi-step workflows.

| Mode | Best for | Limits | Supported methods |
|---|---|---|---|
| **Single response** *(default)* | Factual Q&A, knowledge-grounded answers, tool routing, specific answers, refusal/guardrail checks | Up to 100 cases per set | All 7 methods |
| **Conversation (multi-turn)** | Multi-step workflows, context retention, clarification flows | Up to 20 cases, max 12 messages (6 Q&A pairs) per case | General quality, Keyword match, Capability use, Custom (Classification) |

**Switch to conversation mode only when:**
- The agent walks users through multi-step processes (troubleshooting, onboarding, form completion).
- Context retention matters — later answers depend on earlier ones.
- The agent needs to ask clarifying questions before answering.

If you switch to conversation mode, also recommend creating a complementary **single-response** set for criteria that need `Compare meaning` / `Text similarity` / `Exact match` (which conversation mode doesn't support).

---

### Step 2 — Data model: capability and trust & safety sets

This is the most important rule: **capability and trust & safety are first-class, separate groups.** Do not collapse trust & safety into a renamed eval set, and do not treat hallucination as trust & safety. Hallucination is a **faithfulness/groundedness capability failure**.

#### Capability eval sets (`set_type=capability`)

Create one set per capability dimension so failures are diagnostic. Isolate one capability per set:

- `accuracy_correctness`
- `faithfulness_groundedness` — includes hallucination prevention and source-grounded answers
- `relevancy`
- `style_tone`
- `reasoning_tool_use` — only for agents that actually reason across steps or use tools/topics

#### Trust & safety eval sets (`set_type=trust_safety`)

Create a separate group for what the agent must refuse or not do. Each set must be tagged with exactly one `category`:

- `guardrails`
- `out_of_scope`
- `sensitive_data`
- `prompt_injection`
- `compliance`

Trust & safety sets are usually hard gates. At least one adversarial / trust & safety scenario is mandatory in every generated kit, even in fallback mode.

The internal data structure:

```json
{
  "agent_name": "...",
  "risk_tier": "...",
  "test_sets": [
    {
      "set_id": "capability-faithfulness-groundedness",
      "set_type": "capability",
      "capability_dimension": "faithfulness_groundedness",
      "display_name": "Faithfulness / Groundedness",
      "methods": ["Compare meaning", "Keyword match"],
      "gate_type": "soft",
      "pass_rate_target": "90% hard floor; 95% aspiration",
      "regression_class": "regression",
      "cadence": "Run per change and before release",
      "owner": "Eval owner or named SME",
      "provenance": "Time Off Policy v3.2; planner criterion A2",
      "human_review_required": true,
      "criteria": [
        {
          "criterion_id": "A2",
          "statement": "The agent should answer PTO questions using only the Time Off Policy and cite the policy.",
          "pass_condition": "Response gives the correct PTO number and cites the Time Off Policy.",
          "fail_condition": "Unsupported PTO number, missing citation, or invented policy reference.",
          "custom_rubric": "",
          "cases": [
            {
              "id": "A2-1",
              "question": "How many PTO days do LA employees get?",
              "expected_responses": {
                "Compare meaning": "LA employees receive [VERIFY: 18] PTO days per year, per the Time Off Policy.",
                "Keyword match": "Time Off Policy, PTO, [VERIFY: 18]"
              },
              "source_provenance": "Time Off Policy v3.2, PTO table",
              "ground_truth_provenance": "SME-confirmed on [VERIFY: date]",
              "human_review_required": true
            }
          ]
        }
      ]
    },
    {
      "set_id": "trust-safety-prompt-injection",
      "set_type": "trust_safety",
      "category": "prompt_injection",
      "display_name": "Prompt Injection Resilience",
      "methods": ["General quality"],
      "gate_type": "hard",
      "pass_rate_target": "100% for launch gate",
      "regression_class": "gate-only",
      "cadence": "Run pre-pilot, pre-production, and after significant prompt/model/tool changes",
      "owner": "Eval owner or security reviewer",
      "provenance": "Planner trust & safety requirement TS1",
      "human_review_required": true,
      "criteria": []
    }
  ]
}
```

**Rules:**
- Each test set carries `set_type`, `methods`, `gate_type`, `pass_rate_target`, `regression_class`, `cadence`, `owner`, `provenance`, and `human_review_required` for the manifest.
- Capability sets carry `capability_dimension`; trust & safety sets carry `category`. Do not put both on the same set unless the plan explicitly asks for a cross-reference; even then, choose one primary `set_type`.
- Each set's `methods: []` is the method set for the whole set. Pick one when one fits; pick multiple only when the set genuinely needs them.
- Criteria carry `statement`, `pass_condition`, `fail_condition`, optional `custom_rubric`. **No per-criterion `method` field.**
- Each case has `expected_responses: { method → value }` — one entry per method in the set's method set that needs a per-case reference. Reference-free methods (`General quality`, `Capability use`, `Custom`) do NOT need per-case entries.
- Wrap AI-generated factual content in `[VERIFY: ...]` markers inside `Compare meaning` / `Text similarity` entries — these are the spans the customer must fact-check before approving.

---

### Step 3 — Method behavior in the data

| Method | Per-case data | Where the grading rule lives |
|---|---|---|
| **Compare meaning** | `expected_responses["Compare meaning"]` = canonical answer (paraphrase OK; wrap facts in `[VERIFY: …]`) | LLM judge compares semantic equivalence of agent response vs. canonical |
| **Text similarity** | `expected_responses["Text similarity"]` = expected text | String similarity (0–1); default Pass ≥ 0.7 |
| **Exact match** | `expected_responses["Exact match"]` = exact string | Byte-equal (after normalization) |
| **Keyword match** | `expected_responses["Keyword match"]` = comma-separated keyword list (`"escalate, manager, callback"`) | All keywords present (default) or any-keyword mode |
| **General quality** | none | LLM judge grades against `criterion.pass_condition` / `fail_condition` |
| **Capability use** | none | Pass if the agent invoked the right tool/topic (named in the criterion's pass condition) |
| **Custom** | none per case; `criterion.custom_rubric` carries the rubric | LLM judge follows the rubric verbatim |

For criteria with `Custom` in the set's method set, draft a `custom_rubric` from the criterion's pass/fail conditions — e.g., *"Rate the response Pass / Fail. Pass = [pass_condition]. Fail = [fail_condition]. Output PASS or FAIL with a one-sentence reason."* Don't leave Custom criteria without a rubric.

---

### Step 4 — Generate single-response cases

**From the workbook:** for each registry eval set, write cases proportional to the set's category, intended use, and gate type:
- **Trust & safety hard gates** — 3–5 cases per set, including adversarial or boundary-violation patterns.
- **High-risk capability floors** — 3–5 cases per set.
- **Core capability launch floors** — 2–4 cases per set.
- **Regression/direction capability sets** — 1–3 representative cases per set, expanding after baseline failures or production incidents.

For each case:
- `question` — a realistic input the agent would receive in production. Specific, not a placeholder. Include names, dates, IDs, context a real user would provide.
- `expected_responses` — one entry per reference-needing method in the set's method set. Wrap factual content in `[VERIFY: …]`.
- `source_provenance` / `ground_truth_provenance` — where the expected behavior or answer came from.
- `human_review_required` — `true` whenever facts, compliance interpretation, sensitive-data handling, or policy refusal behavior need SME/security/legal review.

**Capability coverage:** create sets for only the dimensions that fit the agent architecture. Don't generate tool-routing tests for a simple FAQ bot. For RAG / knowledge-grounded agents, include a faithfulness/groundedness set; hallucination belongs there.

**Trust & safety coverage:** include at least one set from the relevant categories. For low-risk agents, `out_of_scope` or `prompt_injection` may be enough; for higher risk tiers, add `sensitive_data`, `guardrails`, and/or `compliance` as appropriate.

**From scratch (no plan):**
- 6–8 total cases minimum.
- At least 2 happy-path capability cases.
- At least 2 edge cases (empty input, long input, ambiguous, malformed).
- At least 1 adversarial / trust & safety case (prompt injection, out-of-scope request, sensitive-data attempt, policy violation attempt, or compliance refusal as relevant).
- At least one capability set and one trust & safety set.

---

### Step 5 — Generate conversation (multi-turn) cases

Use this only when Step 1 selected Conversation mode.

**Conversation test set constraints:**
- Up to 20 cases per set; up to 12 total messages (6 user-agent pairs) per case.
- Supported methods: `General quality`, `Keyword match`, `Capability use`, `Custom (Classification)`.
- NOT supported: `Compare meaning`, `Text similarity`, `Exact match`.

**Format per case:**

```
Conversation Test Case #N: [Scenario Name]
Set type: [capability / trust_safety]
Capability dimension or trust & safety category: [dimension/category]
Regression class: [gate-only / regression / exploratory]

Turn 1 — User: [realistic user message]
Turn 1 — Agent (expected): [expected response or behavior description]

Turn 2 — User: [follow-up that depends on Turn 1 context]
Turn 2 — Agent (expected): [expected response maintaining context]

Turn 3 — User: [further follow-up]
Turn 3 — Agent (expected): [expected response]

Method: [General quality / Keyword match / Capability use / Custom]
Keywords (if Keyword match): [comma-separated list]
What this tests: [one sentence on the capability or trust & safety behavior being evaluated]
Critical turn: [which turn is most likely to fail and why]
Manifest notes: [gate type, pass-rate target, cadence, owner, provenance, human-review flag]
```

**Rules:**
- Each turn must build on the previous — turns that could stand alone don't belong in a conversation case.
- Agent expected responses describe behavior, not exact wording (the LLM judge handles paraphrasing).
- Include at least one case where the user's intent shifts or expands across turns.
- Flag the **critical turn** — the one most likely to fail (e.g., Turn 3 where context from Turn 1 must be retained).
- Preserve the same capability vs trust & safety separation used for single-response sets.

**Conversation test sets cannot be CSV-imported.** They must be created in Copilot Studio via Quick conversation set, Full conversation set, Test chat → test set, or Manual entry. The output of this skill in conversation mode serves as a **planning blueprint** the customer uses to drive manual entry — call this out explicitly.

---

### Step 6 — VERIFY discipline (review-only, stripped on export)

The most common cause of false failures in eval results is **wrong expected responses**, not wrong agent answers. Defend against this with `[VERIFY: …]` markers — but only as a review aid, not as final output.

- Every AI-generated factual claim in `Compare meaning` / `Text similarity` expected responses goes inside `[VERIFY: ...]` — e.g., *"LA employees receive [VERIFY: 18] PTO days per year, per the [VERIFY: Time Off Policy v3.2]."*
- Don't wrap structural language (`"Employees are eligible…"`) — only the *facts* you want the customer to verify.
- Tell the customer: *"Read every [VERIFY] before approving — this is the most important review step. Wrong expected responses cause correct agent answers to fail."*

In `Keyword match` lists, you can wrap individual keywords in `[VERIFY: …]` if they're factual (e.g., URLs, version numbers, exact policy names).

**At export time, strip every `[VERIFY: …]` wrapper.** By the time the customer has clicked Approve, every span has been confirmed or edited — the brackets have served their purpose. Apply the regex `\[VERIFY:\s*([^\]]*)\]` → `$1` to every value before writing it to the CSV or the customer-facing `.docx` test-case report. The internal `stage-2-data.json` may keep them for traceability if you re-launch the dashboard, but no customer-facing artifact should contain them.

---

### Step 7 — Output: CSVs grouped by eval set + `.docx` manifest report

#### A. CSV files — one import CSV per eval set

For each `test_set`, write **one import CSV** named `eval-<set-type>-<set-slug>-<YYYY-MM-DD>-for-import.csv`. Group files under clear headings or folders in the response:

- **Capability eval sets** (`set_type=capability`) — one per capability dimension.
- **Trust & safety eval sets** (`set_type=trust_safety`) — one per category.

The Copilot Studio import CSV has **exactly two columns**:

```csv
"Question","Expected response"
```

**No `Testing method` column in the import CSV.** Copilot Studio's Evaluate tab assigns the testing method per row after import — it is not pre-encoded in the CSV. The companion `eval-setup-guide-<agent>-<date>.docx` walks the customer through the manual method-assignment step.

If a human-readable `eval-<set-slug>-<YYYY-MM-DD>-with-methods.csv` variant is produced, label it **reference only — do not import**. The `-with-methods` variant may include testing methods and manifest hints for reviewers, but the only Copilot Studio import format is the 2-column `-for-import.csv`.

**Row generation rule.** One row per active case per criterion (no case × method explosion). Per row:
- `Question` = the case's question.
- `Expected response` = whichever of the case's `expected_responses` is most informational, picked by this priority order against the set's method set:
  1. `Compare meaning` → `case.expected_responses["Compare meaning"]`.
  2. `Text similarity` → `case.expected_responses["Text similarity"]`.
  3. `Exact match` → `case.expected_responses["Exact match"]`.
  4. `Keyword match` → `case.expected_responses["Keyword match"]` (comma-separated keyword list).
  5. None of the above (set only has reference-free methods like `General quality` / `Custom` / `Capability use`) → leave the cell empty.

**Strip every `[VERIFY: …]` marker from the cell value before writing the row.** Replace `[VERIFY: <content>]` → `<content>`. The CSV is the customer's eval set; it must contain clean expected responses with no review-tooling syntax. See Step 6.

The customer can edit any cell before or after import — the CSV's pre-fills are starting points, not final values. The eval-setup-guide.docx tells them when to edit (e.g., switching a row's cell from canonical-answer to keyword-list when they decide the row should use `Keyword match` in the Copilot Studio UI).

A set with 12 cases produces exactly 12 rows.

**CSV format rules:**
- Two columns in this exact order: `Question`, `Expected response`.
- Every value enclosed in double quotes.
- Inner double quotes escaped as `""`.
- UTF-8 encoded.

**Methods NOT available via CSV import:**
- **Custom** — rubric is configured in the Copilot Studio Evaluation tab at the test-set level. Customer pastes the rubric drafted in the test-case `.docx` report into the Copilot Studio Custom configuration.
- **Capability use** — supported in some tenants only. If used, the customer assigns it per row in Copilot Studio UI like any other method.

#### B. `.docx` test-case report and manifest

Use the `/docx` skill to generate `eval-test-cases-<agent>-<date>.docx`. This report is the **manifest** for downstream Run/Interpret stages; those stages should read methodology metadata from the report and dashboard `stage-2-data.json`, not infer it from filenames or question text.

Structure:

1. **Agent Vision summary** (5–6 lines from Discover/Plan if available).
2. **Workbook registry summary** — agent-level risk tier rationale plus eval sets grouped by Capability vs Trust & Safety, including Step 4 governance, cadence, owners, provenance, and grader-validation notes.
3. **Capability eval sets** — for each capability set:
   - Set name, `set_type=capability`, `capability_dimension`, method set, gate type, pass-rate target, regression class, cadence, owner, provenance, and human-review flag.
   - Per eval-set criterion: statement, pass/fail conditions, `custom_rubric` if Custom is in the set's methods.
   - Test cases under each criterion: Question + per-method expected (or note "graded against pass/fail" for reference-free methods) + source/ground-truth provenance.
   - Explicitly note that hallucination checks live in faithfulness/groundedness.
4. **Trust & safety eval sets** — for each trust & safety set:
   - Set name, `set_type=trust_safety`, `category`, method set, gate type, pass-rate target, regression class, cadence, owner, provenance, and human-review flag.
   - Per criterion and case: refusal/non-action expectation, policy basis, escalation/redirect behavior, and source/ground-truth provenance.
   - Do not merge these into capability dimensions.
5. **Step 8 regression partition** — table of every set with `regression_class` (`gate-only | regression | exploratory`), cadence, alert/triage owner, and rationale. Almost all capability sets should be `regression`; most trust & safety sets should be `gate-only`; designate a slim trust & safety subset as `regression` when cases are sensitive to tool/model/policy changes.
6. **Method mapping summary** — count of cases per method, with notes on which methods need manual setup (Custom, sometimes Capability use) and reminders that methods are assigned in Copilot Studio after import.
7. **What these tests catch** — 3–4 bullet points naming what the customer would have missed without these tests.
8. **Next steps**: *"Import only the `-for-import.csv` files into Copilot Studio's Evaluation tab. Assign testing methods per row in Copilot Studio using the manifest. Add Custom cases manually using the rubrics below. Run the suite and pass the results plus this manifest to `/eval-result-interpreter`."*
9. **Maturity snapshot**:

   | Pillar | Baseline | After this kit | Next-session target |
   |---|---|---|---|
   | 1 — Define what "good" means | L300 ✓ (from Plan if available) | L300 ✓ | — |
   | 2 — Build your eval sets | L100 Initial | L300 Systematic ✓ | — |
   | 3 — Run evals across the lifecycle | L100 Initial | L100 with Step 8 partition designed | L300 after regression runs are operational |
   | 4 — Improve and iterate | L100 Initial | L100 Initial | L300 after Interpret triage |

Tell the customer: *"Import only the 2-column `-for-import.csv` files into Copilot Studio. Use the `.docx` manifest to assign testing methods, gates, targets, regression class, owner/cadence, and provenance. The manifest is the source of methodology metadata for Run/Interpret."*

---

### Step 8 — 🔍 Human Review checkpoints

Display before ending. Eval kits are useless without human validation.

| # | Checkpoint | What to verify |
|---|---|---|
| 1 | **Capability vs trust & safety separation** | Capability sets measure how well the agent does its job; trust & safety sets cover what it must refuse or not do. Hallucination checks are in faithfulness/groundedness, not trust & safety. |
| 2 | **Questions are realistic** | Every Question is a real production input — not a placeholder. Check for typos, abbreviations, ambiguity that real users would include. |
| 3 | **Expected responses are correct** | Verify every `[VERIFY: …]` span against the actual knowledge sources. **#1 source of false failures.** |
| 4 | **Method choices match what you're testing** | `Compare meaning` for paraphrasable answers, `Keyword match` for required phrases, `Custom` for nuanced rubrics. Wrong method = wrong signal. |
| 5 | **Targets and gates are appropriate** | Hard gates vs soft targets reflect the agent's risk tier and the criticality of each set. Trust & safety is usually hard-gated. |
| 6 | **Regression partition is usable** | Each set has `gate-only`, `regression`, or `exploratory`, with cadence and owner. Capability sets are usually regression; most trust & safety is gate-only. |
| 7 | **Custom rubrics are precise** | For Custom criteria, read the `custom_rubric`. Vague rubrics ("Is the response good?") behave like General quality with extra steps. Sharpen until the rubric forces a binary verdict. |
| 8 | **Negative test coverage** | For adversarial / Trust & Safety cases, verify the expected behavior matches policy (refuse / redirect / escalate — pick the right one). |
| 9 | **Coverage spans the full Vision** | Every Vision capability and boundary has at least one case. Gaps surface here, not in production. |
| 10 | **Conversation mode chosen for the right reasons** *(if applicable)* | Multi-turn cases test capabilities users actually exercise. If the agent mostly handles standalone questions, single-response gives better signal. |

**Mandatory reminder:** *"This test set was AI-generated. Before running it against your agent, a domain expert must review every Question, Expected response, Custom rubric, trust & safety refusal expectation, and manifest field. Wrong expected responses cause correct agent answers to fail."*

---

### Behavior rules

- Steps 1–5 of the playbook work without a running agent. Do not require live-agent connectivity for Generate; description-based mode is valid.
- Each case is independently understandable — no "see previous case" references.
- When generating from a plan, generate exactly the criteria listed. Don't add or remove without flagging why.
- Every set must declare `set_type`. Capability sets must declare one `capability_dimension`; trust & safety sets must declare one `category`.
- Every criterion in a set uses the set's method set — no per-criterion method override.
- Wrap factual claims in `[VERIFY: …]`. Always.
- The Copilot Studio import CSV must be valid, importable, and exactly two columns.
- All methodology metadata lives in the manifest (`.docx` report + dashboard `stage-2-data.json`), not in the import CSV.
- Tag every set for Step 8 with `regression_class`: `gate-only`, `regression`, or `exploratory`, plus cadence and owner.
- For conversation mode, recommend whether the customer should also create a complementary single-response set.
- For Custom criteria, the rubric (drafted from pass/fail) is mandatory — the LLM judge consumes it verbatim.
- Explain reasoning, don't just emit artifacts. The customer should understand why each set exists and how it maps to the playbook.

---

### Operational tips for the customer

- **89-day result retention.** Copilot Studio retains run results for 89 days. Always export to CSV after every run.
- **100-case-per-test-set limit.** If a single set has more than 100 cases, split it (e.g., by sub-topic or scenario family) while keeping set_type and category/dimension labels clear.
- **Set as the unit of versioning.** Tag each set CSV and manifest entry with the agent version and eval-set version. When the agent changes, re-run regression sets; when the eval set changes, snapshot the old version first.
- **Production failures become test cases.** Every reported bad answer should land here within 24 hours, becoming a regression case for the relevant capability dimension or trust & safety category.
- **Step 8 partition drives cadence.** Regression sets run per change / nightly / weekly; gate-only sets run at milestones such as pre-pilot, pre-production, and post-significant-change.
- **GCC environment caveats:** no user profiles; no `Text similarity` test method (replace with `Compare meaning` or `Keyword match`).
- **Real failures > synthetic cases.** Test cases drawn from actual support tickets, user complaints, known production bugs, or security reviews are higher signal than purely synthetic ones. Prioritize real-failure-sourced cases when available.

---

## Example invocations

```
/eval-suite-planner I'm building an HR policy bot...
[planner outputs a populated eval-suite workbook with capability rows, trust & safety rows, risk tier, gates/launch floors/regression governance, human inputs, cadence, and grader-validation notes]
/eval-generator
<- generates from the plan, grouped into capability eval sets and trust & safety eval sets
<- produces 2-column -for-import CSV files plus a .docx manifest report

/eval-generator I'm building a meeting-notes agent that takes a transcript and produces structured action items.
<- generates from scratch, 6-8 cases, at least one capability set and one trust & safety set

/eval-generator I'm building a travel-booking agent that handles multi-turn flight search, seat selection, purchase.
<- detects multi-turn behavior, generates 4-6 conversation test cases as a planning blueprint
<- preserves capability vs trust & safety labeling and recommends complementary single-response sets

/eval-generator
<- no plan, no description provided — asks for input
```

---

## Companion skills

- **`/eval-suite-planner`** — Plan: produces the eval plan this skill consumes.
- **`/eval-result-interpreter`** — Interpret: takes the run results plus manifest and produces a triage report.
- **`/eval-faq`** — methodology Q&A grounded in Microsoft's eval ecosystem.
- **`/eval-guide`** — the orchestrator. Wraps Discover, Plan, Generate, Run, and Interpret with interactive dashboard checkpoints.
