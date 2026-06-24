---
name: eval-triage-and-improvement
description: 'Use this skill when the user''s Copilot Studio agent evaluations have come back and they need to interpret scores, diagnose root causes of underperforming test cases, find remediation steps, or analyze patterns to improve their agent. Always use this skill when the user mentions: "eval failed", "why did this fail", "triage", "diagnose failure", "low pass rate", "fix evaluation results", "not passing", "failing test cases", "evaluation results", "improve my eval scores", or any situation where eval scores need interpretation and action.'
---

# Eval Triage & Improvement

You help users interpret their agent evaluation results and find actionable next steps to improve. Follow the hybrid workflow: gather eval results first, then generate a structured triage report with Step 7 root buckets, owners, and recommended fixes.

This skill is grounded in `skills/eval-guide/playbook.md`, the canonical **Practical Guidance on Agent Evaluation: 10-step playbook**. It is the deep-dive for **Step 7 — Iterate to Diagnose Failures** and seeds **Step 9 — Optimization Loop** for production feedback. MS Learn pages and the Eval Guidance Kit remain supporting sources for Copilot Studio mechanics, lifecycle cadence, and checklist artifacts.

### When to use this skill vs. eval-result-interpreter

These two skills share the same triage framework but serve different modes of work:

| Use **eval-triage-and-improvement** when… | Use **eval-result-interpreter** when… |
|---|---|
| You want **interactive guidance** walking through diagnosis step by step | You have a CSV file or concrete results and want a **one-shot structured report** |
| You are in an **ongoing improvement loop** — fixing, re-running, and re-triaging | This is your **first look** at results — you need a verdict and top actions fast |
| You need **detailed remediation help** for specific eval-set failure patterns (e.g., "wrong tool fires — now what?") | You want a **customer-deliverable artifact** (the .docx triage report) |
| You have **many failures** (15+) and need help prioritizing which to investigate | The eval run is relatively straightforward (<20 failures) |
| You need the playbook worked examples and deeper diagnostic walkthroughs | You need the **activity map / result comparison** tool recommendations inline |

**If in doubt:** Start with eval-result-interpreter to get the structured report, then switch to eval-triage-and-improvement if you need interactive help implementing the fixes.

## Workflow

### Step 1: Gather Eval Results

Ask the user to share:
1. **Which eval sets ran** and their pass rates (e.g., "Faithfulness: 71%, Prompt injection: 95%")
2. **Methodology manifest metadata** from the companion `.docx` report or `stage-N-data.json`: `set_type`, `category`/capability dimension, `method`, `gate` (hard/soft), `target`, `regression_class`, human-review flag, and source/ground-truth provenance
3. **Specific failing test cases** — the test case ID, sample input, expected value, actual agent response, and eval method assigned in Copilot Studio
4. **How many times they've run** — is this the first baseline run (Step 6) or a re-run after fixes?
5. **What they've already tried** — any eval, agent, knowledge, or tool changes attempted so far?

If they don't have structured results, help them organize what they have. Prefer manifest metadata over inferring from filenames or question text. If they just have a general complaint ("my agent isn't working well"), guide them to run a baseline first using the scenario library and the 10-step playbook's Steps 1-6.

### Step 2: Score Interpretation

Assess readiness from the manifest's Step 4 targets and gates:

```
READINESS ASSESSMENT

Any failed hard gate            → BLOCK (trust & safety hard gates block regardless of aggregate pass rate)
Capability set below hard floor → ITERATE or BLOCK based on risk tier and capability criticality
Soft target missed only         → SHIP WITH KNOWN GAPS / ITERATE (tracked, not blocking)
All hard gates + targets pass   → SHIP
```

**Setting thresholds** — don't apply fixed numbers. Use the manifest target first; if missing, derive a provisional target from the agent-level **risk tier** and flag that the manifest needs updating.

| Factor | Higher Threshold When... |
|--------|------------------------|
| Criticality of error | Financial loss, safety risk, legal exposure |
| Reach | External customers or large internal population |
| Autonomy / blast radius | Agent can take actions or trigger downstream systems |
| Regulatory exposure | Regulated workflow, audit requirement, compliance obligation |
| Data sensitivity | PII, PHI, confidential, or tenant-sensitive data |

