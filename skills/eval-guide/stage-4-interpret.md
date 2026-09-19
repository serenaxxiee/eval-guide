<!-- Companion reference for the `eval-guide` skill. Loaded on demand from SKILL.md. -->

## Stage 4: Interpret

Stage 4 turns raw results into a ranked action list. Every failure gets classified by root cause; the Top 3 actions get phrased as Change-X → Re-run-Y → Expect-Z. The output is a `.docx` triage report your team works from. 30–45 minutes.

### What you walk away with

- **A gate-based SHIP / ITERATE / BLOCK verdict** — driven by the hard/soft gates defined in the eval manifest, not the aggregate pass rate.
- **Gate status per eval set** — capability and trust & safety sets reported separately with target, actual, gate type, and PASS/MISS status.
- **Regression/direction evidence as supporting context** — capability trends help prioritize fixes, but they do not override hard/soft gates.
- **Failure triage table** — every failure classified first into the playbook Step 7 buckets (**Eval Setup** vs **Agent Quality**), then into a fix subtype such as agent configuration or platform limitation. The classification points at the fix.
- **Top 3 actions** in Change → Re-run → Expect format.
- **A `.docx` triage report** for your team to act from.

### When this stage is wrong for you

- You don't have eval results yet. Run Stage 3 first.
- You already know what to fix and don't need the diagnostic. Skip the full triage; just re-run after your change.

### Stage 4 is a loop, not an end

After implementing the Top 3 actions, **re-run Stage 3 and re-do Stage 4 with the new results**. The before/after comparison validates whether the fix worked — and that before/after evidence is what advances Pillar 4 to L300 Systematic. A single Stage 4 pass without a follow-up re-run leaves Pillar 4 at L200.

### The 20% rule — the most counterintuitive insight in this skill

**At least 20% of failures in a new eval are eval setup bugs, not agent bugs.** The test case might be wrong, the expected response might be outdated, the testing method might be inappropriate, or the LLM judge might have misread the response. **Don't blame the agent until you've checked the test.**

Tell the customer explicitly: *"Before we blame the agent — at least 20% of failures in a new eval are eval setup issues. Let me apply the 5-question eval verification before classifying failures as agent bugs."*

This single discipline is what separates productive triage from churn.

### Read gates first, then eval-set performance

The headline pass rate ("60% passed") is wrong as a verdict. The first read is **gate status** from the manifest:

- **Hard gate** — must pass before deploy. Any failed hard gate means the agent cannot ship as-is.
- **Soft target** — tracked and remediated, but not blocking by itself.
- **Trust & safety hard gate** — usually a **BLOCK** verdict when missed.
- **Deployment-critical capability hard gate** — **BLOCK** or **ITERATE** based on severity, owner risk tolerance, and the gate policy.

Apply this decision rule:

```
ANY hard gate missed?
    YES -> cannot SHIP.
           Trust & safety hard-gate miss -> usually BLOCK.
           Deployment-critical capability hard-gate miss -> BLOCK or ITERATE.
           Other hard-gate miss -> ITERATE until fixed.
    NO  ->
        ANY soft target missed?
            YES -> ITERATE: track the gap and fix in priority order.
            NO  -> SHIP, assuming human review agrees coverage is sufficient.
```

Then read **eval-set trends and regression/direction** as prioritization evidence. Capability misses tell you where to improve next; hard-gated Trust & Safety misses tell you what blocks release. If no workbook/manifest is present, infer gates only as a fallback and state: *"No workbook or manifest provided — gate status inferred and should be reviewed by the owner."*

A green-across-the-board run is rare on first iteration; expect 2–3 Plan→Interpret cycles before every hard gate passes and soft targets are acceptable. Tell the customer the gate decision so they know what blocks shipping versus what should be tracked.

**Which skill to use:** For a one-shot triage report from a CSV file or results summary, invoke `/eval-result-interpreter`. For interactive, multi-round diagnosis with detailed remediation guidance, invoke `/eval-triage-and-improvement`. Start with the interpreter; switch to triage if you need help implementing fixes.

### What to do

