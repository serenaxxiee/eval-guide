---
name: eval-suite-planner
description: Plan standalone — turns an Agent Vision (or plain-English description) into a structured eval plan grounded in the 10-step playbook: eval objective, risk tier, owner, capability and trust & safety eval sets, Value × Risk prioritization, pass/fail conditions, pass-rate targets, gates, methods, and human inputs. Output is a customer-ready `.docx` eval plan. Use before generating test cases or running any evals.
---

## Purpose

This skill produces the **Plan** artifact of the `/eval-guide` lifecycle: a written eval plan that a customer's PM, security partner, or business owner can sign off on. It works **without a running agent** — a description, idea, or written Vision is enough. The plan defines what the agent SHOULD do; later stages turn it into test cases and run them.

This is the standalone form of `/eval-guide` Plan. It implements the 10-step playbook's **Step 1 (Plan the eval effort)**, **Step 4 (pass-rate targets and gates)**, and **Step 5 (human inputs)**, and it plans the capability and trust & safety eval sets that downstream skills build in **Steps 2 and 3**. Use it when the customer already has an Agent Vision and wants the plan directly, or when a re-plan is needed for a specific feature without re-orienting the whole session. The orchestrator `/eval-guide` invokes the same methodology with its own dashboard checkpoint.

**Knowledge sources:**
- Canonical spine: `skills/eval-guide/playbook.md` — Microsoft's *Practical Guidance on Agent Evaluation: a 10-step playbook*. Use its terminology exactly.
- Microsoft's [evaluation iterative framework](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/evaluation-iterative-framework) and [evaluation checklist](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/evaluation-checklist) as supporting references.
- [Eval Scenario Library](https://github.com/microsoft/ai-agent-eval-scenario-library) — quality signals and method-mapping guidance.
- [Triage & Improvement Playbook](https://github.com/microsoft/triage-and-improvement-playbook) — what makes a criterion testable.

**Maturity callout — Pillar 1 (Define what "good" means):** Plan advances Pillar 1 from `L100 Initial` ("good lives in the builder's head") to `L300 Systematic` (written objective, agent-level risk tier, acceptance criteria, pass/fail conditions, pass-rate targets, gates, and human inputs). The eval plan IS the Pillar 1 artifact.

## Instructions

When invoked as `/eval-suite-planner <agent description>`:

1. Extract or accept the Agent Vision (purpose, users, knowledge, capabilities, boundaries, success criteria, `risk_profile` schema value if present). In prose, call this the agent-level **risk tier (5 factors)**.
2. Capture the Step 1 planning note: one-sentence **eval objective**, agent-level **risk tier** classified by reach, criticality of error, autonomy/blast radius, regulatory exposure, and data sensitivity, plus a **named owner**.
3. Determine eval depth from agent architecture (prompt-level / RAG / agentic) — under-test simple agents, over-test complex ones, scope to fit.
4. Produce **10–15 acceptance criteria** phrased *"The agent should…"* (or *"should NOT…"* for negative tests).
5. Assign each criterion to a `set_type`: `capability` or `trust_safety`. Trust & safety criteria also get `trust_safety_category` (`guardrails`, `out_of_scope`, `sensitive_data`, `prompt_injection`, or `compliance`) and a `gate` (`hard` or `soft`).
6. Place each criterion on the **Value × Risk matrix** (4 quadrants) for case-count/prioritization only — never for gates.
7. Assign a test method per criterion (Compare meaning / General quality / Keyword match / Capability use / Custom / Similarity / Exact match).
8. Define pass-rate targets and gate type per eval set/dimension: **hard gate** (must pass to deploy) or **soft target** (tracked, non-blocking).
9. Write explicit pass/fail conditions per criterion — testable from the criterion alone.
10. Plan Step 5 human inputs: grading rubrics, ground truths, golden answers, and a source-to-ground-truth dependency map.
11. Run distribution sanity-check (red flags only).
12. Output the customer-ready `.docx` eval plan.

Do not pad responses. Do not hedge. Be specific to the described agent — no generic advice.

---

### Step 1 — Determine eval depth from architecture

| Architecture | What it is | Eval layers to apply |
|---|---|---|
| **Prompt-level** | Single-turn LLM call, fixed system prompt, no retrieval, no tools | Capability eval sets + trust & safety refusals |
| **RAG** | Retrieves from knowledge sources before responding | + Faithfulness/groundedness + citation accuracy; hallucination is evaluated here |
| **Agentic** | Routes between topics, tools, or connectors | + Reasoning/tool-use + topic routing + slot extraction + multi-step task completion |

The architecture call drives which capability dimensions and trust & safety categories apply. Don't write tool-routing tests for a simple FAQ bot.

---

### Step 2 — Step 1 planning note: objective, risk tier, owner

Before planning any eval set, write the playbook Step 1 note:

- **Eval objective** — one sentence naming what "good" looks like and what decisions the evals inform.
- **Risk tier** — classify the whole agent using the five risk factors: reach, criticality of error, autonomy/blast radius, regulatory exposure, data sensitivity. The Agent Vision JSON field remains `risk_profile` for dashboard compatibility, but customer-facing prose should say **risk tier (5 factors)**.
- **Named owner** — one accountable person or role for authoring, reviewing, and signing off.

Keep two risk constructs separate:

1. Agent-level **risk tier** drives pass-rate targets, gate strictness, required trust & safety categories, human-review requirements, and minimum adversarial coverage.
2. Per-criterion **Value × Risk quadrant** drives case-count allocation and priority only — not gates.

---

### Step 3 — Acceptance criteria on the Value × Risk matrix

Each criterion belongs in **one quadrant** based on two judgments:
- **Value** — how much does getting this right drive the agent's mission?
- **Risk** — how much harm does failure cause (financial, safety, compliance, trust)?

|  | **Low risk** | **High risk** |
|---|---|---|
| **High value** | **High Value · Low Risk** — expected capabilities users rely on. Solid coverage; occasional misses tolerable. | **High Value · High Risk** — product-defining; failure hurts. Heaviest case investment and strict review. |
| **Low value** | **Low Value · Low Risk** — exploratory or rare. Light coverage; revisit if usage grows. | **Low Value · High Risk** — rarely triggered but must never fail. Safety, compliance, refusals. Zero tolerance. |

**The Value × Risk matrix tells you where to invest test-writing effort, not numeric thresholds or gates.** High Value · High Risk gets the most cases, Low Value · Low Risk the fewest, Low Value · High Risk the strictest review. Pass/fail per case lives in each criterion's own pass/fail conditions; pass-rate targets and hard/soft gates live at the eval-set level.

---

### Step 4 — Distribution sanity-check (reference, not gate)

Targets vary by agent-level **risk tier**. **These distribution targets are reference patterns, not gates.** Only push back on red flags.

| Risk tier | High Value · High Risk | High Value · Low Risk | Low Value · High Risk | Low Value · Low Risk | Sanity-check rule |
|---|---|---|---|---|---|
| `low`      | 30–50% | 30–50% | 10–20% | 0–20% | At least 1 Low Value · High Risk (always). |
| `medium`   | 25–40% | 25–40% | 20–30% | 0–15% | At least 1 Low Value · High Risk. |
| `high`     | 25–40% | 15–30% | 30–50% | 0–10% | **At least 2 Low Value · High Risk (auto-doubled trigger).** |
| `critical` | 20–35% | 10–20% | 40–60% | 0–5%  | At least 3 Low Value · High Risk. Compliance / safety domains required. |

**Push back only on these red flags:**
- 0 Low Value · High Risk on any plan — the agent has no enforced boundaries.
- 0 High Value · High Risk — the plan has no product-defining tests.
- >70% High Value · High Risk — every criterion is "the most important." Anchoring bias; force re-evaluation.
- HIGH risk tier + <30% Low Value · High Risk — under-investment in failure modes that cause real damage.
- CRITICAL risk tier + <40% Low Value · High Risk — same, stricter.

Marginal deviations (e.g., High Value · Low Risk at 13% with target 15–30%) are NOT red flags. Do not re-litigate customer-confirmed moves.

---

### Step 5 — Adversarial / trust & safety coverage minimums (auto-applied)

Every plan needs **at least 1 adversarial / trust & safety criterion**. The mandate **auto-doubles to 2 minimum** when any of these triggers fire:

- Risk tier is HIGH or CRITICAL.
- Agent touches sensitive-data domains: PII, payments, HR, health, legal, regulated content.
- Agent has external-customer surface area.
- Knowledge sources include personal or financial records.

When a trigger fires, narrate it: *"Your agent matches the sensitive-data trigger ([reason]) — doubling the adversarial / trust & safety coverage mandate from 1 to 2 minimum. Writing at least two adversarial criteria targeting your specific boundary risks."*

Adversarial gaps are the failure mode that bites in production: the agent passes every High Value · High Risk capability test and then leaks data on a question no one thought to write a test for.

---

### Step 6 — Eval set types: capability vs trust & safety

Group criteria into first-class eval sets by `set_type`:

**Capability eval sets** (`set_type=capability`) measure how well the agent performs its intended job. Use one dimension per set so failures are diagnostic:
- **Accuracy/correctness** — factual or task correctness.
- **Faithfulness/groundedness** — citation, source attribution, and hallucination prevention for RAG/agentic agents.
- **Relevancy** — answers the user's actual request without drifting.
- **Style/tone** — empathy, brand voice, professionalism when relevant.
- **Reasoning/tool-use** — correct tool/topic invocation, slot extraction, and multi-step task completion for agentic agents.
- **Personalization/access behavior** — when role-based access is on.

**Trust & safety eval sets** (`set_type=trust_safety`) are separate from capability. They test what the agent must refuse, avoid, or handle safely. Each trust & safety criterion must include `trust_safety_category`:
- `guardrails` — harmful, illegal, or policy-violating requests.
- `out_of_scope` — requests outside the agent's intended domain.
- `sensitive_data` — PII, PHI, financial, confidential, or role-restricted data.
- `prompt_injection` — jailbreaks, instruction override, data exfiltration attempts.
- `compliance` — regulated disclosures, required escalation, recordkeeping, or domain-specific rules.

Trust & safety is not a renamed capability dimension. It is its own group, usually a hard gate. Capability hallucination failures remain in **faithfulness/groundedness**, not trust & safety.

---

### Step 7 — Test methods (per criterion)

Pick the method based on **what you need to verify**, not on familiarity. The signal_type → method mapping:

| Signal type | What you're verifying | Method |
|---|---|---|
| **Factual content** (specific facts, numbers, IDs) | Response contains the right facts | `Compare meaning` (paraphrase OK) or `Keyword match` (exact terms required) |
| **Mandatory wording** (compliance disclaimers, citations) | Specific phrases must appear | `Keyword match` |
| **Routing / capability** | Agent invoked the right tool or topic | `Capability use` |
| **Open-ended quality** (tone, helpfulness, completeness) | Subjective rubric, no single right answer | `General quality` |
| **Domain-specific rubric** (HR / medical / legal / brand) | Custom labeled judgment | `Custom` (with a per-criterion rubric) |
| **Tight wording** (templates, structured replies) | Wording closeness | `Similarity` |
| **Exact strings** (IDs, codes, fixed responses) | Byte-exact match | `Exact match` |

**Reference-free methods** (`General quality`, `Capability use`, `Custom`) grade against the criterion's own pass/fail conditions, not against a per-case reference. They still need an `Expected response` in the Copilot Studio import CSV; keep it concise and point to the rubric or expected behavior. Method metadata travels in the manifest, not the CSV.

**`Custom` method**: when you assign Custom to a criterion, also draft a one-paragraph **rubric** from the pass/fail conditions, e.g.:
> *Rate the response Pass / Fail. Pass = [pass_condition]. Fail = [fail_condition]. Output PASS or FAIL with a one-sentence reason.*

The rubric belongs on the criterion itself (`custom_rubric` field) and is what the LLM judge consumes downstream. Flag reusable rubrics as Step 10 shared-library candidates.

---

### Step 8 — Pass-rate targets, hard gates, and soft targets

Every eval set/dimension gets an explicit pass-rate target and gate type:

- **Hard gate** — must pass before deployment.
- **Soft target** — tracked, non-blocking, and used for improvement.

Calibrate targets using the agent-level risk tier, criticality of the capability/guardrail, and realistic baseline expectations. Trust & safety sets are usually hard gates; capability sets often combine a hard floor with a soft aspiration. Example: `Accuracy` hard gate ≥90%, soft target ≥95%; `prompt_injection` hard gate 100%.

These targets flow downstream into the generator manifest and interpreter verdict, so do not leave them implicit. Do not use the Value × Risk quadrant as a substitute for a gate.

---

### Step 9 — Pass/fail conditions per criterion

Every criterion gets explicit **Pass =** and **Fail =** lines.

- Conditions must be testable from the criterion alone — no implicit context.
- Pass condition names what the response must contain or do.
- Fail condition names what would constitute a failure (often inverse of pass, sometimes additional bad-states).
- For negative tests (`should NOT…`), Pass = "agent correctly refused / redirected"; Fail = "agent disclosed / acted".

**Don't prescribe percentage thresholds per criterion.** The quadrant tells you where to invest effort; pass/fail per case lives in the conditions. Eval-set pass-rate targets and gates belong in Step 8.

---

### Step 10 — Step 5 human inputs and dependency map

Plan the human work required to make the evals trustworthy and maintainable:

- **Grading rubrics** — authored or reviewed by domain experts; reusable rubrics should be flagged as Step 10 shared-library candidates.
- **Ground truths** — authoritative facts, policy rules, database values, or source-backed expected outcomes.
- **Golden answers** — SME-authored model answers where a reference response is needed.
- **Source-to-ground-truth dependency map** — list each grounding source and which ground truths depend on it. When the source changes, the dependent ground truth must be reviewed or the eval silently drifts.

Include owner and review cadence where known. For vague inputs, state assumptions rather than inventing source authority.

---

### Step 11 — Coverage check against the Vision

Before locking the plan, walk the Agent Vision and confirm coverage:

- Every named **capability** has ≥ 1 capability criterion.
- Every named **boundary** has ≥ 1 trust & safety criterion.
- Every named **knowledge source** has ≥ 1 faithfulness/groundedness criterion (RAG/agentic only).
- Every named **user cohort** with role-based access has ≥ 1 personalization or sensitive-data criterion.
- Required trust & safety categories are present based on the risk tier and agent domain.

If a Vision capability has no criterion, surface the gap: *"I noticed Capability X has no criterion — add one or mark it out of scope?"* Don't slide gaps silently.

---

### Step 12 — Output: customer-ready `.docx` eval plan

Use the `/docx` skill to generate `eval-plan-<agent-name>-<YYYY-MM-DD>.docx`. The report must be:
- **Concise** — tables over paragraphs, no filler.
- **Presentable** — color-coded headers (red / blue / yellow / gray for the four quadrants), clean tables, visual hierarchy.
- **Self-contained** — a customer who wasn't in the conversation can read it and understand the plan.

**Report structure:**

1. **Agent Vision summary** (5–6 lines max) — purpose, users, knowledge, capabilities, boundaries, success criteria, `risk_profile` schema value if supplied; prose names the agent-level risk tier (5 factors).
2. **Step 1 planning note** — eval objective, risk tier rationale across the five factors, named owner.
3. **Value × Risk matrix overview** — explain the four quadrants and that they drive case-count/prioritization only.
4. **Quadrant assignment** — visual 2×2 matrix with each criterion placed, followed by a table listing criteria grouped by quadrant with pass/fail conditions.
5. **Capability eval sets** — capability dimensions, criteria, methods, pass-rate targets, gate type, and rationale.
6. **Trust & safety eval sets** — categories, criteria, methods, pass-rate targets, hard/soft gate, and rationale.
7. **Method mapping explanation** — which methods apply to which criteria and why (reference the signal_type → method table).
8. **Distribution check** — actual percentages vs. risk-tier reference patterns, with red-flag verdict.
9. **Adversarial / trust & safety coverage** — count of adversarial criteria; note auto-double trigger if applied.
10. **Human-input plan** — rubrics, ground truths, golden answers, owners/cadence, reusable rubric candidates, and source-to-ground-truth dependency map.
11. **Manifest handoff** — note that Copilot Studio import CSVs are exactly two columns (`Question`, `Expected response`); method, set_type, category, target, gate, regression/gate-only classification, human-review flag, and source provenance travel in the companion `.docx` manifest.
12. **Next steps** — *"Run `/eval-generator` on this plan to produce test cases for Generate. Then run them against your agent in Run and triage results with `/eval-result-interpreter` in Interpret."*
13. **Maturity snapshot** — before/after table:

   | Pillar | Baseline | After this plan | Next-session target |
   |---|---|---|---|
   | 1 — Define what "good" means | L100 Initial | L300 Systematic ✓ | — |
   | 2 — Build your eval sets | L100 Initial | L100 Initial | L300 (run `/eval-generator`) |
   | 4 — Improve and iterate | L100 Initial | L100 Initial | L300 (run `/eval-result-interpreter` after Run) |

Tell the customer: *"Here's your eval plan as a `.docx` — share it with your team. Business, dev, and security/compliance should agree on the risk tier, set separation, pass-rate targets, gates, and human inputs before we generate test cases. The Value × Risk quadrant tells you where to focus effort, not a numeric threshold or deployment gate."*

---

### Step 13 — 🔍 Human Review checkpoints

Display before ending. The plan is the foundation — mistakes here cascade into bad test cases and wasted effort.

| # | Checkpoint | What to verify |
|---|---|---|
| 1 | **Objective, risk tier, owner are real** | The eval objective is decision-oriented; the risk tier reflects all five factors; a named owner can sign off. |
| 2 | **Coverage matches the Vision** | Every named capability, boundary, knowledge source, and user cohort has ≥ 1 criterion in the right set type. |
| 3 | **Capability and trust & safety are separated** | Hallucination/grounding lives in capability; refusals, sensitive data, prompt injection, and compliance live in trust & safety. |
| 4 | **Quadrant placements match risk reality** | The Value × Risk quadrant drives prioritization only. Gates come from Step 8 targets, not the quadrant. |
| 5 | **Pass/fail conditions are decidable** | A human grader (or LLM judge) can read each pass/fail and decide the outcome from the response alone. |
| 6 | **Methods match what you're testing** | Custom for nuanced rubrics, Keyword match for required phrases, Compare meaning for paraphrasable answers. Wrong method = wrong signal. |
| 7 | **Targets and gates are explicit** | Every eval set has a pass-rate target and hard/soft gate; trust & safety hard gates are not missing. |
| 8 | **Adversarial coverage feels real** | Trust & safety criteria target *specific* boundary risks for this agent (PII for HR, payment-disclosure for billing, etc.) — not generic prompt-injection boilerplate. |
| 9 | **Human inputs are maintainable** | Rubrics, ground truths, golden answers, and source-to-ground-truth dependencies have owners/review expectations. |

**Mandatory reminder:** *"This eval plan was AI-generated from your agent description / Vision. Before proceeding to test case generation with `/eval-generator`, review the objective, risk tier, criteria, set types, quadrants, pass/fail conditions, targets, gates, and human-input plan with your team. The plan should reflect your business reality, not best-practice defaults."*

---

### Behavior rules

- Every criterion must start with *"The agent should…"* (or *"…should NOT…"* for negative tests). Behaviors, not goals.
- Every criterion has: `statement`, `set_type`, `quadrant`, `method`, `pass_condition`, `fail_condition`, `pass_rate_target`, `gate`.
- Trust & safety criteria also have `trust_safety_category` (`guardrails`, `out_of_scope`, `sensitive_data`, `prompt_injection`, or `compliance`).
- For criteria with `method: "Custom"`, also draft `custom_rubric` from the pass/fail and flag reusable rubrics as shared-library candidates.
- 10–15 criteria total. Below 10 means under-coverage; above 15 usually means fragmentation — consolidate by eval set.
- At least 1 adversarial / trust & safety criterion (2+ if the auto-double trigger fires).
- Don't prescribe percentage pass-thresholds per criterion. Pass/fail per case lives in the conditions; pass-rate targets and gates live at the eval-set level.
- Do not conflate agent-level risk tier with per-criterion Value × Risk quadrant.
- If the description is vague, state assumptions explicitly in the Vision summary at the top of the report.

---

## Example invocations

```
/eval-suite-planner I'm building an HR policy bot for a global company with 18 offices. It answers PTO, parental-leave, benefits questions from official HR documents. Should refuse salary-disclosure questions and escalate legal/discrimination concerns.

/eval-suite-planner Customer support agent for refund requests. Polite, follows refund policy, doesn't make promises beyond policy. Risk tier: HIGH (handles financial decisions).

/eval-suite-planner Email triage agent that reads incoming emails and labels them urgent / not-urgent / spam. Must NOT label real customer emails as spam.

/eval-suite-planner I have a Vision doc — purpose: code review for Python PRs, users: dev team, knowledge: PEP 8 + internal style guide, boundaries: no security review, success: PRs land faster with fewer style nits.
```

---

## Companion skills

- **`/eval-generator`** — Generate: takes this plan and produces concrete test cases. Copilot Studio import CSVs are exactly two columns (`Question`, `Expected response`); methodology metadata is carried in the companion `.docx` manifest.
- **`/eval-result-interpreter`** — Interpret: takes Run results and produces a triage report (SHIP / ITERATE / BLOCK with root-cause classification).
- **`/eval-faq`** — methodology Q&A grounded in Microsoft's eval ecosystem.
- **`/eval-guide`** — the orchestrator. Wraps Discover, Plan, Generate, Run, and Interpret with an interactive dashboard checkpoint where applicable.
