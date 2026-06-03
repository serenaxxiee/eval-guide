# The 10-Step Eval Playbook — Canonical Methodology Reference

This is the **source of truth for the methodology** used across the entire eval-guide toolkit. Every skill, prompt, dashboard, and doc derives its framing from this file. When the methodology changes, update this file first, then propagate to consumers (see *Sync rule* at the bottom).

The toolkit is grounded in **Microsoft's *Practical Guidance on Agent Evaluation* — a 10-step playbook** for building, running, and maturing eval suites for enterprise agents. Steps 1–7 follow a single agent from planning to first iteration. Steps 8–9 are the long-term monitoring and optimization loops that begin in pilot/production. Step 10 is cross-cutting — it compounds each agent's work into shared, org-wide assets.

---

## The three layers (do not conflate them)

The toolkit speaks in three registers. They are complementary, not competing — keep their roles distinct everywhere.

| Layer | What it is | Use it to… |
|---|---|---|
| **10-step playbook** | The **canonical methodology**. The spine. | Decide *what* eval work to do and in what order. |
| **Operational stages** — Discover, Plan, Generate, Run, Interpret | The **UX/workflow** the toolkit walks a customer through, each with a dashboard review checkpoint. | Drive the *session experience*. Prefer stage **names**, not numbers, in customer-facing text — avoid "Stage 3 vs Step 3 vs Pillar 3" collisions. |
| **Per-Agent Eval Maturity Model** — 5 pillars × 5 levels | An **outcome scorecard**. Progress framing, not a process. | Show the customer *where they stand* and *how far this session moves them*. Defined in `maturity-model.md`. |

---

## The 10 steps

### Step 1 — Plan the Eval Effort
Three decisions before any test case is written:
- **Eval objective** — one sentence naming what "good" looks like and what decisions the evals inform.
- **Risk tier** — classify the agent using the **five risk factors** (see glossary): reach, criticality of error, autonomy/blast radius, regulatory exposure, data sensitivity. In enterprise contexts, autonomy and regulatory exposure often dominate.
- **Named owner** — one person accountable for authoring, reviewing, and signing off.

Output: a short note — objective, risk tier + rationale, owner. No cadence/tooling/gates yet.

### Step 2 — Build the Capability Eval Sets
One eval set per **capability dimension** so failures are diagnostic. Typical dimensions: **accuracy/correctness**, **faithfulness/groundedness** (hallucination is a faithfulness failure and is caught HERE, not in trust & safety), **relevancy**, **style & tone**, **reasoning & tool use** (multi-step agents). Isolate one capability per set — never bundle.

Output: a versioned set of capability eval sets, each tagged by dimension and intended use (gate, regression, or both).

### Step 3 — Build the Trust & Safety Eval Sets
**Separate from capability.** These cover what the agent must *refuse* or *not do*, not how well it does its job. Categories: **guardrails** (refuse harmful/illegal/policy-violating), **out-of-scope handling**, **sensitive-data handling** (PII/PHI/confidential), **prompt-injection / jailbreak resilience**, **compliance-specific behaviors**. Primarily deployment gates; designate a slim subset as regression (the cases most affected by tool/model/policy changes).

Output: a versioned set of trust & safety eval sets, each tagged by category and intended use. Flag non-agent-specific sets for promotion to the shared library (Step 10).

### Step 4 — Define Pass-Rate Targets and Gates
An eval set without a target is a measurement, not a gate. For each set set an explicit pass-rate target, calibrated to risk tier (Step 1), criticality of the capability/guardrail, and realistic baseline (revisit after Step 6). Be explicit about **hard gates** (must pass to deploy) vs **soft targets** (tracked, not blocking). Trust & safety sets are typically hard gates; capability sets often combine a hard floor (e.g. 90%) with a soft aspiration.

Output: a pass-rate target + gate type (hard/soft) + rationale per set.

### Step 5 — Specify Human Inputs
Make the human work explicit so it can be planned and maintained: **grading rubrics** (domain experts), **ground truths** (authoritative sources), **golden answers** (SMEs). For each ground truth, log whether it depends on a **grounding source** (database, policy doc, knowledge base) that may change — when the source changes the ground truth must be reviewed, or the eval silently drifts.

Output: a **human-input plan** (who authors what, on what cadence) + a **source-to-ground-truth dependency map**. Flag reusable rubrics for the shared library (Step 10).

