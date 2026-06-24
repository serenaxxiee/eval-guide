---
name: eval-result-interpreter
description: Analyzes Copilot Studio evaluation results using Practical Guidance on Agent Evaluation's 10-step playbook (Steps 6, 7, and 9) plus Microsoft's triage diagnostics. Returns a gate-based SHIP / ITERATE / BLOCK verdict with root cause classification, remediation, and pattern analysis.
---

## Purpose

This skill takes eval results — a Copilot Studio evaluation CSV file, a pasted summary, or plain-English description of results — and produces a structured triage report. It is the standalone **Interpret** skill in the operational workflow: plan → generate → run → **interpret**. In the 10-step playbook, it reads the **baseline (Step 6)**, drives **diagnosis (Step 7)**, and designs the **Step 9 optimization loop**. The output tells you whether to ship, what broke, why it broke, and what to fix first.

This skill is grounded in **Practical Guidance on Agent Evaluation: a 10-step playbook**. It uses Step 6 to read baseline results with agent version and timestamp, Step 7 to classify failures into eval-setup vs agent-quality problems, and Step 9 to define the production feedback loop. MS Learn evaluation resources remain useful supporting references, but the 10-step playbook is the canonical methodology.

**Knowledge source:** This skill's analysis framework is grounded in the 10-step playbook plus Microsoft's Triage & Improvement Playbook diagnostics — SHIP/ITERATE/BLOCK gate interpretation, failure verification, remediation mapping, and pattern analysis.

### When to use this skill vs. eval-triage-and-improvement

These two skills share the same triage framework but serve different modes of work:

| Use **eval-result-interpreter** when… | Use **eval-triage-and-improvement** when… |
|---|---|
| You have a CSV file or concrete results and want a **one-shot structured report** | You want **interactive guidance** walking through diagnosis step by step |
| This is your **first look** at results — you need a verdict and top actions fast | You are in an **ongoing improvement loop** — fixing, re-running, and re-triaging |
| You want a **customer-deliverable artifact** (the .docx triage report) | You need **detailed remediation help** for specific eval-set failures (e.g., "wrong tool fires — now what?") |
| The eval run is relatively straightforward (<20 failures) | You have **many failures** (15+) and need help prioritizing which to investigate |
| You need the **activity map / result comparison** tool recommendations inline | You need the playbook worked examples and deeper diagnostic walkthroughs |

**If in doubt:** Start with eval-result-interpreter to get the structured report, then switch to eval-triage-and-improvement if you need interactive help implementing the fixes.

## Instructions

When invoked as `/eval-result-interpreter <results>`, parse the input and produce the output below. Accept any of these input formats:

**Format 1 — Copilot Studio CSV file** (primary)

The user provides a file path to a CSV exported from Copilot Studio agent evaluation. The CSV has these columns:

| Column | Description |
|---|---|
| `question` | The test case input sent to the agent |
| `expectedResponse` | The expected answer (may be empty for General Quality tests) |
| `actualResponse` | The agent's full response |
| `testMethodType_1` | The test method used (e.g., GeneralQuality, CompareMeaning, KeywordMatch, ToolUse, ExactMatch, Custom) |
| `result_1` | Pass or Fail |
| `passingScore_1` | The threshold score (may be empty) |
| `explanation_1` | The grader's reasoning for the verdict |

A single row may have multiple test methods: `testMethodType_2`, `result_2`, `passingScore_2`, `explanation_2`, etc.

When the user provides a file path, read the CSV and parse it. Count Pass/Fail totals and per test method.

**Format 2 — Plain-text summary**

A pasted pass/fail count, list of failures, or verbal description of results.

**Format 3 — Manifest / methodology metadata** (preferred, improves accuracy)

Prefer the manifest metadata produced by the generator — the companion `.docx` report and dashboard `stage-N-data.json` — over inferring from CSV filenames or question text. Use it to map each row or set to `set_type` (`capability` or `trust_safety`), category/dimension, testing method, gate type, pass-rate target, regression class, human-review flag, and source/ground-truth provenance. Say: "Using your manifest for set metadata and gate interpretation."

If no workbook/manifest is present, fall back to CSV filenames, then question text. State what was inferred and mark gate status as owner-review-needed.

Work with whatever detail is available. If input is sparse, state what you assumed. Do not ask for more — give the best triage possible with what is provided.