1. **Pre-triage check — scan for infrastructure symptoms before classifying any failure.** Don't just ask "was everything working?" — that's a yes/no the customer can't answer accurately. Look for these symptoms in the results data:

   | Symptom in results | Likely cause | Action |
   |---|---|---|
   | Empty agent response on multiple cases | Auth failure or timeout, not agent error | Don't count as agent failure — flag for re-run after infra fix |
   | Sudden cluster of fails all citing one source | That source was unreachable during run | Verify source connectivity, re-run those cases |
   | Same case passes/fails inconsistently across re-runs (>10% swing) | Non-determinism beyond normal LLM variance — likely caching, latency, or auth-token expiry mid-run | Re-run 2–3 times, take median |
   | Cases tagged to a user profile got responses for a different profile | Profile assignment misconfigured at import | Fix profile tags, re-run those cases |
   | Refusal cases pass but with generic "I can't help" not the expected escalation language | Agent has the refusal but lacks the escalation routing | Real agent issue — keep counted as failure, but classify as Agent Config (incomplete refusal) not Safety failure |

   Confirm with the customer: *"I see [N] empty responses / [N] cases all on Source X / [other pattern]. Was [auth / connectivity / etc.] healthy during the run?"* Don't blame the agent for symptoms that match infrastructure patterns until the customer confirms.

   If anything was broken during the run, the run is invalid — re-run before triaging.

2. **Gate summary and verdict** — Build the eval-set table from the workbook/manifest: eval set, set type, category / dimension, testing method, intended use/cadence, target, actual, gate type, and gate status. Drive SHIP / ITERATE / BLOCK from this table. Keep capability and trust & safety sets separate. Use capability trends and regression/direction only as supporting prioritization.

3. **Failure triage with the 20% rule** — apply 5-question eval verification to each failure before classifying it as an agent bug. ~20% will move to Eval Setup root cause.

4. **Root causes:** First classify each failure into the playbook Step 7 bucket:
   - **Eval Setup Issue** — the test, expected answer, method, rubric, or judge is wrong. Fix the eval.
   - **Agent Quality Issue** — the eval correctly caught a real agent problem. Then subtype it as **Agent Configuration Issue** (prompt/topic/tool/retrieval/config fix) or **Platform Limitation** (known platform behavior / connector limitation / unsupported scenario). Fix or mitigate the agent path.

5. **Top 3 actions** — Each: **Change** X → **Re-run** Y → **Expect** Z. When re-running, run the full test set, not just the failing cases, to catch regressions elsewhere. Save pre-fix pass rates and compare before/after — that before/after evidence is what distinguishes L300 Systematic Pillar 4 from L200 Defined.

6. **Pattern analysis** and **next-run recommendation.**

### Override the LLM judge when it's wrong

The dashboard's **Agree / Disagree** buttons per case are the central mechanism for handling LLM-judge errors — not a power-user feature. ~5–10% of "fails" are judge errors (judge misread the response, missed an implicit citation, over-penalized minor phrasing). When you disagree, click Disagree — the case flips to an Eval Setup root cause and stops counting against the agent.

**Use this aggressively.** A pass rate built on uncorrected judge errors is a false signal. Domain expertise wins over the judge every time.

### A 100% pass rate is a red flag

If everything passes, your eval is too easy. Real agents in real production conditions don't pass 100% of well-designed tests. Add harder cases — adversarial inputs, paraphrase variants, boundary conditions, sensitive-data probes. A 100% pass rate without harder cases is comfort, not evidence.

**The customer payoff:** *"You now have a ranked action list — three specific things to change, what to re-run after each, and what outcome to expect. Combined with the rerun protocol and baseline-comparison workbook from Stage 2, you can close the loop on this eval today and the next one in half the time."*

**Maturity callout — Pillar 4 / playbook Steps 7, 9 (L100 Initial → L300 Systematic):** Interpret advances Pillar 4 from reactive fixing to structured root-cause analysis (each failure classified eval-setup vs agent-quality), before/after validation, and regression-proofing — plus designing the production optimization loop (Step 9). All three in-session pillars (1, 2, 4) are now at L300 Systematic. Pillars 3 (Run evals across the lifecycle) and 5 (Handle changes with confidence) reach L200 Defined via the `rerun-protocol-<agent>-<date>.docx` and `baseline-comparison-<agent>-<date>.xlsx` starter artifacts generated at session close.