### Step 6 — Run the Baseline
Execute the full suite against the current build. Record per-set / per-case results with **timestamp + agent version** — this is the baseline all future iterations are measured against.

Output: a baseline report (pass rates, failure counts, qualitative observations) for every set.

### Step 7 — Iterate to Diagnose Failures
Case-by-case review for the first several iterations. Every failure is exactly one of:
- **Eval-setup problem** — the response is actually acceptable; the eval flagged it wrongly (too-strict rubric, stale ground truth, ambiguous case, miscalibrated grader). **Action: fix the eval.**
- **Agent-quality problem** — the eval correctly caught a real issue. **Action: log the pattern, define a fix, track it.**

Output: a **failure-pattern log** (classification, action, owner); updated eval sets where setup was wrong; an agent backlog for genuine issues.

### Step 8 — Regression Suite (Long-Term Monitoring)
Partition the suite by how often each set should run:
- **Regression sets** — run on a cadence (per change / nightly / weekly) to detect drift. Almost all capability sets + the slim trust & safety subset from Step 3.
- **Gate-only sets** — run at milestones (pre-pilot, pre-production, post-significant-change). Most trust & safety sets.

Set alerts + dashboards so regressions are noticed in hours, not weeks.

Output: a partitioned suite with documented cadence, alerting, and a triage owner.

### Step 9 — Optimization Loop (Production Data → Improvements)
Once in pilot/production, the closed cycle: **collect signals** (thumbs-down, escalations, manual overrides, support tickets, qualitative feedback) → **cluster** into patterns → **decide where the fix belongs** (agent: prompt/retrieval/tools · rubric: eval was wrong · new cases: coverage gap) → **ship** → **re-evaluate** against the regression suite (incl. new cases). Prioritize thumbs-down and other negative-feedback signals — cheapest, highest-signal source of real failure modes.

Output: a continuous production-feedback-to-improvement pipeline; an expanding regression suite; a record of each optimization cycle.

### Step 10 — Identify and Save Reusable Assets
At the end of each step ask: *could this be reused by another agent?* Anything not specific to this agent's domain is a candidate for a **shared library**. Typical candidates: trust & safety sets, grading rubrics (tone/citation/refusal/brand voice), failure-pattern templates, production-derived edge cases. Structure the shared library with three tiers:
- **Required** — every agent must run these before deploy (org-wide quality gate).
- **Recommended** — applies to most agents in a category (e.g. customer-facing).
- **Opt-in** — domain-specific; borrow when relevant.

Each asset is versioned, owned, and reviewed on a cadence. Reusable assets are the bridge from agent-level evaluation to an **org-wide eval maturity model**.

Output: a maintained shared eval library with tiered assets, clear ownership, and a promotion process.

---

## Canonical crosswalk

How the playbook, the operational workflow, and the maturity scorecard line up. This is the one table every other file should reference rather than redefine.

| Step | Operational stage (session) | Maturity pillar moved | Primary artifact | In-session? | Gate-bearing? |
|---|---|---|---|---|---|
| 1 — Plan the eval effort | **Discover** | P1 Define what "good" means | Agent Vision + risk tier + objective | Yes | — |
| 2 — Build capability eval sets | **Plan** → **Generate** | P2 Build your eval sets | Capability eval plan + CSVs | Yes | targets set (Step 4) |
| 3 — Build trust & safety eval sets | **Plan** → **Generate** | P2 Build your eval sets | Trust & safety eval plan + CSVs | Yes | usually hard |
| 4 — Pass-rate targets & gates | **Plan** | P1 / P2 | Per-set target + hard/soft gate | Yes | defines gates |
| 5 — Specify human inputs | **Plan** → **Generate** | P2 Build your eval sets | Human-input plan + source→ground-truth map | Yes | — |
| 6 — Run the baseline | **Run** | P3 Run evals across the lifecycle | Baseline report (version + timestamp) | If running agent | hard gates evaluated |
| 7 — Iterate to diagnose | **Interpret** | P4 Improve and iterate | Failure-pattern log + verdict | If results exist | SHIP/ITERATE/BLOCK |
| 8 — Regression suite | **Generate** (design) → ongoing | P3 / P5 | `rerun-protocol` + set partition + `baseline-comparison` | Designed in-session; run ongoing | gate-only vs regression |
| 9 — Optimization loop | **Interpret** (design) → ongoing | P4 Improve and iterate | `optimization-loop` reference | Designed in-session; run in prod | — |
| 10 — Reusable assets | session **closeout** | P-bridge → org maturity | `reusable-assets` closeout artifact | Yes (candidates flagged) | Required tier = org gate |