---

### Output structure

**0. Pre-triage infrastructure check** (per the Triage Playbook)

Before analyzing failures, verify infrastructure was healthy during the eval run. If any of these were unhealthy, mark affected cases as infrastructure-blocked, not agent-failed:
- Were all knowledge sources accessible and fully indexed?
- Did any API backends return errors, timeouts, or rate-limiting?
- Were authentication tokens valid throughout the run?
- Did the eval environment match the intended configuration?

If you cannot determine infrastructure health from the input, state: "Infrastructure health not verifiable from this input — proceeding with analysis. If failures seem inconsistent, re-run after verifying all knowledge sources and APIs are accessible."

**1. Baseline score summary (Step 6)**

Parse the results and produce:

| Metric | Value |
|---|---|
| Total test cases | X |
| Passed | X |
| Failed | X |
| Aggregate pass rate | X% |
| Test methods used | GeneralQuality, CompareMeaning, etc. |
| Baseline timestamp / agent version | From input if available; otherwise "not provided" |

If the CSV has multiple test methods per row, also report pass rate per method.

Then report eval sets separately by **capability** and **trust & safety**. Prefer manifest metadata for `set_type`, `category`, testing method, gate type, pass-rate target, regression class, and provenance. Use this table for each set:

| Eval set | Set type | Category / dimension | Testing method | Regression class | Target | Actual | Gate type | Gate status |
|---|---|---|---|---|---:|---:|---|---|
| X | capability / trust_safety | X | X | regression / gate-only / both | X% | X% | hard / soft | PASS / MISS |

Keep capability and trust & safety reporting separate. Trust & safety categories are `guardrails`, `out_of_scope`, `sensitive_data`, `prompt_injection`, and `compliance`. Hallucination is a **faithfulness/groundedness capability** failure, not a trust & safety failure.

**2. Verdict — gate-based SHIP / ITERATE / BLOCK decision (D5)**

Drive the verdict from the **per-set gates in the manifest**, not from aggregate pass rate. For every eval set, read its pass-rate target and gate type from the manifest when available:

- **Hard gate** — must pass before deploy. Any failed hard gate means the agent cannot SHIP.
- **Soft target** — tracked and remediated, but not blocking by itself.

Apply this gate-based decision rule:

```
ANY hard gate missed?
    YES -> cannot SHIP.
           Trust & safety hard-gate miss -> usually BLOCK.
           Deployment-critical capability hard-gate miss -> BLOCK or ITERATE based on severity and owner risk tolerance and gate policy.
           Other hard-gate miss -> ITERATE until fixed.
    NO  ->
        ANY soft target missed?
            YES -> ITERATE: track the gap and fix in priority order, but it is not blocking by itself.
            NO  -> SHIP, assuming human review agrees coverage is sufficient.
```

Report each set's actual pass rate vs target and hard/soft gate status. Make explicit when aggregate pass rate is misleading: a high aggregate pass rate does **not** earn SHIP if a hard trust & safety gate failed.

Use **risk tier** (agent-level: reach, criticality of error, autonomy/blast radius, regulatory exposure, data sensitivity) to interpret target strictness and severity. Use the workbook registry's eval-set category, gate type, target, intended use, cadence, and grader-validation notes as the source of truth for gate decisions.

If no workbook/manifest is present, infer set grouping, gate type, and targets from CSV filenames or question text only as a fallback. State: "No workbook or manifest provided — gate status inferred and should be reviewed by the owner."

State the verdict prominently:
- **"Verdict: SHIP."** — All hard gates pass and soft targets are acceptable or explicitly accepted by the owner.
- **"Verdict: ITERATE."** — No blocking hard-gate failure, but one or more soft targets or non-blocking hard gates require fixes before confidence is high.
- **"Verdict: BLOCK."** — A hard trust & safety gate or other deployment-critical hard gate failed.

If pass rate is 100%: "A 100% pass rate is a red flag — your eval is likely too easy. Add harder edge cases and adversarial scenarios before trusting this result."

**3. Failure triage — per the Triage Playbook's Layer 2**

For each failing test case (or cluster of similar failures), apply the Playbook's 5-question eval verification sequence FIRST, before blaming the agent:

