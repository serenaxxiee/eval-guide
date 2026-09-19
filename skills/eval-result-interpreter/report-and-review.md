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

