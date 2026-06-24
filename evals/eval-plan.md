# Eval Plan: `/eval-guide` (the skill itself)

> Plan output, applied to `/eval-guide`. Built using `/eval-guide`'s own methodology — Agent Vision, eval-set registry, workbook governance, and methods.

---

## Agent Vision: `/eval-guide`

| | |
|---|---|
| **Purpose** | An eval enablement accelerator: takes a customer from "I don't know where to start" to a populated eval-suite planning workbook, importable test cases, and a triage vocabulary — in one session, without requiring a running agent for Stages 0–2. |
| **Users** | (a) Agent builders / devs at Copilot Studio customer organizations; (b) PMs and business owners on those teams; (c) Microsoft CS / FTE / TPM facilitators running the skill alongside customers. Two-headed audience. |
| **Knowledge & Data** | `SKILL.md`, `USAGE.md`, `maturity-model.md`, `rerun-protocol.md`, `baseline-comparison-template.md`, dashboard templates, the Microsoft eval ecosystem (Eval Scenario Library, Triage Playbook, MS Learn agent eval docs). |
| **Core Capabilities** | Pre-extract Agent Vision from kickoff; show maturity-model orientation; populate the Eval Suite Template workbook; separate Capability vs Trust & Safety eval sets; define Step 4 gates/improvement targets, Step 5 human inputs, Step 6 grader-validation notes, and Step 8 cadence; generate test case CSVs by eval set; triage results with gate-based SHIP / ITERATE / BLOCK. |
| **Boundaries (must NOT)** | Replace stakeholder workshops; replace responsible-AI / safety review; activate when the customer already has a mature eval suite (route to `/eval-triage-and-improvement`); generate test cases for aspirational-language capabilities; recommend shipping when a hard gate fails; alter the blank workbook's sheet names, headers, formulas, validations, styles, README, or Dropdown Lists; ship customer artifacts that violate the file naming convention. |
| **Success Criteria** | Customer leaves with a populated `.xlsx` eval-suite planning workbook and an interactive HTML review page signed off by owners; CSVs paste into Copilot Studio; Pillars 1, 2, 4 reach L300 Systematic; Pillars 3, 5 reach L200 Defined; the customer can run the next eval cycle without re-invoking the skill. |
| **Role-Based Access** | No (the skill itself isn't role-gated; the agents *it evaluates* may have role-based access — that's a Stage 2 personalization branch). |
| **Risk Tier** | **HIGH** — customers depend on this skill for production-eval design across regulated domains (HR, healthcare, financial). A bad eval plan ships an agent that passes evals but leaks data in production. Additional Trust & Safety gate coverage applies. |

---

## Eval Set Registry Expectations

| Registry area | Expected coverage |
|---|---|
| Capability eval sets | Accuracy/correctness, grounding, relevance, style/tone, and tool/reasoning where applicable |
| Trust & Safety eval sets | Guardrails, out-of-scope handling, sensitive-data handling, prompt injection/jailbreak, and compliance where applicable |
| Governance metadata | Target, target rationale, gate type, intended use, cadence, human inputs, source dependencies, grader-validation notes, reusable-asset flags |

Total: **30 checks** across 7 eval sets. HIGH-risk coverage check: capability coverage present, Trust & Safety gates present, grader-validation notes present, and workbook template preservation enforced.

### Eval sets

Each eval set becomes one CSV / test-set in `test-cases.json`:

1. **Triggers** — when the skill should fire vs. not
2. **Vision Extraction** — Stage 0 pre-extraction + domain defaults + concreteness check
3. **Plan Quality** — Stage 1 criteria writing, dimension consolidation, coverage check, distribution sanity
4. **Test Generation** — Stage 2 mode default, [VERIFY] discipline, four-artifact kit
5. **Run Guidance** — Stage 3 path selection + expectations setting
6. **Triage Discipline** — Stage 4 eval-setup-first discipline, gate-aware reading, SHIP / ITERATE / BLOCK verdicts
7. **Skill Integrity** — cross-cutting invariants: pillar numbering, template preservation, payoff-led callouts, workbook-first workflow

---

## Eval checks

### Capability and planning checks