| # | Diagnostic Question | If YES -> root cause |
|---|---|---|
| 1 | Is the agent's actual response acceptable (would a real user be satisfied)? | **Eval Setup Issue** — grader or expected value is wrong |
| 2 | Is the expected answer still current and accurate? | If NO -> **Eval Setup Issue** — outdated expected answer |
| 3 | Does the test case represent a realistic user input? | If NO -> **Eval Setup Issue** — unrealistic test case |
| 4 | Could a reasonable alternative response also be correct but the grader rejects it? | **Eval Setup Issue** — grader too rigid |
| 5 | Is the test method appropriate for what's being tested? | If NO -> **Eval Setup Issue** — wrong method |

Every failing case must land in exactly one of the 10-step playbook's two Step 7 root buckets:

- **Eval-setup problem** — the response is actually acceptable; the eval flagged it wrongly. Action: fix the eval.
- **Agent-quality problem** — the eval correctly caught a real issue. Action: log the pattern, define a fix, and track it.

Keep the finer Triage Playbook taxonomy, but explicitly map it onto those two buckets:

| Fine taxonomy | Step 7 bucket | Meaning | Action |
|---|---|---|---|
| **Eval Setup Issue** | **eval-setup problem** | The test case, expected answer, rubric, grader, or test method is wrong. The agent may be performing correctly. Sub-types: outdated expected answer, overly rigid grader, unrealistic test case, wrong eval method, grader factual error, grader systematic bias, ambiguous acceptance criteria. | Update the eval case, rubric, expected response, or method; re-run only affected cases unless the rubric changed broadly. |
| **Agent Configuration Issue** | **agent-quality problem** | The agent genuinely produced a bad response. | Fix system prompt, knowledge sources, tool config, topic routing, orchestration, or action behavior. |
| **Platform Limitation** | **agent-quality problem** | The failure is real but caused by underlying platform behavior you cannot fix through normal configuration. Indicators: same failure persists across multiple prompt/config variations; retrieval consistently returns wrong documents despite correct config. | Document the limitation, design a workaround, and decide whether the hard/soft gate can be met. |

Maintain a **failure-pattern log** with case ID, eval set, set type, category/dimension, gate type, Step 7 bucket, fine taxonomy, pattern, owner, fix location, and re-run target. Group failures that share a root cause. For example: "Cases 3, 5, and 7 all fail with 'Question not answered' — this is one agent-quality pattern (missing knowledge source or scope gap), not three independent problems."

**3b. Platform diagnostic tools (recommend when applicable)**

Copilot Studio provides built-in tools that accelerate triage. Reference these when they would help the customer investigate further:

| Tool | What it does | When to recommend |
|---|---|---|
| **Activity map** | Shows the agent's decision process for a test case — which topics triggered, which knowledge sources were retrieved, which actions were called. Available by clicking into any test case result in the UI. | Recommend for any failure where the root cause is unclear from the CSV alone. Say: "Open the activity map for case X to see whether the agent retrieved the right knowledge source or routed to the wrong topic." |
| **Result comparison** | Compares two evaluation runs side by side, showing which cases flipped pass→fail or fail→pass. Available when you have multiple runs of the same test set. | Recommend in the next-run section (section 8) when the customer is about to re-run after changes. Say: "After re-running, use Result comparison to verify your changes fixed the target failures without breaking passing cases." |
| **Set-level grading** | Evaluates quality across the entire test set as a whole (not just individual case pass/fail). Provides an aggregate quality assessment. | Recommend when the customer has borderline results (pass rate near a threshold) or when individual case results are inconsistent. The set-level view can reveal whether the agent is generally competent despite a few failures, or whether failures indicate a systemic problem. |

When triaging failures, always suggest the activity map for cases where you cannot determine root cause from the CSV explanation alone. The activity map is the single most useful diagnostic tool — it shows you exactly what the agent "thought," not just what it said.

**Supplementary signal: User reactions (thumbs up/down)**

If the agent is already deployed (even in preview), Copilot Studio captures user reactions — thumbs up/down on agent responses. These are not part of the eval CSV, but they complement eval results:

- **Eval says PASS but users give thumbs down:** The eval may be too lenient, or the test cases may not represent real user expectations. Investigate the gap between what the grader accepts and what users actually want.
- **Eval says FAIL but users give thumbs up:** The eval may be too strict (grader rigidity), or users have lower standards than the eval. Revisit the expected responses for these scenarios.
- **Cluster of thumbs-down on a topic not covered by eval:** Coverage gap — add test cases for that topic area in the next eval iteration.