### Step 3: Pre-Triage Infrastructure Check

Before diagnosing individual failures, verify infrastructure was healthy during the eval run:

- [ ] All knowledge sources accessible and fully indexed?
- [ ] API backends and connectors returned no errors/timeouts?
- [ ] Authentication tokens valid throughout the run?
- [ ] Correct agent version was published and evaluated?

If any dependency was unhealthy, recommend re-running after fixing infrastructure before triaging.

### Step 4: Prioritize Failures

If the user has many failures, recommend this triage order:

| Priority | Triage First | Rationale |
|----------|-------------|-----------|
| 1 | Failed hard gates, especially trust & safety sets | Highest consequence; blocks deploy regardless of aggregate score |
| 2 | High-risk capability failures (accuracy, faithfulness, tool use) | Direct impact on agent value; hallucination is a faithfulness/capability failure |
| 3 | Lowest-scoring eval set failures | Likely systemic — fixing one pattern resolves multiple |
| 4 | Recurring failures across baseline/re-runs | Most diagnosable and regression-prone |
| 5 | Soft-target misses | Important but non-blocking unless pattern worsens |

**15+ failures?** Don't triage every one. Review 3-5 from the lowest-scoring eval set. If they share a root bucket/subtype pattern, fix that and re-run.

### Step 5: Classify Root Cause

For each failure, work through the diagnostic questions in order. **Every failure must end in exactly one Step 7 root bucket**: `eval-setup problem` (response is acceptable; fix the eval) or `agent-quality problem` (real issue; fix the agent/platform and log the pattern).

```
TRIAGE DECISION TREE (for each failing test case)

1. Is the agent's response actually acceptable, even though it failed?
   → YES = Eval-setup problem (grader, expected value, rubric, or method is wrong)

2. Is the expected answer still current against the actual source/ground truth in the manifest?
   → NO = Eval-setup problem (expected answer outdated or source dependency drifted)

3. Does the test case represent a realistic user input for this eval set's `set_type` and category?
   → NO = Eval-setup problem (unrealistic or mis-scoped test case)

4. Could a valid alternative response also be correct, but the grader rejects it?
   → YES = Eval-setup problem (rubric/grader too rigid)

5. Is the eval method appropriate for what you're testing?
   → NO = Eval-setup problem (wrong method; update the manifest and Copilot Studio row assignment)

ALL PASS → The eval is valid. Classify as **agent-quality problem** and proceed to operational subtype diagnosis:

6. Does the issue come from prompt/topic/tool/retrieval configuration or stale knowledge?
   → YES = Agent Configuration / Knowledge Issue (agent-quality problem)

7. Does the behavior persist after reasonable config and knowledge fixes plus re-run?
   → YES = Platform Limitation (agent-quality problem; log evidence and workaround)
```

### Step 5b: Conversation (Multi-Turn) Triage

For conversation eval failures, the standard decision tree still applies but you must first identify the **critical turn** — the earliest turn where the agent went wrong. Everything after a bad turn is a cascade, not independent failures.

**Critical turn identification:**
1. Walk the conversation turn by turn
2. Find the first turn where the agent response diverges from expected behavior
3. Classify that turn using the decision tree above
4. Mark downstream turns as "cascade — blocked by Turn N fix"

**Conversation-specific failure patterns and remediations:**

| Pattern | How to spot it | Root cause area | Remediation |
|---------|---------------|-----------------|-------------|
| **Context loss** — Turn 1 fine, Turn 3+ forgets | Agent re-asks or contradicts earlier turns | Agent Config | Review topic management; ensure conversation context is preserved across topic switches |
| **State loop** — Agent repeats the same response | Identical or near-identical agent turns in sequence | Agent Config | Check topic routing for circular references; add explicit exit conditions |
| **Clarification failure** — Agent can't handle follow-ups | Turn 2 fails when user provides clarification or correction | Agent Config | Add follow-up handling instructions; check that topics accept partial/corrective inputs |
| **Last-mile failure** — Understands but can't resolve | Early turns diagnose correctly, final resolution turn fails | Agent Config or Platform | Check action/connector configuration; verify the resolution path is wired correctly |
| **Eval rigidity** — Conversation is acceptable but grader rejects | Reading the full conversation, the outcome is reasonable | Eval Setup | Conversation grading is limited (AI Generated or Approval Rating only); adjust rubric or expected values |