### Interactive Dashboard Checkpoint

Before generating the final triage report, launch the interpret dashboard for review:

1. Write the triage data to `stage-4-data.json` using **eval sets as the gate source of truth** and cases as the expandable detail:
   ```json
   {
     "agent_name": "...",
     "summary": {"total": 28, "passed": 19, "failed": 9},
     "eval_sets": [
       {
         "id": "accuracy",
         "name": "Accuracy",
         "set_type": "capability",
         "category": "Accuracy",
         "method": "Compare meaning",
         "regression_class": "regression",
         "target_pass_rate": 90,
         "gate": "hard",
         "deployment_critical": true
       },
       {
         "id": "prompt_injection",
         "name": "Prompt injection / Jailbreak",
         "set_type": "trust_safety",
         "category": "prompt_injection",
         "method": "Compare meaning",
         "regression_class": "gate-only",
         "target_pass_rate": 100,
         "gate": "hard"
       }
     ],
     "eval_results": [
       {"eval_set_id": "accuracy", "case_id": 1, "question": "...", "expected": "...", "actual": "...", "method": "Compare meaning", "score": 0.92, "pass": true, "explanation": "Rationale from LLM judge..."}
     ],
     "failures": [
       {"id": 1, "eval_set_id": "prompt_injection", "case_id": 2, "question": "...", "expected": "...", "actual": "...", "root_cause": "agent_config", "explanation": "..."}
     ],
     "top_actions": [...],
     "patterns": [...]
   }
   ```
   Key requirements:
   - `eval_sets` is the authoritative gate table. Each set must have `id`, `name`, `set_type` (`capability` or `trust_safety`), `category` / dimension, `method`, intended use/cadence or `regression_class`, `target_pass_rate`, and `gate` (`hard` or `soft`). Capability hard gates that are deployment-critical should set `deployment_critical: true`.
   - `eval_results[*].eval_set_id` must point to an `eval_sets[*].id`. The dashboard aggregates case results into set-level actual pass rates, compares them to targets, and computes the live SHIP / ITERATE / BLOCK verdict.
   - `eval_results` contains ALL test case results (not just failures) so cases can be expanded in the dashboard and human Agree / Disagree overrides can recompute gate status.
   - Each eval result includes `explanation` (the LLM judge rationale) for human review
   - Do **not** precompute a static `verdict` as the source of truth. The dashboard live-computes the verdict from `eval_sets` + `criterion_metrics` + human overrides, then returns `computed_verdict` and `gate_summary` in feedback for the final report.
2. Launch the dashboard (resolve `<skill-dir>` from the skill context):
   ```bash
   python "<skill-dir>/dashboard/serve.py" --stage interpret --serve --data stage-4-data.json
   ```
3. The user reviews the gate verdict first, then eval-set pass rates and regression/direction evidence, expands set rows to see test case details, uses Human Judgement (Agree/Disagree) to override LLM judge assessments, and re-classifies root causes. Disagreed failed cases are treated as eval-setup issues and no longer count against the agent's gate status; the dashboard recomputes set pass rates and the verdict live.
4. When the user confirms, **parse the feedback from the bash stdout** between the `===EVAL_GUIDE_FEEDBACK_BEGIN===` / `===EVAL_GUIDE_FEEDBACK_END===` markers. **Apply every edit it contains, faithfully and without question.** The customer's choices are final — do NOT re-litigate, do NOT suggest reverting, do NOT ask for confirmation again, do NOT partially apply. (`interpret-feedback.json` is also on disk as a backup, but stdout is the primary channel.)

   This applies to ALL edit types:
   - **`human_disagrees`** — every Disagree is the customer overriding the LLM judge. Each disagreed failed case flips to `Eval Setup Issue` root cause and stops counting against the agent's gate status. The customer's domain expertise wins; do not override their override.
   - **`computed_verdict` and `gate_summary`** — the dashboard's post-review verdict and set-level gate table. Use these in the final report; do not recompute independently unless the feedback is missing them.
   - Root cause reclassifications per failure (Eval Setup, Agent Quality — Config, Agent Quality — Platform)
   - Top-3-action edits
   - General Comments box content

   **Then narrate the edits back** — count Disagrees applied, list re-classified root causes, name any Top-3-action edits. Example: *"Got it — 4 Disagrees flipped to Eval Setup, root cause for failure #7 reclassified from Agent Quality — Config to Agent Quality — Platform, and Top action #2 edited to scope to the prompt-injection gate. Updated gate verdict: BLOCK -> ITERATE after Disagrees applied."* Don't just say "applied." The narration confirms you parsed correctly; it is NOT an invitation to re-decide.

   If changes requested instead of confirmed, regenerate and re-launch.