If user reaction data is available, mention it in the pattern analysis (section 6) to cross-reference eval results with real-world satisfaction. Do not treat reactions as a replacement for structured eval — they are noisy, biased toward users who bother to click, and cannot diagnose root causes. They are a signal, not a verdict.

**4. Explanation analysis**

**4a. General Quality scoring criteria**

When the test method is GeneralQuality, Copilot Studio scores the response on **4 distinct criteria**. A low General Quality score means one or more of these failed — the customer needs to know WHICH one to fix the right thing:

| Criterion | What it evaluates | Low score means | Remediation direction |
|---|---|---|---|
| **Relevance** | Does the response address the user’s question? | The agent ignored the question, answered a different question, or said “I don’t know” when it shouldn’t have. | Check knowledge source coverage — is the topic in scope? Check topic routing — is the right topic triggering? Open the activity map to see what the agent retrieved. |
| **Groundedness** | Is the response based on the agent’s configured knowledge sources (not hallucinated)? | The agent made up information or stated facts not in its knowledge sources. This is the hallucination detector. | Review which knowledge sources were retrieved (activity map). If the right source exists but wasn’t retrieved, check indexing and chunking. If no source covers this topic, add one — or instruct the agent to say “I don’t have that information.” |
| **Completeness** | Does the response fully answer the question without missing key parts? | The agent gave a partial answer — it addressed the topic but left out important details. | Check whether the knowledge source contains the full answer. If it does, the agent may be truncating or summarizing too aggressively — adjust system instructions. If the source is also incomplete, update the source. |
| **Abstention** | Does the agent appropriately decline when it should? (Not over-answering, not under-answering.) | The agent either answered when it should have declined (e.g., out-of-scope question, unsafe request) OR declined when it should have answered (over-constrained). | Review system instructions for scope boundaries. Low abstention + low relevance = agent answering everything poorly. Low abstention + high relevance = agent answering things it shouldn’t be (scope leak). |

**How the 4 criteria interact:** A passing General Quality score means all 4 criteria passed. A failing score means at least one failed — check the `explanation` field to determine which. The most common failure pattern is Relevance failing alone (knowledge gap), followed by Groundedness failing alone (hallucination). When both Relevance and Groundedness fail together, the agent is likely retrieving the wrong knowledge source entirely.

**When NOT to rely on General Quality alone:** General Quality checks response quality holistically but cannot verify specific factual values, check tool invocation correctness, or validate structured output formats. Use it alongside targeted methods (CompareMeaning for factual accuracy, ToolUse for action verification, KeywordMatch for required terms).

**4b. Explanation pattern mapping**

Parse the `explanation` fields from the CSV. Copilot Studio’s General Quality explanations use these patterns — map each to the criteria above and the Playbook’s diagnostic questions:


| Explanation pattern | Quality signal | Playbook diagnostic area |
|---|---|---|
| "Seems relevant; Seems complete; Based on knowledge sources" | All passing | — |
| "Question not answered; Further checks skipped because relevance failed" | Relevance failure | Diagnostics 2.1-2.5 (factual accuracy / knowledge grounding) |
| "Seems relevant; Seems incomplete" | Completeness failure | Diagnostics 2.15-2.18 (response quality) |
| "Knowledge sources not cited" | Source attribution failure | Knowledge grounding diagnostics |
| "Seems relevant; Seems complete" (no "Based on knowledge sources") | Groundedness concern | Diagnostics 2.4-2.5 (hallucination risk) |

For each explanation pattern found in the failures, name the diagnostic area and suggest the specific Playbook question to investigate.

**4c. Conversation (multi-turn) result interpretation**

When interpreting results from conversation test sets (multi-turn evaluations), the failure patterns differ from single-response tests. Apply these additional diagnostic lenses:

**Turn-level diagnosis:** A conversation test case fails as a whole, but the root cause is usually in a specific turn. Read the agent's responses turn by turn to locate the first turn where quality degrades. Common patterns:

