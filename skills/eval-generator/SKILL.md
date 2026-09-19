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

### Step 2 — Open `data-model-and-methods.md` before constructing test sets

Read `data-model-and-methods.md` now. It contains the complete Step 2 data model and Step 3 method-behavior tables moved out of this file verbatim. Use it whenever you create capability/trust & safety set objects, select among the valid testing methods (`General quality`, `Compare meaning`, `Text similarity`, `Exact match`, `Keyword match`, `Capability use`, `Custom`), decide which per-case `expected_responses` entries are required, or draft Custom rubrics.

Do not continue case generation until you have applied the companion's rules, especially:
- capability and trust & safety are first-class, separate groups;
- at least one adversarial / trust & safety scenario is mandatory;
- factual expected-response spans need `[VERIFY: ...]` markers during review.

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

### Step 7 — Open `deliverables-and-review.md` before writing outputs

Read `deliverables-and-review.md` before exporting anything. It contains the complete Step 7 output rules and Step 8 human-review checkpoints moved out of this file verbatim. Use it for CSV filenames, CSV quoting, `.docx` manifest structure, review reminders, and final customer instructions.

Critical export invariants from that companion:
- The Copilot Studio import CSV is EXACTLY 2 columns: "Question","Expected response".
- No `Testing method` column in the import CSV; methods are assigned per row in Copilot Studio after import.
- Strip every `[VERIFY: ...]` marker from customer-facing exports.

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

### Operational tips, examples, and handoffs

Read operations-and-examples.md when preparing the customer-facing operational guidance, example invocation flow, or companion-skill handoff language. It contains those moved sections verbatim.

---
