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