| Pattern | What it means | Fix direction |
|---|---|---|
| Turn 1 passes, Turn 3+ fails | **Context loss** — the agent forgot earlier context. Check whether the agent's orchestration maintains conversation state. | Review system instructions for context retention. Check if the topic resets mid-conversation (classic orchestration) or if the LLM context window is being exceeded (generative orchestration). |
| All turns fail on same criterion | **Systemic issue** — not a multi-turn problem. The agent has a baseline quality problem regardless of turn count. | Treat as a single-response failure and diagnose with the standard framework above. |
| Turn 2 fails (clarification turn) | **Clarification handling** — the agent didn't ask the right follow-up or misinterpreted the user's clarification. | Check system instructions for clarification behavior. Verify the agent has instructions for handling ambiguous or incomplete user inputs. |
| Last turn fails (resolution turn) | **Incomplete task completion** — the agent understood the request across turns but failed to deliver the final answer or action. | Check whether the agent has the right knowledge sources or tool connections to complete the end-to-end task. The diagnosis tools are correct but the "last mile" fails. |
| Agent repeats itself across turns | **State loop** — the agent is stuck. Often caused by topic routing that keeps re-triggering the same topic. | Open the activity map for this conversation to see if the agent is cycling through the same topic or action repeatedly. |

**Available methods are limited:** Conversation tests only support General Quality, Keyword Match, Capability Use (Capabilities match), and Custom. If you see failures that would benefit from Compare Meaning or Exact Match analysis (e.g., the agent gave the right answer but phrased differently), note this limitation and recommend the customer also create a complementary single-response test set for those specific scenarios.

**Critical turn identification:** When reporting failures, identify and call out the **critical turn** — the specific turn where the conversation went wrong. Downstream turns often fail as a consequence of an earlier turn's failure, not independently. Fixing the critical turn may resolve multiple downstream failures in one change.

**4d. Set-level grading interpretation**

Copilot Studio’s set-level grading evaluates the test set as a whole — not just aggregating individual pass/fail counts, but assessing overall agent quality across the full set. When the customer has set-level results, interpret them alongside case-level results using this framework:

**When set-level and case-level results agree:** The straightforward case. A high set-level grade with a high case-level pass rate confirms the agent is performing well. A low set-level grade with many case-level failures confirms systemic problems. Use the standard triage framework above.

**When set-level and case-level results diverge — this is where interpretation matters:**

| Divergence | What it means | Action |
|---|---|---|
| **High case-level pass rate, low set-level grade** | Individual responses pass their graders, but the agent’s overall behavior has quality gaps — inconsistent tone across responses, uneven depth, or passing “by the letter” but not “in spirit.” | Review a sample of passing cases manually. The graders may be too lenient (accepting mediocre responses), or the set-level evaluation is catching patterns invisible at the case level (e.g., the agent gives correct but robotic answers). Consider tightening individual graders. |
| **Low case-level pass rate, high set-level grade** | Many individual cases fail their specific graders, but the agent’s overall behavior is competent. Common when graders are overly strict (e.g., requiring exact phrasing when the agent’s paraphrases are fine). | This is a strong signal that **eval setup issues** dominate. Audit failing cases using the 5-question eval verification sequence (section 3). Likely action: loosen graders or update expected responses, not fix the agent. |
| **Set-level grade changes across runs but case-level results are stable** | The holistic quality assessment is picking up something the individual graders miss — possibly tone drift, increasing verbosity, or subtle quality shifts. | Compare actual responses between runs qualitatively. The set-level grader may be detecting stylistic degradation that case-level pass/fail cannot capture. |

**How to use set-level grades in the verdict:** Set-level grading is supplementary — it does not override the SHIP/ITERATE/BLOCK verdict, which is based on manifest-defined hard gates and soft targets. However, a low set-level grade on an otherwise SHIP-ready result should trigger a human review checkpoint: "Gate status says SHIP, but set-level quality assessment is below expectations. Review a sample of passing responses before shipping."

**5. Top 3 actions — per the Triage Playbook's Layer 3 (Remediation Mapping)**

List exactly three actions in priority order. Each must follow the Playbook's remediation pattern: **change X -> re-run Y -> expect Z.**

Prioritize using the playbook's gate-first order:
1. Failed hard gates first, especially trust & safety (`guardrails`, `out_of_scope`, `sensitive_data`, `prompt_injection`, `compliance`)
2. Agent-quality patterns affecting high-value/high-risk capability sets
3. Missed soft targets, starting with the lowest-scoring eval set
4. Recurring failures or regressions (same case failing across runs)
5. Eval-setup problems that distort the verdict or hide true risk

