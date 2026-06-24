# Eval Guide — How to Use This Skill

Your step-by-step guide to running a `/eval-guide` session. Written for you, the person running the session — not the AI. It tells you what to bring, what to decide, and what you'll walk away with.

> **Install:** see the repo [README](../../README.md#install). This guide assumes the skill is already installed and you're about to type `/eval-guide` in Claude Code or GitHub Copilot.

## 1. What this is

`/eval-guide` is an eval enablement accelerator. In one session it takes you from "I don't know where to start" to a populated eval-suite planning workbook, a set of test cases importable into Copilot Studio, and the vocabulary to triage results the next time evals come back.

**This guide is for:**
- **Customers** — Copilot Studio agent builders, product managers, or business owners — running `/eval-guide` on their own.
- **Internal team members** (CS, FTE, TPM) facilitating a session *alongside* a customer. Use as a shared script so you both know what's coming.

**You are the right audience if you are:**
- New to eval, or new to doing eval *systematically*.
- Working on any agent architecture: prompt-level, RAG, or agentic.

**Use this when you:**
- Are planning a new agent.
- Are adding a feature to an existing agent.
- Have an agent but have never written a real eval.
- Got eval results back and don't know what they mean.

**Do not use this when you:**
- Already have a mature eval suite running on cadence — use `/eval-triage-and-improvement` instead.
- Need ethics, responsible AI, or content-safety review — eval measures correctness, not safety posture. Use content safety filters alongside.

## 2. Before you start

**What you need ready:**
- A description of the agent (idea, draft, spec, or live) — see `workshop/sample-agent.yaml` for a well-formed example.
- A browser. Dashboards open as local HTML files.
- Python 3 on your machine (no dependencies) to launch the dashboards.
- About **30 minutes** of focused time.

**What you do NOT need:**
- A running agent. Stages 0, 1, and 2 work from a description.
- A DirectLine endpoint, tenant ID, or a test environment — nice to have, not required.
- Written requirements or a PRD. The Discover conversation draws these out of you.

## 3. The Eval Maturity Journey

Eval maturity has five pillars and five levels each — from `L100 Initial` (no practice in place) to `L500 Optimized` (continuous improvement built into operations). Today's session takes Pillars 1, 2, and 4 to **L300 Systematic** through in-session work, and Pillars 3 and 5 to **L200 Defined** through reference protocols you keep after the session. The full 5×5 lives in `maturity-model.md`.

| Pillar | What it covers | Today's session | Where you land |
|---|---|---|---|
| **1. Define what "good" means** | Agent Vision, one-sentence eval objective, five-factor risk tier, owner, and sign-off gates | **Delivered (Discover + Plan)** | L300 Systematic |
| **2. Build your eval sets** | Capability and Trust & Safety eval sets in the workbook registry, then CSVs ready for Copilot Studio | **Delivered (Plan + Generate)** | L300 Systematic |
| **3. Run evals across the lifecycle** | When and where evals execute (offline, pre-deploy, in production) | **Starter delivered (`rerun-protocol-<agent>-<date>.docx`)** | L200 Defined |
| **4. Improve and iterate** | Root-cause triage, failure patterns, next-action playbook | **Delivered (Stage 4 — if eval results are available)** | L300 Systematic |
| **5. Handle changes with confidence** | Comparing eval runs, validating prompt/tool/model changes before shipping | **Starter delivered (`baseline-comparison-<agent>-<date>.xlsx`)** | L200 Defined |

**Why Pillars 3 and 5 stop at L200 Defined:** they aren't single-session deliverables — they're ongoing operating practices. Pillar 3 needs a release cadence with codified triggers (CI hooks, scheduled runs, production-quality tracking); Pillar 5 needs version-tagged baselines accumulated over multiple changes. The two starter artifacts you'll receive at session close (`rerun-protocol-<agent>-<date>.docx` and `baseline-comparison-<agent>-<date>.xlsx`) give you the documented protocol and fill-in workbook to execute when triggered — the L200 Defined milestone — and the path to L300 Systematic is described inside each artifact. Once you have a running agent and a few changes' worth of comparison history, come back to push them to L300.

## 4. Stage 0 — Discover *(advances Pillar 1)*

**Goal:** Articulate what your agent does and what "good" looks like.

### 0.1 Kick off

- **What you provide:** One or two sentences — *"I'm building / planning / evaluating an agent that does X for Y users."*
- **What you get back:** The AI confirms the mode (idea / description / live agent) and begins the Discover questions.

### 0.2 Answer the seven Discover questions

- **What you provide:** Plain-language answers to:
  1. What problem does the agent solve?
  2. Who are the users?
  3. What knowledge sources will it use?
  4. What must it do — and not do?
  5. What does success look like?
  6. What's the cost of getting it wrong?
  7. Does behavior differ per user role?
- **What you decide:**
  - **Risk tier (low / medium / high)** — based on reach, criticality of error, autonomy/blast radius, regulatory exposure, and data sensitivity. It informs gate strictness, Trust & Safety coverage, and human-review needs.
  - **Role-based access (yes / no)** — if yes, Stage 2 generates separate test sets per role using Copilot Studio user profiles. *Note: multi-profile eval doesn't work with connectors and isn't available in GCC.*
- **What you get back:** An **Agent Vision** block (name, purpose, users, knowledge sources, capabilities, boundaries, success criteria, role-based access, risk tier).

### 0.3 Confirm the Agent Vision

- Review the Agent Vision in chat. Add anything missing. Say "confirmed" when it reflects reality.
- **What you get back:** `stage-0-data.json` on disk. *No dashboard at this stage — the conversation is the checkpoint.*

## 5. Stage 1 — Plan *(advances Pillar 1)*

**Goal:** Turn the Agent Vision into a populated Eval Suite Template workbook that follows the 10-step guidance.

### 1.1 Confirm the architecture

- **What you decide:** Prompt-level / RAG / Agentic. The AI proposes based on knowledge sources and tools; you confirm.
- **Tradeoff:** Over-scoping wastes effort on criteria that never matter; under-scoping misses real failure modes.
- **What you get back:** Eval layers to apply (RAG adds grounding + hallucination; Agentic adds tool-selection + task-completion).

### 1.2 Populate the workbook

- **Goal:** Fill a copy of the blank Eval Suite Template without changing its structure.
- **What the AI populates:**
  - `1 . Planning`: agent identity, objective, five-factor risk tier, owner, stakeholder roles, sign-off criteria.
  - `2 . Eval Suite Registry`: one row per eval set, separated into Capability and Trust & Safety rows, with Step 4 gates/improvement targets, intended use, cadence, human inputs, source dependencies, grader-validation notes, and reusable-asset flags.
  - `3 . Run Log`: baseline placeholders, if useful.
  - `4 . Reusable Library`: reusable eval assets and candidate tiers.
- **What stays unchanged:** README, `Dropdown Lists`, sheet names, headers, formulas, validations, widths, styles, and workbook structure.
- **What you get back:** `eval-suite-<agent>-<date>.xlsx` plus `eval-suite-<agent>-<date>-review.html`. The workbook is the source of truth; the HTML page is the interactive review surface for summary cards, filters, TBDs, and the checklist.

## 6. Stage 2 — Generate *(advances Pillar 2)*

**Goal:** Turn each workbook registry eval set into concrete test cases, grouped into CSVs importable into Copilot Studio.

### 2.1 Choose the evaluation mode per criterion

- **What you decide:**
  - **Single response** (up to 100 cases, all 7 methods) — use for independent Q&A criteria.
  - **Conversation / multi-turn** (up to 20 cases, max 6 Q&A pairs, limited methods) — use for slot-filling, clarification flows, or multi-step workflows.
- **Tradeoff:** Conversation mode matches real user behavior but caps at 20 cases and drops `Compare meaning`, `Text similarity`, and `Exact match`.

### 2.2 Review generated test cases

- The AI generates from the plan. Share real production phrasings now if you want them used.
- Factual content in expected responses is wrapped in `[VERIFY: …]` markers in the dashboard so you can spot-check it. The markers are stripped automatically when the CSV and the test-case `.docx` are generated — the customer-facing artifacts ship clean.

### 2.3 Generate dashboard checkpoint

- **What happens:** The AI launches the generate dashboard from the eval-guide plugin install. Your browser opens `generate-dashboard.html`.
- **What you do in the browser:**
  - **Eval Sets Overview at the top** — a table listing every workbook eval set with set type, # test cases, test methods, gate type, target, cadence, and owner. Edits below update this table in real time.
  - **Stacked eval-set sections** — grouped by Capability and Trust & Safety. Hard-gated Trust & Safety sets are visibly marked.
  - **"Test Methods to Use:"** bar at the top of each eval-set section lists the methods that apply to that set. Hover a chip and click × to remove. Use **+ Add method** to add another.
  - **Case cards/tables** show the question, expected response/rubric fields, and workbook metadata needed for review.
  - **Custom rubric callout** appears when `Custom` is in the set's methods — an editable LLM-judge rubric drafted from the set purpose and pass/fail expectation. Edit it for your domain.
  - **A small reference-free note** appears when `General quality` or `Capability use` is in the set's methods — those methods grade against the set's rubric/conditions, not a reference, so they don't add a per-case column.
  - **Test cases table** has columns driven by the set's reference-needing methods: one column for `Question`, then one column per method that needs a per-case reference (`Compare meaning`, `Text similarity`, `Exact match`, `Keyword match`). Each cell is editable. `[VERIFY: …]` spans in `Compare meaning` / `Text similarity` cells are highlighted yellow — fact-check before approving. The brackets are stripped automatically when the CSV and `.docx` are generated, so the customer-facing files ship clean.
  - Add or delete test cases with the per-row buttons.
  - Click **Approve & Continue to Next Stage** or **Incorporate Changes & Generate New Plan**.
- **What happens when you click:** Your edits go straight from the browser to the localhost dashboard server, which forwards them to the AI's terminal output and shuts down. **No download, no file to move.** The AI applies your edits and either generates the deliverables (Approve) or re-launches a fresh dashboard with the changes already incorporated (Regenerate).
- **What you get back (after Approve):**
  - **One CSV per eval set** — e.g. `eval-capability-accuracy-<date>.csv`, `eval-trust-safety-sensitive-data-<date>.csv`, etc. **Two columns only: `Question`, `Expected response`.** No testing method column — that is set manually per row in Copilot Studio's Evaluate tab UI after import. The `eval-setup-guide-<agent>-<date>.docx` walks you through that step in detail.
  - For methods that grade against pass/fail (`General quality`, `Capability use`, `Custom`), the `Expected response` cell is empty — Copilot Studio uses the criterion's pass/fail (and for `Custom`, the rubric you set in the test-set configuration).
  - A customer-ready test case report, if requested — test cases grouped by eval set with methods, workbook metadata, and a "What these tests catch" callout.

## 7. Stage 3 — Run *(Pillar 3 starter — skip if agent isn't built)*

**Goal:** Execute the CSVs against a live agent.

This session reaches **L200 Defined on Pillar 3** through the `rerun-protocol-<agent>-<date>.docx` reference document you'll receive at session close — a documented protocol for re-running evals when the agent changes. L300 Systematic on Pillar 3 (offline + production evals running on a defined cadence with production-quality tracking) requires automation and production signal that the starter doc points toward but doesn't deliver. Run Stage 3 yourself later when the agent is ready; the rerun protocol tells you when to trigger and what scope to run.

If the agent IS running:

- **What you provide:** DirectLine token endpoint, or access to `/chat-with-agent` via the Copilot Studio plugin.
- **What you decide:** Which CSVs to run now vs. later. Run hard-gated Trust & Safety sets and directly impacted capability regression sets first.
- **What you get back:** `eval-results-YYYY-MM-DD.csv` and `.json`. **Export immediately** — Copilot Studio only retains results for 89 days.
- **Checkpoint:** None. This stage executes; no dashboard.

## 8. Stage 4 — Interpret *(advances Pillar 4)*

**Goal:** Turn raw results into a ranked list of actions.

### 4.1 Provide results

- **What you provide:** `eval-results-*.csv` from Stage 3, a pasted summary, or exported Copilot Studio results.
- **What you get back:** Total / passed / failed counts, pass rate by eval set and method, gate status, and regression/direction notes.

### 4.2 Pre-triage check

- Confirm knowledge sources were reachable, APIs healthy, auth valid during the run. If anything was broken, the run is invalid.

### 4.3 Root-cause classification

- Apply the "at least 20% of failures are eval bugs, not agent bugs" lens.
- **What you get back:** Each failure classified as **Eval Setup Issue** / **Agent Configuration Issue** / **Platform Limitation**, plus a Top 3 actions list formatted as **Change X → Re-run Y → Expect Z** (always re-running the full set, not just failing cases, so regressions surface).

### 4.4 Interpret dashboard checkpoint

- **What happens:** The AI launches the interpret dashboard from the eval-guide plugin install. Your browser opens `interpret-dashboard.html`.
- **What you do in the browser:**
  - Scan the **gate verdict and eval-set summary** — hard gate status first, then capability target/regression status.
  - Expand criterion rows to see every test case with the LLM judge's explanation.
  - Click **Agree** / **Disagree** per case. Disagrees flip the case to an Eval Setup root cause — your human judgment overrides the LLM judge.
  - Reclassify root causes via the dropdown.
  - Edit the Top 3 actions if the AI missed context.
  - Click **Approve & Continue to Next Stage** or **Incorporate Changes & Generate New Plan**.
- **What happens when you click:** Your edits go straight from the browser to the localhost dashboard server, which forwards them to the AI's terminal output and shuts down. **No download, no file to move.** The AI applies your edits and either generates the triage report (Approve) or re-launches a fresh dashboard with the changes already incorporated (Regenerate).
- **What you get back:** A **`.docx` triage report** — SHIP / ITERATE / BLOCK verdict, eval-set gate table, failure triage table with human-disagreed entries flagged as *"Eval Setup — Human Disagrees"*, Top Actions, pattern analysis, and next steps.

## 9. After the session

**You walk away with:**
- `stage-0-data.json` — confirmed Agent Vision.
- `.xlsx` eval-suite planning workbook (Plan) populated from the blank template.
- One CSV per eval set (Generate): `eval-<set-type>-<set-slug>-<date>.csv` — 2 columns (Question, Expected response), one row per case. Testing method is set manually per row in Copilot Studio's Evaluate tab UI; the `eval-setup-guide-<agent>-<date>.docx` walks through that.
- `.docx` test case report (Stage 2).
- *If Stage 3 ran:* results CSV/JSON and `.docx` triage report (Stage 4).
- **`eval-setup-guide-<agent>-<date>.docx`** — step-by-step walkthrough for setting up and running the CSVs in Copilot Studio's Evaluate tab. Per-method setup details (`General quality`, `Compare meaning`, `Keyword match`, `Custom`, etc.), threshold guidance tied to workbook eval-set gates/targets, and a troubleshooting table for common import/run problems. Open it the first time you set up the run and any time someone new on the team picks it up.
- **`rerun-protocol-<agent>-<date>.docx`** — Pillar 3 L200 Defined starter. Reference document — when to re-run evals after the agent changes, what scope to run, how to log results, exit criteria for L200, path to L300. Read it, share it with your team, keep it next to your eval set.
- **`baseline-comparison-<agent>-<date>.xlsx`** — Pillar 5 L200 Defined starter. Fill-in Excel workbook — comparison table for Run 1 vs. Run 2 metrics, four case-level buckets (Pass-Pass / Fail-Pass / Pass-Fail / Fail-Fail), decision rules, capability-vs-regression cheat sheet. Open it each time you compare two eval runs.
- The vocabulary to do the next round yourself: eval objective, risk tier, Capability vs Trust & Safety eval sets, hard gates vs soft targets, `[VERIFY]` discipline, grader validation, and Step 7 root-cause classification.

**How to push Pillars 3 and 5 from L200 Defined to L300 Systematic:**
- **Pillar 3 (Run evals across the lifecycle)** — your `rerun-protocol-<agent>-<date>.docx` gets you to L200 Defined: a documented protocol you execute when triggered. L300 Systematic requires automation (CI hooks, scheduled runs) and production-quality tracking on a defined cadence. Come back when you have a DirectLine endpoint and want to codify the triggers and start sampling production traffic.
- **Pillar 5 (Handle changes with confidence)** — your `baseline-comparison-<agent>-<date>.xlsx` gets you to L200 Defined: a fill-in workbook for comparing two runs. L300 Systematic requires per-change-type routing (a prompt edit triggers the prompt subset; a tool change triggers the tool-routing subset) and at least three changes' worth of comparison history. Come back when you've accumulated that history.

**How to re-run as the agent evolves:**
- Any change to knowledge, topics, or tools → follow the trigger table in your `rerun-protocol-<agent>-<date>.docx`. Hard-gated Trust & Safety sets and directly impacted capability regression sets run first, then the prescribed scope.
- New feature → new `/eval-guide` session, jumping to Stage 1 with the existing Agent Vision as input.
- Comparing two runs → open your `baseline-comparison-<agent>-<date>.xlsx`, fill in the Comparison sheet and Case-level delta sheet. Pass-Fail (regression) cases are highest priority.
- Production signal (real user issues) → add cases to the relevant eval-set CSV, re-run, re-interpret.
- **Export every run's results to CSV immediately** — Copilot Studio retains them for only 89 days.

**A 100% pass rate is a red flag, not a trophy** — it means your eval is too easy. Add edge cases and adversarial criteria before trusting the number.