**Key difference from single-response triage:** Do NOT triage each turn independently. Triage the critical turn, apply the fix, re-run, and then see which downstream turns self-resolve. Expect 40-60% of downstream failures to clear after fixing the critical turn.

### Two Root Buckets + Operational Subtypes

| Step 7 root bucket | Operational subtype | Who acts | What it means |
|-----------|---------|----------|---------------|
| **Eval-setup problem** | Eval setup issue | Eval author | The response is acceptable or the eval metadata/rubric/expected answer/method is wrong. Fix the eval and manifest. |
| **Agent-quality problem** | Agent configuration issue | Agent builder | The agent genuinely produced a bad response. Fix prompt, topics, tools, retrieval, grounding, or knowledge. |
| **Agent-quality problem** | Platform limitation | Platform team + agent owner | The eval caught a real issue caused by platform behavior. Log evidence, workaround if possible, and track the pattern. |

Maintain a **failure-pattern log** for every agent-quality problem: test case, set_type/category, root bucket, subtype, suspected pattern, owner, fix location, verification eval set, and whether it should become a regression case (Step 8).

### Step 6: Map to Remediation

For detailed remediation steps by Step 7 root bucket, operational subtype, and eval-set failure pattern, read the playbook files:
- **Full triage decision tree**: Read `triage-and-improvement-playbook/triage-decision-tree.md`
- **Remediation mapping**: Read `triage-and-improvement-playbook/remediation-mapping.md`
- **Pattern analysis**: Read `triage-and-improvement-playbook/pattern-analysis.md`
- **Worked examples**: Read `triage-and-improvement-playbook/worked-examples.md`

#### Quick Remediation Reference

**Eval-setup fixes:**

| Sub-Type | Fix |
|----------|-----|
| Outdated expected answer | Update expected value to match current source content |
| Overly rigid grader | Switch to Compare Meaning, or broaden keyword set |
| Unrealistic test case | Rewrite input using actual user language |
| Wrong eval method | Change method to match the eval-set purpose and evidence type |
| Grader error/bias | Review rubric, add examples, consider deterministic method |

**Agent-quality fixes — agent configuration / knowledge / tools:**

| Failure pattern | Common Fix |
|-----------------|-----------|
| Factual accuracy (wrong source) | Review knowledge source config, verify indexing, check vocabulary match |
| Factual accuracy (wrong extraction) | Add extraction guidance to system prompt |
| Hallucination (faithfulness capability failure) | Improve retrieval/chunking first; add instruction: "Only answer from knowledge sources. If unavailable, say so." |
| Wrong tool fires | Rewrite tool descriptions to differentiate; add negative examples |
| Tool doesn't fire | Review trigger conditions; check if tool is enabled and accessible |
| Wrong topic fires | Review trigger phrase overlap; adjust priority ordering |
| Lacks empathy | Add context-specific tone instructions to system prompt |
| Scope violation | Add explicit out-of-scope instruction |
| PII leakage | Add PII protection instruction; review authentication scope |

**Agent-quality fixes — platform limitation response:**
- Document the limitation with evidence
- Implement workaround where possible
- Adjust eval thresholds to account for known platform behavior
- File with platform team with reproduction steps

### Step 7: Triage Rationale (teach the WHY)

Before generating the report, add rationale that teaches the customer the reasoning behind triage decisions — not just the conclusions. For each of these, use the actual eval data from this triage:

1. **Why each failure got its root bucket and subtype** — Walk through the decision tree for at least one example per Step 7 root bucket. E.g., "Test case KB-014 was classified as an Eval Setup Issue because the agent response is factually correct per the current knowledge source, but the expected value still references the old 14-day policy. The agent is right; the eval is stale."

2. **Why the remediation targets config vs. content vs. eval** — Explain the logic: "We recommended updating the knowledge source rather than changing the prompt because the agent retrieval worked correctly — it found the right document — but the document itself contains outdated information. A prompt change would mask the real problem."

3. **Why the priority order is what it is** — Connect to blast radius and dependency chains: "Failed hard gates come first because they block deploy and can change downstream behavior. Fix the hard-gate issue, re-run the regression suite, then triage the rest — otherwise you may diagnose failures that disappear once the blocking guardrail is corrected."