Examples of required specificity:
- "**Change:** Add the product FAQ document to the agent's knowledge sources. **Re-run:** Cases 4 and 7 (both show 'Question not answered'). **Expect:** Relevance to pass for product-related queries."
- "**Change:** Add an escalation instruction to the system prompt: 'If you cannot resolve the request, offer to connect the user with a human agent.' **Re-run:** Case 3 ('speak to a representative'). **Expect:** Relevance to pass."
- "**Change:** Update the expected response in case 5 — it references an outdated process. **Re-run:** Case 5 only. **Expect:** Compare Meaning score to improve (this is an eval setup fix, not an agent fix)."

**6. Pattern analysis — per the Triage Playbook's Layer 4**

Check for these cross-signal patterns from the Playbook:

| Pattern | Likely indicates |
|---|---|
| All failures share "Question not answered" | Knowledge source gap or scope definition issue |
| Factual accuracy AND knowledge grounding both failing | Knowledge source issue (wrong docs retrieved or missing) |
| Accuracy passing but tone/quality failing | Right answer, poor delivery — style instruction needed |
| Trust & safety passing but capability accuracy failing | Agent may be over-constrained — review safety restrictions |
| All failures cluster in one question type | Systemic gap — fix the category, not individual cases |
| 80%+ failures are eval setup issues | Pause agent work — audit and fix the evals first |
| One signal improving, another degrading after a change | Instruction conflict (instruction budget problem) |

Also check for concentration: if most failures share a root cause type, call it out. Per the Playbook: "80%+ same root cause = systemic issue, fix the category."

**7. Interpretation rationale (teach the WHY)**

After presenting the triage, explain the reasoning so the customer can apply this framework independently next time. Cover these four points:

- **Why the verdict landed where it did:** Walk through the hard gates and soft targets with the actual per-set numbers. Example: "The guardrails trust & safety set is a hard gate and passed at 100% against a 95% target, so it did not BLOCK. The billing accuracy capability set is a soft target and passed at 72% against an 80% target, so the verdict is ITERATE — even though aggregate pass rate is 86%, the missed soft target still needs a fix."
- **Why failures were classified the way they were:** For each Step 7 bucket and fine taxonomy used, explain the reasoning chain. Example: "Cases 3 and 7 were classified as eval-setup problems / Eval Setup Issue because the agent's actual response is substantively correct — it answers the question accurately — but the grader rejected it due to phrasing differences. The expected response says 'Contact support at 1-800-555-0100' but the agent says 'You can reach our support team at 1-800-555-0100.' Same information, different words. This is a grader rigidity problem, not an agent-quality problem."
- **Why the top 3 actions are in that priority order:** Connect each action's priority to the triage framework. Example: "The knowledge source fix is #1 because it addresses 4 of 6 agent failures and they're all core business scenarios. The prompt tweak is #2 because it fixes 2 failures but they're capability scenarios, which rank lower in the Playbook's priority order. The eval fix is #3 because it doesn't improve the agent — it just corrects the measurement."
- **What this triage does NOT tell you:** Name the limits. Example: "This triage is based on a single eval run. It cannot detect non-determinism issues (run the eval 3 times to check for variance). It also cannot assess whether your test cases cover the right scenarios — a passing eval with poor coverage is worse than a failing eval with good coverage."

This section teaches the methodology so customers can eventually interpret results without the skill. Each bullet must reference the specific data from this eval run, not generic advice.

**8. Next-run recommendation**

End with one sentence naming exactly what to re-run after making changes. Per the Playbook's re-run targeting:

| What changed | What to re-run |
|---|---|
| Single test case (eval fix) | Only the affected test case |
| Agent config change | Affected test cases + spot-check one unrelated set |
| System prompt change | Full eval suite |
| Knowledge source update | All faithfulness/groundedness and factual-accuracy capability cases |
| Hard-gate fix | The failed hard-gate set + the Step 8 regression suite |

**Tip:** After re-running, use Copilot Studio's **Result comparison** feature to compare the new run against the previous one. It shows which cases flipped pass→fail or fail→pass, making it easy to verify your changes fixed the intended failures without introducing regressions.

**8b. Version comparison interpretation (when the customer provides two runs)**

If the customer provides results from two eval runs (before/after a change, or two agent configurations), produce a comparison analysis in addition to the standard triage above. Accept this as two CSV files, two pasted summaries, or a description like "Run 1 was 78%, Run 2 is 85%."