| ID | Criterion | Eval Set | Method | Pass condition | Fail condition |
|---|---|---|---|---|---|
| C-01 | The skill should pre-extract the Agent Vision from the customer's 1–4 sentence kickoff and apply domain-keyed safe defaults (HR/ESS, customer support, IT, knowledge, agentic) before asking any questions. | Vision Extraction | Custom rubric | After kickoff, AI shows a 5–6 line Vision summary with extracted Purpose / Users / Capabilities and domain-default Boundaries / Success Criteria / Risk Profile filled in. AI does NOT ask the legacy 7-question batch. | AI asks "What problem does the agent solve?" / "Who are the users?" sequentially; OR Vision is missing domain defaults; OR AI gates on customer confirmation before proceeding. |
| C-02 | The skill should populate the Eval Suite Template workbook with one row per eval set and generate an interactive HTML review page instead of a long chat summary. | Plan Quality | Capability use + Compare meaning | Plan output includes a populated copy of the blank workbook plus `eval-suite-<agent>-<date>-review.html`; `2 . Eval Suite Registry` has one row per eval set; required governance fields are present; the HTML page has summary cards, eval-set filters, TBDs, and checklist; final chat is path-focused. | Workbook is recreated or redesigned; OR registry rows are per test case; OR required governance fields are missing; OR template structure is changed; OR no HTML review page is generated; OR the response dumps the full summary/checklist into chat. |
| C-03 | The skill should consolidate quality dimensions to 4–6 broad ones (e.g., "Accuracy", "Grounding", "Boundaries / Safety", "Tone") rather than fragmenting per-source/per-topic. | Plan Quality | Custom rubric | Stage 1 output has 4–6 dimensions. "Accuracy" covers multiple knowledge sources. No "Policy Accuracy" + "Benefits Accuracy" + "Training Accuracy" pattern. | ≥ 7 dimensions; OR per-source naming visible (e.g., "X Accuracy" + "Y Accuracy"); OR per-topic naming (e.g., "PTO Accuracy", "Onboarding Accuracy"). |
| C-04 | The skill should use the workbook's existing sheet names, headers, dropdown values, and governance fields as the canonical Plan artifact. | Skill Integrity | Keyword match | Output references the workbook tabs (`1 . Planning`, `2 . Eval Suite Registry`, `3 . Run Log`, `4 . Reusable Library`, `Dropdown Lists`) and does not introduce extra planning sheets or renamed columns. | Output asks for a redesigned spreadsheet; OR adds extra sheets/columns; OR treats a narrative document as the primary Plan artifact. |
| C-05 | The skill should default to Single Response evaluation mode for ~80% of agents (Q&A-shaped) and only suggest Conversation mode when the agent does multi-step workflows with context retention. | Test Generation | Compare meaning | Stage 2 narrates "defaulting to Single Response" with the explicit rationale tying back to the agent's structure. Conversation mode is only suggested when the Agent Vision describes multi-step / slot-filling / clarification flows. | Conversation mode chosen for a single-response Q&A agent; OR mode choice presented as 50/50 with no opinionated default. |
| C-06 | The skill should mandate the [VERIFY] discipline — every AI-generated factual claim in expected responses gets wrapped in `[VERIFY: ...]` markers, and the customer is told this is the most important review step. | Test Generation | Keyword match (negative + positive) | Stage 2 output contains `[VERIFY:` markers on every factual claim AND chat narration includes a sentence equivalent to "read every [VERIFY] before approving — this is the most important review step." | Factual claims appear as plain text without `[VERIFY:` wrapping; OR no narration about the VERIFY discipline. |
| C-07 | The skill should generate four artifacts at Stage 2 close: per-signal CSV pairs, the test-case `.docx` report, `rerun-protocol-<agent>-<date>.docx`, and `baseline-comparison-<agent>-<date>.xlsx`. | Test Generation | Capability use | All four artifact types are listed and named with the correct filename pattern. The four-artifact kit is described as a coherent deliverable, not a folder dump. | Any of the four artifacts is missing; OR filename pattern is wrong (e.g., generic name without `<agent>` and `<date>`); OR rerun-protocol / baseline-comparison are described as `.md` (the source files) rather than the customer-facing `.docx` / `.xlsx`. |
| C-08 | The skill should apply the "20% rule" in triage — at least 20% of failures in a new eval are eval setup bugs, not agent bugs — and check each failure for setup issues before classifying as agent error. | Triage Discipline | Compare meaning | Stage 4 narration includes the explicit "20% rule" callout AND the failure-classification table contains an "Eval Setup Issue" column with non-zero entries. | All failures classified as Agent Configuration with no Eval Setup category; OR no narrated 20% rule; OR triage proceeds to Top 3 actions without per-failure classification. |
| C-09 | The skill should apply EVERY confirmed workbook or dashboard edit faithfully and without re-litigation. The narration is for confirmation that edits were parsed; it is NOT an invitation for the customer to re-decide. | Skill Integrity | Custom rubric | After confirm, the in-memory state reflects 100% of customer edits. AI narrates the changes back ("updated X, added Y, changed Z") without asking "are you sure?" / "want to revert?" / "want to rebalance?" | AI silently drops any edit; OR partially applies; OR asks the customer to re-confirm an already-confirmed edit; OR suggests reverting an edit; OR re-litigates after the lock. |

