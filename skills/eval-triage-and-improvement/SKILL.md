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

**Core discipline — 20% rule:** In a new eval, assume at least 20% of failures are **eval setup problems**, not agent bugs, until triage proves otherwise. Every failure classifies first as Eval Setup vs Agent Quality; do not skip that root-bucket decision.

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

Open `remediation-catalog.md` when you need concrete remediation steps for a classified failure. It contains the full moved section for Step 6, including the playbook file pointers, quick remediation reference, eval-setup fixes, agent-quality fixes, and platform-limitation response.

### Step 7 and report/optimization references

Open `triage-report-and-optimization.md` before writing triage rationale, generating the triage report, doing post-triage verification, handling non-determinism, using Step 9 production signals, presenting human review checkpoints, or warning about data retention. It contains the full moved sections for Step 7 onward, including the report template and appendices.
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
