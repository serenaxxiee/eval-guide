# Eval generator deliverables and review checkpoints

Open this companion before writing CSV files, `.docx` manifest content, or final review reminders. It contains the moved output-format and review-checkpoint reference material from `SKILL.md` verbatim.

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

