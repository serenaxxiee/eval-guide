<!-- Companion reference for the `eval-guide` skill. Loaded on demand from SKILL.md. -->

## How This Maps to Microsoft's 10-Step Eval Playbook

The toolkit's canonical methodology is **Microsoft's *Practical Guidance on Agent Evaluation* — a 10-step playbook** (full definition in `playbook.md`). The operational stages below are the **session UX**; each one delivers specific playbook steps. Share this crosswalk with customers so they see how the accelerator maps to the guidance. Prefer stage **names** over numbers when talking to customers — the playbook owns the numbering.

| Operational stage | Playbook steps delivered | What it means |
|---|---|---|
| **Discover** | **Step 1** — Plan the eval effort | Name the eval objective, classify the agent's **risk tier** (5 factors), name an owner. Articulate purpose/users/boundaries/success — the eval spec. |
| **Plan** | **Steps 2–5** (plan side) | Decompose into **capability** eval sets and **trust & safety** eval sets, set **pass-rate targets + hard/soft gates**, specify **human inputs** (rubrics, ground truths, source→ground-truth map). |
| **Generate** | **Steps 2, 3, 5** (build) + **Step 8** (design) | Produce the capability + trust & safety eval sets (CSVs + manifest); tag each set `gate-only | regression | exploratory` for the regression suite. |
| **Run** | **Step 6** — Run the baseline | Execute the suite vs the current build; record per-set results with version + timestamp. |
| **Interpret** | **Step 7** — Iterate to diagnose (+ **Step 9** design) | Classify each failure as eval-setup vs agent-quality; SHIP/ITERATE/BLOCK on gates; design the production optimization loop. |
| **Closeout** _(folded into the Interpret report)_ | **Step 10** — Reusable assets | Flag reusable rubrics / trust & safety sets for the shared library (Required / Recommended / Opt-in). |

Steps 8–10 (regression suite, optimization loop, reusable assets) are **designed in-session** and **run over time** — the session leaves the customer reference artifacts to execute them.

**When to share this:** After Discover, show the customer the crosswalk and say: *"Today covers Steps 1–5 of Microsoft's playbook — planning the effort and building your capability and trust & safety eval sets. Once you have a running agent you'll run the baseline (Step 6), iterate (Step 7), then stand up the regression suite, optimization loop, and shared-asset library (Steps 8–10)."*

**Downloadable reference:** Point customers to Microsoft's [Eval Guidance Kit](https://aka.ms/EvalGuidanceKit) to track progress through all ten steps independently.

---