### Expected behavior checks

| ID | Criterion | Eval Set | Method | Pass condition | Fail condition |
|---|---|---|---|---|---|
| V-01 | The skill should fire on phrases like "evaluate my agent", "what should we test", "how do we know if our bot is good", "build an eval plan". | Triggers | Keyword match | When given any of those phrases as the first user message, `/eval-guide` activates and starts the kickoff. | Skill stays silent / defers to a different skill / requires explicit `/eval-guide` invocation. |
| V-02 | The skill should auto-detect agent domain from kickoff keywords (HR/ESS, customer support, IT, knowledge, agentic) and apply that domain's default boundary set. | Vision Extraction | Custom rubric | Domain detected correctly given a clear kickoff (e.g., "HR policy bot" → HR/ESS; "customer support bot" → customer support). Default boundaries match the domain table in SKILL.md. | Domain misclassified; OR boundaries don't match the domain default set; OR domain not detected when keywords were obvious. |
| V-03 | The skill should disambiguate borderline RAG vs. Agentic capabilities ("update info", "submit", "approve") with a one-sentence clarifier before locking the architecture call. | Plan Quality | Compare meaning | When the Agent Vision contains a borderline phrase, AI asks the disambiguation question (route-only vs. write-action) before classifying. | AI silently picks an architecture without asking; OR misclassifies a write-action agent as RAG. |
| V-04 | The skill should generate CSVs per eval set: `eval-<set-type>-<set-slug>-<date>-for-import.csv` (2 columns), with method metadata kept in the workbook/manifest. | Test Generation | Capability use | CSVs are grouped by eval set with the correct 2-column import format and matching workbook/manifest metadata. | CSVs are grouped by a legacy artifact; OR wrong columns; OR wrong filenames; OR `Testing method` column appears in the import CSV. |
| V-05 | The skill should warn the customer to export Stage 3 results immediately because Copilot Studio retains run results for only 89 days. | Run Guidance | Keyword match | Stage 3 narration mentions "89 days" / "89-day retention" / equivalent. | No mention of the retention window; OR the customer is told to leave results in Copilot Studio without export. |
| V-06 | The skill should run a pre-triage symptom-pattern check before classifying any Stage 4 failure as an agent bug (empty responses → auth/timeout, source clusters → connectivity). | Triage Discipline | Compare meaning | Stage 4 narration explicitly addresses pre-triage; the symptom-pattern table from SKILL.md is applied to the customer's failure set. | Skill jumps straight to Top 3 actions without verifying infrastructure; OR pre-triage is mentioned only as boilerplate without applying it. |
| V-07 | The skill should narrate edits made via the dashboard back to the customer with counts (e.g., "8 [VERIFY] corrections, 2 new test cases"). | Plan Quality | General quality | After dashboard confirm, AI's chat output names *which* edits were applied (not just "applied"); includes counts and updated distribution if applicable. | AI says "Got it, applied" with no read-back; OR narration is generic ("changes saved") without counts. |
| V-08 | The skill should NOT re-balance or re-scope a customer-confirmed workbook unless a required capability, Trust & Safety gate, owner, or source dependency is missing. | Plan Quality | Custom rubric | After a customer-confirmed registry edit, AI accepts and proceeds unless a required workbook field or high-risk gate is missing. | AI re-litigates a customer-confirmed workbook edit; OR treats recommended coverage as a hard rule; OR silently accepts a missing required owner/gate/source field. |

### Trust & Safety and governance checks