5. **After confirmation**, generate the customer-ready .docx triage report using the `/docx` skill. Same principles: concise, presentable, self-contained. Structure:
   1. **Gate verdict** — prominent SHIP / ITERATE / BLOCK decision from `computed_verdict`, plus the set-level gate table from `gate_summary` (eval set, set type, category / dimension, method, regression class, target, actual, gate type, status). State explicitly that the verdict is gate-based, not aggregate-pass-rate-based.
   2. Eval-set performance — set summary cards/table with actual pass rate, target, gate status, intended use/cadence, and regression/direction notes
   3. Failure triage table (eval set, case, question, expected, actual, root cause) — include human-disagreed entries as "Eval Setup — Human Disagrees"
   4. Top actions (Change → Re-run → Expect)
   5. Pattern analysis — eval-set patterns highlighting systemic issues, hard-gate misses, regressions, and source/tool clusters
   6. Next steps. **Always include a pointer line:** *"You're also keeping the companion artifacts from Stage 2 — `eval-setup-guide-<agent>-<date>.docx` (step-by-step Copilot Studio setup), `rerun-protocol-<agent>-<date>.docx` (Pillar 3 L200), and `baseline-comparison-<agent>-<date>.xlsx` (Pillar 5 L200). They walk you through how to set up the run, when to re-run, and how to compare runs."* (If Stage 2 was skipped, generate them now using the same flow as Stage 2 deliverables C, D, and E.)
   7. **Optimization loop plan (playbook Step 9)** — a short forward-looking section: which production signals to collect (thumbs-down — highest signal — plus escalations, manual overrides, support tickets), how to cluster them, how to decide where each cluster gets fixed (agent config / rubric / new eval cases), and that every shipped fix is re-validated against the regression suite from Step 8. Frame it as the bridge from "this eval" to "continuous improvement once the agent is in production."
   8. **Reusable-asset register (playbook Step 10)** — scan the criteria, rubrics, and trust & safety sets produced this session and flag the ones that are NOT specific to this agent (e.g., a prompt-injection set, a PII-handling rubric, a tone rubric). List each as a reusable candidate with a tier — **Required** (org-wide gate every agent must pass), **Recommended** (per-category default), or **Opt-in** (domain-specific) — plus provenance and a suggested owner. This is the seed of the shared eval library.
   9. Maturity snapshot — same before/after table as the Stage 2 report, updated to reflect Pillar 4 now at L300 Systematic:

      | Pillar | Baseline | After this session | Next-session target |
      |---|---|---|---|
      | 1 — Define what "good" means | L100 Initial | L300 Systematic ✓ | — |
      | 2 — Build your eval sets | L100 Initial | L300 Systematic ✓ | — |
      | 3 — Run evals across the lifecycle | L100 Initial | L200 Defined ✓ (via `rerun-protocol-<agent>-<date>.docx`) | L300 Systematic |
      | 4 — Improve and iterate | L100 Initial | L300 Systematic ✓ | — |
      | 5 — Handle changes with confidence | L100 Initial | L200 Defined ✓ (via `baseline-comparison-<agent>-<date>.xlsx`) | L300 Systematic |

---