4. **What this triage does NOT tell you** — Name the limits explicitly: "This triage analyzed [N] failures from a single eval run. It cannot detect issues in scenarios you have not written test cases for, and it cannot distinguish between a flaky failure (non-determinism) and a real failure from a single data point. If a failure is borderline, re-run before investing in a fix."

Include this rationale in the triage report (see Triage Rationale section in the report template below).

### Step 8: Generate Triage Report

Output a structured triage report:

```markdown
# Triage Report: [Agent Name] — [Date]

## Score Summary
| Eval Set | Set Type / Category | Pass Rate | Target | Gate | Status |
|----------|---------------------|-----------|--------|------|--------|
| ... | capability / faithfulness | ... | ... | hard/soft | PASS/BLOCK/ITERATE |

## Readiness Assessment
[SHIP / SHIP WITH KNOWN GAPS / ITERATE / BLOCK]
[Rationale]

## Failure Analysis
### Failure 1: [Test Case ID]
- **Set Type / Category:** capability or trust_safety / ...
- **Eval-set focus:** ...
- **Sample Input:** ...
- **Expected:** ...
- **Actual:** ...
- **Root Bucket:** [Eval-setup problem / Agent-quality problem]
- **Operational Subtype:** [Eval Setup / Agent Config / Knowledge / Platform Limitation]
- **Diagnosis:** [specific diagnosis]
- **Owner:** [who needs to act]
- **Remediation:** [specific action]
- **Verification:** [how to verify the fix worked]

[Repeat for each triaged failure]

## Triage Rationale
### Why these root bucket classifications
[Walk through the decision tree for representative examples — show the reasoning, not just the label]

### Why these remediations
[Explain the logic connecting root bucket/subtype to fix — why this fix and not an alternative]

### Why this priority order
[Connect priority to blast radius and dependency chains]

### What this triage does NOT tell you
[Name the limits: coverage gaps, single-run non-determinism, untested scenarios]

## Failure-Pattern Log
[Summarize recurring Step 7 patterns, owner, fix location, and whether each pattern should be added to the Step 8 regression suite]

## Systemic Patterns
[If 80%+ of failures share a root bucket/subtype/category, call it out]

## Action Items
| # | Action | Owner | Priority | Verification |
|---|--------|-------|----------|-------------|
| 1 | ... | ... | ... | Re-run [eval set] |

## Post-Triage Checklist
- [ ] All failed hard gates addressed before deploy
- [ ] Root buckets verified by reading actual responses
- [ ] Eval-setup fixes applied to expected answers/rubrics/method assignments/manifest
- [ ] Agent-quality patterns logged with owners and fix location
- [ ] Full Step 8 regression suite re-run after fixes
- [ ] Platform limitations filed if applicable

## Human Review Required
[Include human review checkpoints table — see Human Review Checkpoints section below]
```

### Post-Triage Verification

After fixes are applied:
- **Scores flat after fix?** → Wrong root bucket/subtype, re-triage
- **One score up, another down?** → Instruction conflict — the fix improved one behavior but degraded another
- **80%+ of failures share a root bucket/subtype?** → Systemic issue — fix the category, not individual test cases

## Non-Determinism Handling

LLM-based agents and graders produce variable outputs:
- **Establish baselines:** Run 3+ times before treating any score as the Step 6 baseline. Use the average and record agent version + timestamp.
- **Normal variance:** +/-5% between runs is expected. Investigate if >10%.
- **Flaky test cases** (pass sometimes, fail others): Agent may produce two valid responses but eval is too rigid. Investigate whether to broaden the expected value.
- **Small eval sets (<30 test cases):** A single test case flip changes the score by 3%+. Don't over-interpret.

## Step 9 Optimization Loop: Production Signals

If the agent is deployed (even in preview), treat production feedback as the Step 9 loop: **collect signals → cluster → decide fix location → ship → re-evaluate against the Step 8 regression suite**. Prioritize signals in this order: thumbs-down (highest-signal negative feedback), escalations, manual overrides, support tickets, then qualitative comments.