**Comparison table:**

| Metric | Run 1 (Before) | Run 2 (After) | Delta |
|---|---|---|---|
| Overall pass rate | X% | Y% | +/-Z% |
| Core business pass rate | X% | Y% | +/-Z% |
| Trust & safety pass rate | X% | Y% | +/-Z% |
| Capability pass rate | X% | Y% | +/-Z% |
| Hard gates missed | X | Y | +/-Z |
| Soft targets missed | X | Y | +/-Z |

**Case-level delta analysis:**

Categorize every test case into one of four buckets:

| Bucket | Meaning | Action |
|---|---|---|
| Pass-Pass (Stable) | Passed in both runs, no regression | None, but note these as the regression baseline |
| Fail-Pass (Fixed) | Failed before, passes now, the change worked | Verify the fix is genuine (not non-determinism). Run 2-3 more times to confirm stability |
| Pass-Fail (Regressed) | Passed before, fails now, the change broke something | **Highest priority.** Regressions are worse than pre-existing failures because they represent lost ground. Investigate immediately |
| Fail-Fail (Persistent) | Failed in both runs, the change did not help | Re-examine root cause. If the fix was supposed to address this case and did not, the diagnosis was wrong |

**Interpreting deltas:**
- **+/-5% overall variance between runs is normal** (LLM non-determinism). Do not celebrate or panic over small swings. Run the eval 3 times and take the median to distinguish signal from noise.
- **A case that flips between runs** (pass in one, fail in another, on the same agent version) is a **reliability problem**, not a quality problem. Flag it separately.
- **Regressions outnumbering fixes** after a change means the change had a net negative impact, consider reverting.
- **All fixes in one category, all regressions in another** = instruction conflict. The prompt change that fixed trust & safety responses may have over-constrained business responses. This is the most common pattern when system prompt edits have unintended side effects.

**Capability, trust & safety, and regression framing:** Help the customer understand what each eval run type is FOR:
- **Capability eval sets** measure task quality dimensions such as accuracy, faithfulness/groundedness, relevancy, style/tone, and reasoning/tool use. Hallucination belongs here as a faithfulness failure.
- **Trust & safety eval sets** measure refusal, scope, sensitive-data, prompt-injection, and compliance behavior. These are usually hard gates.
- **Regression sets** re-run previously passing or critical test cases after changes. Pass rates should be near target; any drop is a regression that must be investigated.
- **Gate-only sets** run at milestones such as pre-pilot, pre-production, or post-significant-change.

A healthy eval practice uses both capability and trust & safety sets, partitioned into Step 8 regression and gate-only suites. If the customer is only running one type, recommend adding the other.

**9. Production optimization-loop plan (Step 9)**

Close with a short forward-looking optimization-loop plan that bridges this eval to continuous post-deployment improvement:

1. **Collect signals:** thumbs-down (highest signal), escalations, manual overrides, support tickets, and qualitative feedback.
2. **Cluster:** group signals into recurring failure patterns and map each cluster to existing eval sets or coverage gaps.
3. **Decide fix location:** agent config (prompt/retrieval/tools/orchestration), rubric/eval setup, or new eval cases.
4. **Ship:** make the smallest safe change with an owner and version note.
5. **Re-evaluate:** run the affected cases plus the Step 8 regression suite, then update the failure-pattern log and regression suite with any new cases.

---

### Step 3 — Generate output file

After displaying the triage report in conversation, generate a formatted report:

**Eval Results Triage Report (.docx)**
Use the docx skill to create a formatted document containing:
- Title: "Eval Results Triage Report"
- Date and agent name (if known)
- Score summary table
- Verdict (SHIP/ITERATE/BLOCK) with gate explanation
- Per-set actual pass rate vs target and hard/soft gate status
- Capability and trust & safety results reported separately
- Failure triage details for each failing case, including Step 7 bucket and fine taxonomy
- Failure-pattern log
- Top 3 prioritized actions
- Pattern analysis
- Interpretation rationale (from section 7 — the WHY behind the verdict, classifications, and priorities)
- Human review checkpoints table (from Step 4)
- Next-run recommendation
- Production optimization-loop plan (Step 9)

---

### Step 4 — Human review checkpoints

