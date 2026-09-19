# Eval generator operations and examples

Open this companion when preparing customer-facing operational guidance, examples, or cross-skill handoff notes. It contains the moved operational tips, examples, and companion-skill reference material from `SKILL.md` verbatim.

---

### Operational tips for the customer

- **89-day result retention.** Copilot Studio retains run results for 89 days. Always export to CSV after every run.
- **100-case-per-test-set limit.** If a single set has more than 100 cases, split it (e.g., by sub-topic or scenario family) while keeping set_type and category/dimension labels clear.
- **Set as the unit of versioning.** Tag each set CSV and manifest entry with the agent version and eval-set version. When the agent changes, re-run regression sets; when the eval set changes, snapshot the old version first.
- **Production failures become test cases.** Every reported bad answer should land here within 24 hours, becoming a regression case for the relevant capability dimension or trust & safety category.
- **Step 8 partition drives cadence.** Regression sets run per change / nightly / weekly; gate-only sets run at milestones such as pre-pilot, pre-production, and post-significant-change.
- **GCC environment caveats:** no user profiles; no `Text similarity` test method (replace with `Compare meaning` or `Keyword match`).
- **Real failures > synthetic cases.** Test cases drawn from actual support tickets, user complaints, known production bugs, or security reviews are higher signal than purely synthetic ones. Prioritize real-failure-sourced cases when available.

---

## Example invocations

```
/eval-suite-planner I'm building an HR policy bot...
[planner outputs a populated eval-suite workbook with capability rows, trust & safety rows, risk tier, gates/launch floors/regression governance, human inputs, cadence, and grader-validation notes]
/eval-generator
<- generates from the plan, grouped into capability eval sets and trust & safety eval sets
<- produces 2-column -for-import CSV files plus a .docx manifest report

/eval-generator I'm building a meeting-notes agent that takes a transcript and produces structured action items.
<- generates from scratch, 6-8 cases, at least one capability set and one trust & safety set

/eval-generator I'm building a travel-booking agent that handles multi-turn flight search, seat selection, purchase.
<- detects multi-turn behavior, generates 4-6 conversation test cases as a planning blueprint
<- preserves capability vs trust & safety labeling and recommends complementary single-response sets

/eval-generator
<- no plan, no description provided — asks for input
```

---

## Companion skills

- **`/eval-suite-planner`** — Plan: produces the eval plan this skill consumes.
- **`/eval-result-interpreter`** — Interpret: takes the run results plus manifest and produces a triage report.
- **`/eval-faq`** — methodology Q&A grounded in Microsoft's eval ecosystem.
- **`/eval-guide`** — the orchestrator. Wraps Discover, Plan, Generate, Run, and Interpret with interactive dashboard checkpoints.