- **High thumbs-down on a topic where eval passes:** Coverage gap. Add or revise eval cases and tag them for regression if they represent recurring production risk.
- **Thumbs-down clustering after a config change:** Possible regression. Re-run the Step 8 regression suite and add a case if the suite missed it.
- **Escalations/manual overrides:** Agent-quality problem until proven otherwise; cluster by capability or trust & safety category and choose the fix location: agent config/retrieval/tools, rubric/expected answer, or new eval coverage.
- **Steady thumbs-up on a topic where eval fails:** Possible eval-setup problem. Review the actual responses before weakening a gate.

Production signals are not verdicts by themselves. They seed hypotheses, new eval cases, and failure-pattern log entries; the regression suite verifies the fix.

## Human Review Checkpoints

Before acting on the triage report, review these checkpoints. Triage decisions directly drive agent changes — a wrong diagnosis wastes an entire iteration cycle.

| # | Checkpoint | Why it matters |
|---|---|---|
| 1 | **Verify Step 7 root buckets yourself** — For each failure classified as an eval-setup problem, read the agent actual response. Is it truly acceptable, or is the triage giving the agent the benefit of the doubt? | Misclassifying agent-quality problems as eval setup means real problems get ignored. The two-bucket distinction requires judgment, not score-only automation. |
| 2 | **Confirm systemic pattern diagnoses before applying systemic fixes** — If the report says 80%+ failures share a root bucket/subtype, verify by reading the actual responses. Similar symptoms can have different causes. | A wrong systemic diagnosis means you apply one fix expecting to resolve many failures, but only fix some or none. |
| 3 | **Validate remediation feasibility and priority order** — Can your team actually make the suggested changes? Is the priority order right for your timeline and constraints? | The triage prioritizes by impact, but your team knows effort and dependencies. A knowledge source fix may take 2 weeks; a prompt tweak may unblock you now. |
| 4 | **Check that proposed fixes will not regress passing scenarios** — Before making changes, consider which currently-passing test cases could be affected. Prompt changes especially have ripple effects. | Fixing 3 failures while introducing 5 new ones is a net loss. Plan to re-run the full suite after any agent configuration change. |
| 5 | **Validate platform limitation classifications before escalating** — If a failure is classified as a platform limitation, confirm the behavior persists across multiple prompt and config variations before filing with the platform team. | Escalating a configuration issue as a platform bug wastes platform team time and delays your actual fix. |
| 6 | **Review manifest targets and gates against your actual risk tier** — Does SHIP/ITERATE/BLOCK honor hard gates, soft targets, and the five risk factors? | Only your team knows your real risk tolerance. A soft-target miss may be acceptable for an internal helper but a failed hard trust & safety gate blocks deploy. |

Include this table in the triage report output. Add: This triage report accelerates diagnosis but does not replace human judgment. Review checkpoints 1 and 2 before acting on any remediation — the distinction between eval-setup problems and agent-quality problems requires reading the actual responses.

## Data Retention Warning

Copilot Studio **deletes test run results after 89 days**. This means your baseline results from an initial eval may be gone before your next quarterly review. After every triage cycle:

1. **Export the results CSV** immediately (Test set → Export results)
2. **Store alongside your triage report** in SharePoint, a repo, or wherever your team keeps versioned artifacts
3. **Tag with agent version and date** so future comparisons are possible

If your triage identified a fix-and-rerun cycle, export the pre-fix results *before* applying changes. You need the before/after comparison, and Copilot Studio won't keep the "before" forever.

## Cross-Reference

This skill uses `skills/eval-guide/playbook.md` as the methodology spine. It also works alongside the **AI Agent Evaluation Scenario Library** (`github.com/microsoft/ai-agent-eval-scenario-library`), which defines supporting scenario patterns and quality dimensions, and the **Triage & Improvement Playbook** (`github.com/microsoft/triage-and-improvement-playbook`), which provides supporting diagnostic frameworks for Step 7.

### Related eval skills

| After triage, if you need to... | Use this skill |
|---|---|
| Build or expand the eval plan with new scenarios identified during triage | `/eval-suite-planner` |
| Generate new test cases for expanded or revised scenarios | `/eval-generator` |
| Get a quick structured report from a new CSV (without interactive triage) | `/eval-result-interpreter` |
| Answer a methodology question that came up during triage | `/eval-faq` |
| Walk the customer through the full eval pipeline end-to-end | `/eval-guide` |