After the output file and before the conversation ends, display a **Human Review Required** section. Eval interpretation is where bad assumptions become bad decisions — a wrong verdict can ship a broken agent or block a good one. These checkpoints flag where human judgment is essential.

**Human Review Required**

| # | Checkpoint | What to verify | Why it matters |
|---|---|---|---|
| 1 | **Verdict matches your business reality** | The thresholds that produced SHIP/ITERATE/BLOCK are defaults. Does the verdict align with what you'd actually be comfortable deploying? A "SHIP" at 86% may be unacceptable for a healthcare agent; an "ITERATE" at 78% may be fine for an internal FAQ bot. | Only your team knows your actual risk tolerance and gate policy. The verdict is a recommendation, not a decision. |
| 2 | **Eval setup issues are real, not excuses** | For every failure classified as "eval setup issue," read the agent's actual response yourself. Is it truly acceptable? Or is the AI giving the agent the benefit of the doubt? | Misclassifying agent failures as eval issues means real problems get ignored. The 20% estimate is a starting point, not a free pass. |
| 3 | **Root cause groupings make sense** | When failures are grouped ("Cases 3, 5, 7 share a root cause"), verify they actually stem from the same problem. Different symptoms can look similar from CSV data alone. | Wrong grouping means wrong fix means wasted iteration. One bad grouping can send you fixing the wrong thing for a full cycle. |
| 4 | **Top 3 actions are feasible and correctly prioritized** | Can you actually make the suggested changes? Is the priority order right for your timeline and constraints? A knowledge source fix may be suggested first but take 2 weeks; a prompt tweak may be faster and unblock you now. | The recommended priority is based on impact, but your team knows the effort and dependencies. |
| 5 | **100% pass rate is investigated, not celebrated** | If the result is 100%, do NOT ship without adding harder test cases. Check: Are expected responses too vague? Are test methods too lenient? Are you only testing the happy path? | A perfect score almost always means the eval is too easy, not that the agent is perfect. |
| 6 | **Remediation will not break passing scenarios** | Before making changes based on the top 3 actions, check whether those changes could affect currently-passing test cases. Prompt changes especially have ripple effects. | Fixing 3 failures while introducing 5 new ones is a net loss. Always re-run the full suite after changes. |

After the checkpoints, add:
- **Mandatory reminder:** "This triage report was AI-generated from your eval results. Before acting on the verdict or remediation actions, review the failing cases with your team — especially any classified as eval setup issues. The distinction between an agent problem and an eval problem requires human judgment."

---

### Data Retention Warning

Copilot Studio **deletes test run results after 89 days**. Always recommend that the user:
1. **Export the results CSV** immediately after each eval run (Test set → Export results)
2. **Store alongside the agent version** in SharePoint or a repo
3. If the report recommends re-running after fixes, export the current results *before* changes so before/after comparison is possible

Include this reminder at the end of every generated report.

---

### Behavior rules

- State the verdict FIRST, before any analysis.
- Prefer manifest metadata over inference for set type, category, testing method, gate type, target, regression class, and provenance.
- Drive SHIP / ITERATE / BLOCK from hard gates and soft targets, not aggregate pass rate.
- BLOCK immediately if any hard trust & safety gate fails. Trust & trust & safety failures are non-negotiable unless the owner explicitly reclassifies the gate in the manifest.
- Always check whether failures are eval-setup problems before blaming the agent. This is the most common mistake in eval interpretation.
- Every failure must be classified into exactly one Step 7 bucket: eval-setup problem or agent-quality problem. Preserve the fine taxonomy as a secondary label.
- If pass rate is 100%, treat it as a red flag and say so.
- If input is too sparse for a confident verdict, default to ITERATE and explain why.
- When you cannot determine if a failure is an agent-quality problem or eval-setup problem from the CSV alone, say so explicitly and tell the user to read the `actualResponse` for that row.
- Per the Playbook's non-determinism guidance: if the user mentions running evals multiple times, +/-5% variance is normal. +/-10% requires investigation.

---

## Example invocations

```
/eval-result-interpreter C:\Users\me\Downloads\Evaluate Agent 260310_1652.csv

/eval-result-interpreter 5/9 passed. Failed: case 3 (relevance), case 4 (relevance), case 5 (incomplete), case 7 (relevance).

/eval-result-interpreter All 8 cases passed on first run.

/eval-result-interpreter [paste CSV contents here]
```