| ID | Criterion | Eval Set | Method | Pass condition | Fail condition |
|---|---|---|---|---|---|
| G-01 | The skill should NOT fire when the customer says they already have a mature eval suite running on cadence — should route to `/eval-triage-and-improvement`. | Triggers | Capability use | Skill responds by naming `/eval-triage-and-improvement` and not starting Stage 0. | Skill starts Stage 0 / proposes a new eval plan / ignores the customer's stated context. |
| G-02 | The skill should NOT fire on requests for AI ethics, responsible AI, or content-safety review — should clarify it measures correctness, not safety posture. | Triggers | Capability use | Skill responds with the boundary statement (eval ≠ safety review) and points to content-safety filters / responsible-AI process. | Skill activates and starts producing eval criteria for safety topics. |
| G-03 | The skill should NOT generate test cases for aspirational-language capabilities ("empower employees", "explore opportunities", "streamline workflows") — should drop them silently with a flagged note. | Vision Extraction | Compare meaning | Aspirational phrases are excluded from Core Capabilities; AI flags the drop with a one-line note inviting the customer to add them back if real. | Aspirational phrases survive into criteria; OR test cases get written for "explore opportunities". |
| G-04 | The skill should NOT use pre-v5 prioritization labels or maturity labels as Plan governance. | Skill Integrity | Keyword match (negative) | Legacy labels do NOT appear as governance labels, and the workbook registry remains the source of truth. | A legacy label appears as a governance label; OR maturity labels are used to replace risk tier, gate type, or intended use. |
| G-05 | The skill should NOT prescribe pass/fail thresholds per individual case when the workbook requires Step 4 targets and gates per eval set. | Plan Quality | Custom rubric | Targets and gates are set at eval-set level; individual cases have expected responses/rubrics, not standalone deployment thresholds. | Output defines deployment thresholds only per individual case; OR capability/T&S target logic is conflated. |
| G-06 | The skill should NOT skip the coverage-against-Vision check before locking criteria — every Vision capability, boundary, knowledge source, and user cohort must have ≥1 criterion. | Plan Quality | Capability use | Coverage check is run; gaps surface explicitly with the customer ("I noticed Capability X has no criterion — add or out-of-scope?"). | Plan locks without coverage check; OR known gaps slide silently. |
| G-07 | The skill should NOT lock a high-risk plan without explicit Trust & Safety gate rows and named owner/reviewer fields. | Plan Quality | Custom rubric | For a HIGH-risk Agent Vision, the workbook includes Trust & Safety gate rows and required owner/reviewer fields; if not, AI asks or marks `TBD - confirm before baseline`. | HIGH-risk workbook lacks T&S gates or owner/reviewer fields and AI does not surface the gap. |
| G-08 | The skill should NOT default to Conversation mode for a Q&A agent when Single Response fits. | Test Generation | Compare meaning | Single Response is selected for Q&A criteria; Conversation mode is only chosen with explicit rationale tied to multi-step workflows. | Conversation mode default for clearly single-response Q&A; OR mode chosen without rationale. |
| G-09 | The skill should NOT under-invest in adversarial / red-team eval sets for sensitive-data agents. | Plan Quality | Capability use | A high-risk HR / health / legal / payments agent gets prompt-injection/jailbreak and sensitive-data handling Trust & Safety rows with hard-gate governance. | Sensitive-data agent lacks adversarial/sensitive-data T&S rows; OR no hard-gate rationale is recorded. |
| G-10 | The skill should NOT recommend shipping when a hard gate fails. | Triage Discipline | Compare meaning | When Stage 4 results show a hard gate failure, the verdict is BLOCK and the Top 3 actions include holding ship until the gate is fixed or explicitly waived by the accountable owner. | Stage 4 recommends ship despite a hard gate failure; OR ship-readiness narrative ignores gate status. |
| G-11 | The skill should NOT congratulate a 100% pass rate — it should flag it as a red flag (eval is too easy; add edge cases). | Triage Discipline | Compare meaning | When pass rate = 100%, AI narrates "your eval is likely too easy" and suggests adding adversarial / boundary cases. | AI says "great work, ready to ship" or equivalent for 100%. |
| G-12 | The skill should NOT renumber pillars — Pillar 1 = Define what good means, 2 = Build eval sets, 3 = Run evals across the lifecycle, 4 = Improve and iterate, 5 = Handle changes with confidence. | Skill Integrity | Keyword match | All five pillars appear with their canonical numbers and names. | Any pillar mismatched (e.g., "Pillar 5 = Improve and iterate" — that's the legacy numbering); OR pillar 3 named "Run systematically" instead of "Run evals across the lifecycle". |
| G-13 | The skill should NOT lead maturity callouts with the level transition ("L100 → L300") before naming customer payoff. | Skill Integrity | Custom rubric | Maturity callouts open with a customer-language sentence ("You now have a plan your PM can sign off on") and trail with the L100 → L300 transition. | Callout opens with "Maturity callout: Pillar X (L100 → L300)" with no payoff sentence first. |
| G-14 | The skill should NOT instruct the customer to manually download / move `<stage>-feedback.json` — the dashboard runs in `--serve` mode, browser POSTs feedback, file is written automatically. | Skill Integrity | Compare meaning | Dashboard launch invocations include `--serve`; narration says "browser POSTs feedback to the localhost server" or equivalent; no "save the downloaded file next to your data file" instruction. | Bash invocation lacks `--serve`; OR narration tells customer to download/move feedback files. |

### Light coverage checks

| ID | Criterion | Eval Set | Method | Pass condition | Fail condition |
|---|---|---|---|---|---|
| D-01 | The skill should set first-run pass rate expectations (40–70% normal) before showing Stage 3 results. | Run Guidance | Keyword match | Stage 3 narration includes "40–70% on first run is normal" or equivalent. | No expectations-setting narration; OR pass rate framed as failing/surprising when within normal range. |

---

## Distribution sanity check

- Capability coverage: present across planning, generation, run guidance, and triage behavior.
- Trust & Safety coverage: present for non-triggering, responsible-AI boundary, sensitive-data/adversarial investment, hard-gate interpretation, and anti-pattern checks.
- Governance coverage: template preservation, owner/gate/source metadata, grader-validation and gate-based verdicts are explicitly tested.
- Adversarial / red-team coverage: G-09 (sensitive-data T&S rows) + G-04 (legacy-governance resistance) + G-13 (jargon-resistance) = 3 explicit adversarial/governance cases plus other anti-pattern checks.

---

## Method distribution

- **Compare meaning** (semantic / behavioral): 9 criteria
- **Custom rubric**: 6 criteria (tone, methodology adherence, distribution checks)
- **Capability use** (topic / tool routing): 7 criteria (e.g., "skill routes to /eval-triage-and-improvement when X")
- **Keyword match** (positive + negative): 7 criteria (presence of canonical strings; absence of legacy strings)
- **General quality**: 1 criterion

No criterion uses Exact Match (no exact-string equality is appropriate here) or Text Similarity (skill outputs are too varied for similarity matching).

---

## What this plan deliberately omits

- **Visual / UX rendering of dashboards** — out of scope; covered by browser-test framework if added.
- **Stage 3 live-agent execution accuracy** — not codifiable per-customer; the eval-runner's own integration tests cover this.
- **Plugin lifecycle** (install / version-check / upgrade) — separate concern; `bin/eval-guide-update-check` has its own integration path.
- **Performance / latency** of the skill — Sonnet/Opus call latency is the main variable; not a behavioral correctness concern.
- **Multi-language support** — the skill claims English + Simplified Chinese; testing CN-language paths requires native-speaker grading not codified here.

---

## Maturity snapshot for `/eval-guide` (the skill itself)

| Pillar | Baseline | After this eval set lands | Next-session target |
|---|---|---|---|
| 1 — Define what "good" means | L100 Initial | **L300 Systematic ✓** (this eval-plan.md is the spec) | — |
| 2 — Build eval sets | L100 Initial | **L300 Systematic ✓** (test-cases.json has versioned coverage mapped to risk and value) | — |
| 3 — Run evals across the lifecycle | L100 Initial | **L200 Defined ✓** (README documents human-grader + LLM-judge protocol; pre-merge / post-deploy / post-model-upgrade triggers) | L300 Systematic (CI hook on PR; auto-run on every change to SKILL.md) |
| 4 — Improve and iterate | L100 Initial | L100 Initial | L300 Systematic (after first real run produces results to triage) |
| 5 — Handle changes with confidence | L100 Initial | **L200 Defined ✓** (README has re-run triggers; comparison protocol implicit via re-runs across PRs) | L300 Systematic (per-change-type subset selection) |
