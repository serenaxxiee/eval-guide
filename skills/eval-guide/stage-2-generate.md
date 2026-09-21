<!-- Companion reference for the `eval-guide` skill. Loaded on demand from SKILL.md. -->

## Stage 2: Generate

Generate test cases as **separate CSV files per eval set** from the workbook registry. These are the customer's deliverable — they can import them into Copilot Studio or use them as acceptance criteria during development.

**Which sets to generate comes from `skills/eval-guide/targeted-eval-sets.md`** — the canonical generation catalog. It defines the three categories (common capabilities, trust & safety, agent-specific instruction-following), the signal-to-dimension mapping, architecture gating, and the workbook mapping. Read it before generating.

### What you walk away with (one kit)

| Artifact | Use it for |
|---|---|
| `eval-<set-type>-<set-slug>-<date>.csv` (per eval set — **2 columns: Question, Expected response**; one row per case; the Testing method is assigned per row in Copilot Studio's Evaluate tab after import) | Paste directly into Copilot Studio Evaluation tab |
| `eval-test-cases-<agent>-<date>.docx` | PM / stakeholder review |
| `eval-setup-guide-<agent>-<date>.docx` | Step-by-step walkthrough for setting up + running the eval in Copilot Studio's Evaluate tab |
| `rerun-protocol-<agent>-<date>.docx` | Pillar 3 L200 — when to re-run the eval as the agent changes |
| `baseline-comparison-<agent>-<date>.xlsx` | Pillar 5 L200 — your version-comparison workbook |

The kit is one deliverable. CSVs go to Copilot Studio. The test-case .docx goes to your PM. The setup guide, rerun protocol, and baseline-comparison workbook go to your eval-process docs.

### When this stage is wrong for you

- You already have a test set you trust. Bring it; skip to Stage 3.
- You have production traffic. Sample real conversations directly into a test set rather than synthesizing — generated cases anchor to AI voice; real user language beats it.
- You're testing agent UX (turn-taking, error-recovery flow). That's conversation testing, not eval — different tool.

### Choose evaluation mode: Single Response vs. Conversation

**Default to Single Response.** ~80% of agents are single-response Q&A. Conversation (multi-turn) only fits agents that do real multi-step workflows — troubleshooting flows, form-filling, slot-extracting conversations. If you're not sure, you don't need Conversation mode.

| Mode | Best for | Limits | Supported test methods |
|---|---|---|---|
| **Single response** *(default — fits ~80% of agents)* | Factual Q&A, tool routing, specific answers, safety tests | Up to 100 test cases per set | All 7 methods (General quality, Compare meaning, Keyword match, Capability use, Text similarity, Exact match, Custom) |
| **Conversation (multi-turn)** | Multi-step workflows, context retention, clarification flows, process navigation | Up to 20 test cases, max 12 messages (6 Q&A pairs) per case | General quality, Keyword match, Capability use, Custom (Classification) |

**When to switch to conversation eval:**
- The agent walks users through multi-step processes (e.g., troubleshooting, onboarding, form completion)
- Context retention matters — later answers depend on earlier ones
- The agent needs to ask clarifying questions before answering
- The criterion involves slot-filling or information gathering across turns

**When to stay with single response (the default):**
- Each question is independent (FAQ, policy lookup, data retrieval)
- You need Compare meaning, Text similarity, or Exact match (conversation mode doesn't support these)
- You need more than 20 test cases in a set

**Explain the choice:** "I'm recommending single response eval for your knowledge-lookup criteria because each question is independent — the agent doesn't need previous context to answer. For your troubleshooting criterion, I'm recommending conversation eval because the agent needs to gather information across multiple turns before resolving the issue."

**Note for CSV generation:** Single response test sets use the **2-column import CSV** (`Question`, `Expected response`); the testing method is assigned per row in Copilot Studio's Evaluate tab after import (see the manifest note below). Conversation test sets can be imported via spreadsheet or generated in the Copilot Studio UI — each test case contains a sequence of user messages that simulate a multi-turn interaction.

### Agent instructions branch — ask this first, before anything else in Stage 2

Two of the three generated categories are predictable from the Agent Vision. The third — **agent-specific instruction-following** — is not. It comes from what this agent was actually told to do, and it's usually the highest-signal part of the kit, because a failure points straight at the instruction the agent ignored. It cannot be generated without the instruction block.

**Check first, then ask.** If Stage 0 captured `agent_instructions`, or the conversation / attachments / workbook `Notes` already carry the instruction block, use it and say so. **Don't ask twice.**

**Otherwise ask exactly one question and wait for the answer:**

> *"Before I generate — want to paste your agent's instructions (the system prompt / instruction block from Copilot Studio)? I'll turn each testable instruction into its own small eval set, so when something fails you know which instruction was ignored. Without them I'll still generate common capability and trust & safety sets, which cover most of the kit."*
>
> Options: **Paste the instructions** · **Point me at a file or attachment** · **Skip — generate common sets only**

**If supplied**, mine them with the catalog's testability filter — observable in a bounded response · has a trigger you can write a question for · has a discriminating negative — then confirm the extraction before generating:

> *"From your instructions I can test N behaviors: [each quoted verbatim]. Dropping M as untestable: [list with one-line reasons]. I'll generate one eval set per testable instruction, each with a positive trigger and a negative control."*

**If the customer skips**, generate Categories 1 and 2 and name the gap plainly rather than letting it pass silently: *"No instruction-following sets — I don't have your agent's instructions. Those are the ones that catch behaviors specific to how you told this agent to act. Re-run Generate with your instruction block whenever you want them."* Carry the gap into `stage-2-data.json`, the dashboard, the `.docx` manifest, and the human-review checklist.

**Never block on this, and never bundle it with other questions.** One question, one answer, then generate either way. Never invent instructions the customer didn't write.

### Personalization branch — handle this before generating test cases

If the Agent Vision has `role_based_access: true` (set in Discover), the test cases for personalization criteria need **user profiles** in Copilot Studio. Without profiles, the agent has no context to personalize from — and the test results are misleading.

**Walk the customer through this BEFORE generating cases:**

1. **Identify which criteria need profiles.** Check the criteria list for ones that test personalization (criteria mentioning "for the employee's [attribute]" — office, tenure, plan, role, etc.).

2. **Draft 3 user profiles that span the personalization axes.** Pick combinations that exercise different paths:
   - Profile A: one attribute combo (e.g., `Boston-2yr-PPO`)
   - Profile B: a contrasting combo (e.g., `Seattle-7yr-HMO`)
   - Profile C: an edge combo (e.g., `Remote-FirstYear-HDHP`)
   - Each profile has explicit attribute values and a one-line note on which criteria it exercises.

3. **Tell the customer to create the profiles in Copilot Studio** (Settings → Evaluation → User Profiles) before importing test sets. The CSV import won't fail without profiles, but personalization-criterion results will be misleading.

4. **Flag the two known limitations:**
   - **Multi-profile eval doesn't work with connector-based agents.** If the Vision includes any tool/connector use, multi-profile eval can't run against those criteria — fall back to standard cases without profile context.
   - **Multi-profile eval is not available in GCC.** Ask the customer's tenant type: standard or GCC. If GCC, drop personalization test cases or run them as standard cases (lose the personalization signal).

5. **Generate one test case set per criterion per profile**, OR a single set with profile-tagged expected responses (when criterion is the same question, different expected answer). Use whichever is more efficient.

**If `role_based_access: false`**, skip this branch entirely — no profile setup needed.

### The [VERIFY] discipline — the most important review step in the whole skill

When generating expected responses, the AI wraps factual content it can't independently confirm in `[VERIFY: ...]` markers. **These are the failures-in-waiting.** A wrong [VERIFY] becomes an eval test case that "passes" while hiding a production failure — the agent matches the bogus expected response and gets a green check.

The dashboard highlights every [VERIFY] span in yellow. **Read every one before approving.** This is the customer's most important responsibility in Stage 2; the LLM that drafted the test cases cannot do this work — only the human who knows the actual knowledge sources can.

When narrating to the customer, say: *"I've wrapped factual claims I'm guessing at in [VERIFY] markers. Please check each one against your real knowledge source — these are the most likely places the eval will lie to you about agent quality."*

### What to do

1. Generate one or more test cases per acceptance criterion from the plan. For conversation criteria, generate multi-turn test cases with realistic dialogue sequences (up to 6 Q&A pairs). A single criterion can and often should have multiple test cases exercising different phrasings, user contexts, and edge inputs. For instruction-following sets, keep them small — 2–4 cases, with at least one positive trigger and one negative control where the behavior should *not* fire.

2. **Write expected responses so they satisfy the criterion's pass condition** — i.e., what the agent SHOULD say according to the Agent Vision, the criterion's statement, and its pass_condition. Note: "These expected responses reflect your stated requirements. Refine them once the agent is built and you see how it actually responds."

3. **Group by eval set** into separate CSV files:
   - `eval-capability-accuracy-correctness.csv`
   - `eval-capability-faithfulness-groundedness.csv`
   - `eval-capability-reasoning-tool-use.csv`
   - `eval-trust-safety-sensitive-data-handling.csv`
   - `eval-trust-safety-prompt-injection-jailbreak.csv`
   - `eval-trust-safety-compliance-specific.csv` (if applicable)
   - `eval-instruction-following-<instruction-slug>.csv` — one per testable instruction, if instructions were supplied (e.g. `eval-instruction-following-cite-policy-section.csv`)

   Only create files for categories that apply.

   **Versioning:** Name each file with a date stamp or agent version (e.g., `eval-knowledge-accuracy-2026-04-22.csv`) so successive sessions produce a version history rather than overwriting the baseline. Versioning is a requirement of L300 Systematic Pillar 2.

4. **CSV format** — Copilot Studio import format is **exactly two columns**:

```csv
"Question","Expected response"
"How many PTO days do LA employees get?","LA employees receive 18 PTO days per year."
```

The **Testing method is NOT a CSV column** — it is assigned per row in Copilot Studio's Evaluate tab after import (see the manifest note below). The method chosen for each criterion travels in the companion `.docx` manifest and the `eval-setup-guide-<agent>-<date>.docx`, which walk the customer through the manual per-row assignment. Valid Testing method values (assigned in the UI): `General quality`, `Compare meaning`, `Text similarity`, `Exact match`, `Keyword match` (core five), plus `Capability use` and `Custom` (extensions). *(A 3-column `-with-methods` variant may be emitted as a human-readable reference only — never import it.)*

5. **Inherit the method from each criterion** — the method was set in Stage 1 and should carry through to every test case for that criterion. If a criterion's method doesn't fit a specific test case (e.g., one particular case needs exact-keyword verification while the rest use semantic match), override per-case rather than rewriting the criterion. Refresher table:

| Criterion style | Method | Why |
|---|---|---|
| Factual with known answer | Compare meaning | Semantic equivalence |
| Open-ended quality | General quality | LLM judge |
| Must-include terms (URL, email) | Keyword match | Exact presence |
| Agent should refuse | Compare meaning | Refusal matches expected |
| Domain-specific criteria (compliance, tone, policy) | Custom | Define your own rubric and pass/fail labels |

6. **Highlight the value:** "You now have [X] test cases across [Y] eval sets from your workbook registry. Compare that to the 5–10 happy-path prompts most customers start with. These include adversarial attacks, hallucination traps, robustness tests, and edge cases your users will encounter in production."

### Output

Display a summary table of test cases per eval set.

**The customer payoff:** *"You now have a test suite that imports directly into Copilot Studio, plus the .docx report your PM can sign off on, plus the Pillar 3 and Pillar 5 starter artifacts you'll keep for ongoing operations. That's the eval kit a new team member would need to evaluate this agent — questions, expected responses, methods, re-run protocol, comparison template."*

**Maturity callout — Pillar 2 / playbook Steps 2, 3, 5 (L100 Initial → L300 Systematic):** Generate advances Pillar 2 from "no established eval set" to versioned **capability** eval sets and separate **trust & safety** eval sets, coverage mapped to risk and value, each tagged `gate-only | regression | exploratory` for the regression suite (Step 8). Pillar 4 advances in Interpret. Pillars 3 and 5 reach L200 Defined via the `rerun-protocol-<agent>-<date>.docx` and `baseline-comparison-<agent>-<date>.xlsx` starter artifacts generated at session close — surface these to the customer when delivering them.

### Interactive Dashboard Checkpoint

Before generating final CSV and report files, launch the test cases dashboard for review:

1. Write the test cases to `stage-2-data.json`. **Methods and governance metadata live at the eval-set (`test_set`) level**, inherited from the workbook registry.

   ```json
   {
     "agent_name": "...",
     "test_sets": [
       {
         "eval_set_id": "CAP-ACC-001",
         "display_name": "Policy answer correctness",
         "set_type": "capability",
         "capability_dimension": "Accuracy / correctness",
         "methods": ["Compare meaning", "Keyword match"],
         "gate_type": "Hard floor + soft target",
         "target_pass_rate": "Launch floor 90%; regression/direction after baseline",
         "run_cadence": "Weekly",
         "cases": [
           {
             "id": 1,
             "question": "...",
             "expected_responses": {
               "Compare meaning": "Canonical answer, with [VERIFY: factual content to check] markers",
               "Keyword match": "PTO, Time Off Policy, accrual"
             },
             "custom_rubric": ""
           }
         ]
       },
       {
         "eval_set_id": "IF-CLARIFY-001",
         "display_name": "Instruction: ask a clarifying question when the request is ambiguous",
         "set_type": "instruction_following",
         "source_instruction": "Ask a clarifying question when the request is ambiguous.",
         "also_covers": "Relevancy",
         "methods": ["General quality"],
         "gate_type": "Soft target",
         "target_pass_rate": "Launch floor 85%; regression/direction after baseline",
         "run_cadence": "Per-change",
         "cases": [
           {
             "id": 2,
             "question": "How much leave do I have left?",
             "case_role": "positive trigger — office and tenure are both missing",
             "expected_responses": {},
             "custom_rubric": ""
           },
           {
             "id": 3,
             "question": "How much annual leave does a London employee with 3 years of service get per year?",
             "case_role": "negative control — nothing is missing, so the agent should answer, not ask",
             "expected_responses": {},
             "custom_rubric": ""
           }
         ]
       }
     ]
   }
   ```

   Key requirements:
   - Group test cases by workbook eval set, with cases nested directly under each set.
   - Each test set carries `set_type`: `capability`, `trust_safety`, or `instruction_following`. Instruction-following sets also carry `source_instruction` (the instruction **quoted verbatim**) and optional `also_covers`; their cases carry `case_role` naming positive trigger vs negative control. Every instruction-following set needs at least one of each.
   - If the customer declined to supply instructions, include `"instruction_following_skipped": true` at the top level so the dashboard and manifest can surface the gap.
   - Each test set carries a `methods: []` array — **the methods for this eval set's CSV**. Choose one method when one fits; choose multiple only when the eval set genuinely needs them. Default to one method.
   - Each test set carries workbook governance metadata (`gate_type`, `target_pass_rate`, `target_rationale`, `run_cadence`, owner/source/grader notes where available).
   - Each case has `expected_responses: { method → value }` — one entry per method in the eval set's `methods` array that needs a per-case reference (`Compare meaning`, `Text similarity`, `Exact match`, `Keyword match`). Methods that grade against a set-level rubric (`General quality`, `Capability use`, `Custom`) do NOT need entries.
   - Wrap AI-generated factual content in `[VERIFY: ...]` markers inside the `Compare meaning` / `Text similarity` entries so the dashboard highlights them for review.
   - **`Custom` method in the eval set**: also write a `custom_rubric` field on each set or case — a short LLM-judge rubric drafted from the eval-set purpose and expected behavior ("Rate the response Pass / Fail. Pass = …. Fail = …. Output PASS or FAIL with a one-sentence reason."). The dashboard shows this as an editable textarea. Don't leave Custom sets without a rubric.
   - **`Keyword match` method**: the per-case `expected_responses["Keyword match"]` value is a **comma-separated keyword list** (not a reference answer). The dashboard renders this as a "Keywords" column.
2. Launch the dashboard (resolve `<skill-dir>` from the skill context):
   ```bash
   python "<skill-dir>/dashboard/serve.py" --stage generate --serve --data stage-2-data.json
   ```
3. The user reviews the **Eval Sets Overview** at the top, then walks the stacked eval-set sections. Per eval set: edits the **Test Methods to Use** chips (set-level), checks gate/target/cadence metadata, edits Custom rubric callouts if Custom is used, edits per-method columns in the cases table, checks VERIFY-highlighted factual content, and adds/removes test cases.
4. When the user confirms, **parse the feedback from the bash stdout** between the `===EVAL_GUIDE_FEEDBACK_BEGIN===` / `===EVAL_GUIDE_FEEDBACK_END===` markers. **Apply every edit it contains, faithfully and without question.** The customer's choices are final — do NOT re-litigate, do NOT suggest reverting, do NOT ask for confirmation again, do NOT partially apply. (`generate-feedback.json` is also on disk as a backup, but stdout is the primary channel.)

   This applies to ALL edit types:
   - [VERIFY] span corrections (the customer fact-checked your draft against their real knowledge sources — their version wins). **At export time (CSV + .docx), strip every remaining `[VERIFY: …]` wrapper:** `[VERIFY: <content>]` → `<content>`. By the time the customer has confirmed, every span is either edited (already clean) or accepted (marker is now noise).
   - Question edits
   - Per-method per-case expected-response edits — keyed by method: `test_sets[i].cases[k].expected_responses["Compare meaning"]`, `test_sets[i].cases[k].expected_responses["Keyword match"]`, etc. Each method's value updates that method's column for that case.
   - Custom-method rubric edits (`test_sets[i].custom_rubric` or `test_sets[i].cases[k].custom_rubric`) — the customer's refined rubric is final; use it as the LLM judge prompt verbatim.
   - Eval-set-level method additions / removals (`test_sets[i].methods`) — adding/removing a method changes which columns and rubric blocks render for that eval set.
   - Test case additions and deletions.
   - General Comments box content.

   **Then narrate the edits back so the customer sees their changes were captured** — count [VERIFY] corrections, count test case additions/deletions, list significant edits, restate updated total case count. Example: *"Got it — 8 [VERIFY] corrections captured, 2 new cases for CAP-ACC-001, total now 56 cases across 7 eval sets."* Don't just say "applied." The narration confirms you parsed correctly; it is NOT an invitation to re-decide.

   If changes requested instead of confirmed, regenerate and re-launch.
5. **After confirmation, automatically generate ALL FIVE deliverables (A through E) — do not wait for the user to ask, do not ask "should I generate the docx now?", do not generate them in stages.** The CSVs, the test-case `.docx` report, the eval-setup-guide `.docx`, the rerun-protocol `.docx`, and the baseline-comparison `.xlsx` are one delivery, produced together. The customer should see the artifact list in chat ("five files generated") and find the files on disk before they say anything more.

**A. CSV files** — One CSV per eval set: `eval-<set-type>-<set-slug>-<date>.csv`. **Exactly two columns**:

   ```csv
   "Question","Expected response"
   ```

   **No Testing method column.** Copilot Studio's Evaluation tab requires the customer to **set the testing method manually per row in the UI** after import — it is not pre-encoded in the CSV. The companion `eval-setup-guide-<agent>-<date>.docx` (deliverable E below) walks the customer through that manual step in detail.

   **Row generation rule.** One row per active case per eval set (no case × method explosion). Per row:
   - `Question` = the case's question.
   - `Expected response` = whichever of the case's `expected_responses` is most informational, picked by this priority order against the eval set's method set:
     1. `Compare meaning` → `case.expected_responses["Compare meaning"]`.
     2. `Text similarity` → `case.expected_responses["Text similarity"]`.
     3. `Exact match` → `case.expected_responses["Exact match"]`.
     4. `Keyword match` → `case.expected_responses["Keyword match"]` (comma-separated keyword list).
     5. None of the above (signal only has reference-free methods like `General quality` / `Custom` / `Capability use`) → leave the cell empty.

   **Strip every `[VERIFY: …]` marker from the cell value before writing the row.** Replace `[VERIFY: <content>]` → `<content>`. The markers exist only as a review aid in the dashboard — by the time the customer has clicked Approve, every span has either been confirmed or edited. The CSV is the eval set the customer is importing into Copilot Studio; it must contain clean expected responses with no review-tooling syntax. Apply the regex `\[VERIFY:\s*([^\]]*)\]` → `$1` (or equivalent) to every Expected response cell before emitting the row.

   The customer can still edit any cell in CPS or in the CSV before import — for example, switching a row from canonical-answer to keyword-list when they decide that row should use `Keyword match`. The eval-setup-guide.docx makes this explicit.

   An eval set with 12 cases produces exactly 12 rows. (No multiplication by methods.)

   Tell the customer: "One CSV per eval set — two columns: Question and Expected response. Import each into Copilot Studio's Evaluation tab. Then in the CPS UI, set the **Testing method** for every row — this is a manual step. The eval-setup-guide.docx walks you through which method to pick per eval set and what threshold or regression rule to use."

**B. .docx report** — Generate a customer-ready report using the `/docx` skill. The report must be:
- **Concise** — no filler, no walls of text. Tables over paragraphs.
- **Presentable** — professional formatting with color-coded headers, clean tables, visual hierarchy
- **Self-contained** — a customer who wasn't in the conversation can read it and understand the eval plan + test cases

Report structure:
1. Agent Vision summary (from Stage 0) — 5-6 lines max
2. Workbook registry summary — eval sets grouped by Capability, Trust & Safety, and Agent-specific instruction-following, with Step 4 governance and Step 8 cadence
3. Test cases organized by eval set, with set-level target/gate/regression metadata. For instruction-following sets, print the verbatim `source_instruction` above the cases and label each case positive trigger or negative control. Close the category with the instructions dropped as untestable and why — or, if none were supplied, state the gap: *"No instruction-following sets — the agent's instruction block wasn't provided."*
4. For each test case: Question, Expected Response, and suggested test method. **Strip `[VERIFY: …]` markers** the same way as in the CSV — `[VERIFY: <content>]` → `<content>`. The dashboard's review markers don't belong in the customer-facing report.
5. Summary table: eval set, category, test case count, methods
6. "What these tests catch" callout — 3-4 bullet points on what the customer would have missed
7. Next steps — what to do with these files. **Always include a pointer line:** *"You're also receiving three companion artifacts (generated below) — `eval-setup-guide-<agent>-<date>.docx` (step-by-step Copilot Studio setup), `rerun-protocol-<agent>-<date>.docx` (Pillar 3 L200), and `baseline-comparison-<agent>-<date>.xlsx` (Pillar 5 L200). They walk you through how to set up the run today and advance Pillars 3 and 5 from L100 Initial to L200 Defined."*
8. Maturity snapshot — before/after table showing where the agent stands after this session:

   | Pillar | Baseline | After this session | Next-session target |
   |---|---|---|---|
   | 1 — Define what "good" means | L100 Initial | L300 Systematic ✓ | — |
   | 2 — Build your eval sets | L100 Initial | L300 Systematic ✓ | — |
   | 3 — Run evals across the lifecycle | L100 Initial | L200 Defined ✓ (via `rerun-protocol-<agent>-<date>.docx`) | L300 Systematic |
   | 4 — Improve and iterate | L100 Initial | L100 Initial | L300 Systematic (Stage 4) |
   | 5 — Handle changes with confidence | L100 Initial | L200 Defined ✓ (via `baseline-comparison-<agent>-<date>.xlsx`) | L300 Systematic |

**C. Pillar 3 starter — `rerun-protocol-<agent>-<date>.docx`** — Generate using the `/docx` skill, sourcing structure and content from `skills/eval-guide/rerun-protocol.md`. This is the customer's takeaway reference for Pillar 3 L200 Defined: when to re-run evals, what scope to run, how to log the result. The docx is portable, printable, and shareable with the team.

   Render the markdown sections as docx sections with the same headings (Purpose, Prerequisites, When to re-run, Run order rule, Logging discipline, Interpreting re-run results, You've reached L200 Defined when…, Path to L300 Systematic, References). Format the trigger table as a styled docx table, color-code the priority column, and put the "You've reached L200 Defined when…" exit criteria in a callout box.

**D. Pillar 5 starter — `baseline-comparison-<agent>-<date>.xlsx`** — Generate using the `/xlsx` skill, sourcing structure and content from `skills/eval-guide/baseline-comparison-template.md`. This is the customer's fill-in workbook for Pillar 5 L200 Defined: a structured template they fill in each time they compare two eval runs.

   Workbook structure (auto-size columns; freeze header rows; protect instruction sheets):

   | Sheet | Contents |
   |---|---|
   | **Instructions** | Purpose, when to use, prerequisites. Read-first sheet — protected. |
   | **Comparison** | 5-metric comparison table with empty Run 1 / Run 2 / Delta cells (Overall, Capability eval-set pass rate, Trust & Safety gate status, Regression eval-set pass rate, Hard gate failures). Above the table: editable cells for Run 1 name/version, Run 2 name/version, Eval set version, Change description. |
   | **Case-level delta** | 4-row bucket table (Pass-Pass / Fail-Pass / Pass-Fail / Fail-Fail) with empty Count and Notable cases columns. Conditional formatting highlights Pass-Fail row in red. |
   | **Decision rules** | Variance rules, ship/hold logic. Read-only reference sheet. |
   | **Capability vs. regression** | Cheat sheet on the two run types, when to use each. Read-only reference sheet. |

**E. Eval setup guide — `eval-setup-guide-<agent>-<date>.docx`** — **Always generate this alongside the CSVs (A). It is not optional and not on-request.** Without it, the customer is staring at CSVs with no instructions for the manual method-assignment step in CPS. Generate using the `/docx` skill, sourcing structure and content from `skills/eval-guide/eval-setup-guide.md`. This is the customer's step-by-step walkthrough for setting up and running the CSVs in Copilot Studio's Evaluate tab — the operational companion to the eval set.

   Render the markdown sections as docx sections with the same headings (What you should have before you start, Step 1–8, Per-method setup table, How to choose a threshold, Common setup issues, You've finished setup successfully when…, Related artifacts, References). Format the per-method setup section as styled docx tables; pull the eval-set method decision tree into a callout box; preserve the troubleshooting symptom/cause/fix table verbatim.

Tell the customer: "Five artifacts: the CSVs go straight into Copilot Studio, the test case .docx is for sharing, the new `eval-setup-guide-<agent>-<date>.docx` walks you through the Evaluate tab step by step (open it the first time you set up the run), and `rerun-protocol-<agent>-<date>.docx` + `baseline-comparison-<agent>-<date>.xlsx` are your Pillar 3 and Pillar 5 starter kits — keep them with your eval set."

---

