<!-- Companion reference for the `eval-guide` skill. Loaded on demand from SKILL.md. -->

## Eval Maturity Journey

Use the **Per-Agent Eval Maturity Model** as an **outcome scorecard** to orient customers on where they are today and where this session takes them. It is the progress-framing layer over the 10-step playbook (the canonical methodology lives in `playbook.md`). Five pillars of eval practice, five levels each — from `L100 Initial` (no practice in place) to `L500 Optimized` (continuous improvement built into operations). Assume the agent starts at **L100 Initial on all pillars**. This session targets **L300 Systematic on Pillars 1, 2, and 4** (in-session deliverables) and **L200 Defined on Pillars 3 and 5** (via reference protocols delivered alongside the session).

The full 5×5 definitions live in `maturity-model.md` — that file is the canonical scorecard reference. Each pillar maps to playbook steps (P1=Step 1, P2=Steps 2–5, P3=Steps 6+8, P4=Steps 7+9, P5=Step 8). Update `maturity-model.md` first when level definitions change.

| Pillar | What it measures | After this session | Mechanism |
|---|---|---|---|
| **1 — Define what "good" means** | Acceptance criteria quality | L300 Systematic ✓ | Stage 0 (Discover) + Stage 1 (Plan) |
| **2 — Build your eval sets** | Coverage and versioning | L300 Systematic ✓ | Stage 2 (Generate) |
| **3 — Run evals across the lifecycle** | Where and when evals execute (offline, pre-deploy, production) | L200 Defined ✓ | `rerun-protocol-<agent>-<date>.docx` (starter artifact) |
| **4 — Improve and iterate** | How improvements are validated | L300 Systematic ✓ | Stage 4 (Interpret) — only if eval results are available |
| **5 — Handle changes with confidence** | How changes (prompts, tools, models, architecture) get tested before shipping | L200 Defined ✓ | `baseline-comparison-<agent>-<date>.xlsx` (starter artifact) |

**Pillars 3 and 5 stop at L200 Defined this session.** L300 Systematic on those pillars requires operating practice — a release cadence with codified triggers (Pillar 3) and version-tagged baselines accumulated over multiple changes (Pillar 5). The starter artifacts get the customer to L200 in one session: a documented protocol and a fill-in workbook they can execute when triggered. Generate `rerun-protocol-<agent>-<date>.docx` and `baseline-comparison-<agent>-<date>.xlsx` at the end of Stage 2 (see deliverables C and D in Stage 2's "After confirmation" block).

Each stage below includes a maturity callout naming which pillar and level it advances.

---