Pillar → step rollup: **P1**=Step 1 (+4) · **P2**=Steps 2,3,4,5 · **P3**=Steps 6,8 · **P4**=Steps 7,9 · **P5**=Step 8. Step 10 is the bridge to the org-wide maturity model.

---

## Glossary — terms that must be used consistently

- **Risk tier** *(agent-level)* — classification of the whole agent via the **five risk factors**: reach, criticality of error, autonomy/blast radius, regulatory exposure, data sensitivity. Drives pass-rate targets, gate strictness, which trust & safety categories are required, human-review requirements, and minimum adversarial coverage. *Do not call this "risk profile."*
- **Value × Risk quadrant** *(per-criterion)* — a prioritization tool placing each acceptance criterion in one of four quadrants by Value (mission impact) × Risk (harm of failure). Drives **case-count allocation and priority only** — not gates. Distinct from the agent-level risk tier.
- **Capability eval set** — a set measuring one capability dimension (accuracy, faithfulness/groundedness, relevancy, style/tone, reasoning/tool-use). Hallucination = a faithfulness failure → lives here.
- **Trust & safety eval set** (`set_type=trust_safety`) — a set covering refusal/policy behavior, tagged by `category`: `guardrails | out_of_scope | sensitive_data | prompt_injection | compliance`. Separate from capability sets; usually hard gates.
- **Hard gate** — must pass before deploy. **Soft target** — tracked but non-blocking.
- **Regression set vs gate-only set** — regression sets run on a cadence to catch drift (≈all capability + slim T&S subset); gate-only sets run at milestones (most T&S).
- **Eval-setup problem vs agent-quality problem** — the two and only two root buckets for any failure in Step 7.
- **Manifest** — the methodology metadata that the 2-column Copilot Studio CSV cannot carry (set_type, category, method, gate type, target, regression class, human-review flag, source/ground-truth provenance). Carried in the companion `.docx` report and the dashboard `stage-N-data.json`. Interpreters prefer the manifest over inferring from filenames or question text.

---

## CSV contract

The Copilot Studio import CSV is **exactly two columns: `Question`, `Expected response`** — one row per case. The **testing method is assigned per row in Copilot Studio's Evaluate tab after import**, not encoded in the CSV. Group cases by quality signal / eval set into separate CSV files (`eval-<signal-slug>-<YYYY-MM-DD>.csv`). All other methodology metadata travels in the manifest (`.docx` report + `stage-N-data.json`), never in the CSV.

Valid testing methods (assigned in the UI): `General quality`, `Compare meaning`, `Similarity`, `Exact match`, `Keyword match` (Copilot Studio core five), plus `Capability use` and `Custom` (extensions).

---

## Invariants (preserve across all skills)

- Steps 1–5 (Discover/Plan/Generate) work **without a running agent** — description-based mode is the default; live-agent connection is an enhancement.
- **Architecture-aware scoping** — prompt-level vs RAG vs agentic changes which capability sets apply. Don't generate tool-routing tests for a simple FAQ bot.
- Every eval plan includes **at least one adversarial / trust-&-safety scenario**.
- **Explain reasoning, don't just emit artifacts** — the customer should learn the methodology.
- The dashboard is the **review checkpoint** — no `.docx`/`.csv` is generated until the customer confirms.

---

## Sync rule

When this file changes, propagate to:
- `skills/eval-guide/maturity-model.md` (scorecard pillar→step mapping)
- `skills/eval-guide/SKILL.md` (playbook-mapping section, per-stage callouts, snapshot tables)
- `skills/eval-guide/USAGE.md`
- `skills/eval-{suite-planner,generator,result-interpreter,triage-and-improvement,faq}/SKILL.md` + their `.github/prompts/*.prompt.md` mirrors
- `skills/eval-guide/dashboard/` (serve.py framing strings, templates, examples, orient rebuild)
- `AGENTS.md`, `README.md`, `CLAUDE.md`, `.github/copilot-instructions.md`

Sync is manual and copy-by-rule. Every consumer should **point to this file**, not restate the methodology.
