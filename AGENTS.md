# Eval Guide — AI Agent Evaluation Toolkit

This repository contains an AI agent evaluation toolkit for [Copilot Studio](https://copilotstudio.microsoft.com), grounded in Microsoft's ***Practical Guidance on Agent Evaluation* — a 10-step playbook**. The canonical methodology spine is `skills/eval-guide/playbook.md`.

## What This Toolkit Does

Helps users go from "I don't know where to start with eval" to "I have a plan, test cases, and know how to interpret results" — in one session. No running agent required for planning and test generation.

## Available Prompt Files

This toolkit provides 6 prompt files in `.github/prompts/`. When the user's request matches one of these, attach or reference the appropriate prompt file:

| Prompt File | When to Use |
|---|---|
| `eval-guide.prompt.md` | Full eval lifecycle — discover, plan, generate, run, interpret. **Start here** when the user mentions agent evaluation, eval planning, "what should we test", or "how do we know if the agent is good". |
| `eval-suite-planner.prompt.md` | Populated Eval Suite Template workbook plus an interactive HTML review page with eval sets, methods, gates, human inputs, and grader-validation notes. Use when the user has an agent description and needs a plan before generating test cases. |
| `eval-generator.prompt.md` | Generate test cases (CSV for single-response, blueprints for multi-turn). Use after planning, or standalone with an agent description. |
| `eval-result-interpreter.prompt.md` | SHIP / ITERATE / BLOCK verdict from eval results. Use when the user has CSV results or pass/fail data to interpret. |
| `eval-triage-and-improvement.prompt.md` | Interactive diagnosis and remediation for failing evals. Use when the user needs help debugging specific failures. |
| `eval-faq.prompt.md` | Methodology questions answered from Microsoft's eval ecosystem. Use for "how do I...", "what is...", "when should I..." eval questions. |

## Routing Guide

| User says... | Use this prompt |
|---|---|
| "We're planning to build an agent for..." | eval-guide |
| "Help us think through what good looks like" | eval-guide |
| "Here's our agent, plan the eval" | eval-suite-planner |
| "I have a plan, generate test cases" | eval-generator |
| "My evals came back, what do they mean?" | eval-result-interpreter |
| "Some tests are failing and I don't know why" | eval-triage-and-improvement |
| "How is evaluating X different from Y?" | eval-faq |

## Methodology Summary

This toolkit is grounded in Microsoft's ***Practical Guidance on Agent Evaluation* — a 10-step playbook**. The canonical methodology spine lives in `skills/eval-guide/playbook.md`; every skill, prompt, dashboard, and doc derives its framing from that file. Do not restate the methodology elsewhere — point to the playbook.

The 10 steps:

1. **Plan the eval effort** — eval objective, agent **risk tier** (5 factors: reach, criticality of error, autonomy/blast radius, regulatory exposure, data sensitivity), named owner
2. **Build the capability eval sets** — one set per capability (accuracy, faithfulness/groundedness [hallucination lives here], relevancy, style/tone, reasoning/tool use)
3. **Build the trust & safety eval sets** — separate from capability: guardrails, out-of-scope, sensitive-data, prompt-injection/jailbreak, compliance
4. **Define pass-rate targets and gates** — per set; **hard gates** (block deploy) vs **soft targets** (tracked)
5. **Specify human inputs** — rubrics, ground truths, golden answers + a source-to-ground-truth dependency map
6. **Run the baseline** — full suite vs current build, recorded with version + timestamp
7. **Iterate to diagnose failures** — every failure is an eval-setup problem OR an agent-quality problem
8. **Regression suite** — partition into regression sets (cadence) vs gate-only sets (milestones)
9. **Optimization loop** — production signals (thumbs-down highest) → cluster → fix → re-evaluate
10. **Identify & save reusable assets** — promote non-agent-specific sets/rubrics to a shared library (Required / Recommended / Opt-in)

The operational stages the toolkit walks a customer through — **Discover, Plan, Generate, Run, Interpret** — are the UX workflow over these steps (Discover=Step 1; Plan=Steps 1,4,5; Generate=Steps 2,3 + Step 8 design; Run=Step 6; Interpret=Steps 7,9 + Step 10 closeout). The **Per-Agent Eval Maturity Model** (5 pillars × 5 levels) in `maturity-model.md` is the outcome scorecard. See the canonical crosswalk in `playbook.md`.

### Architecture-Aware Scoping

| Architecture | What Gets Tested |
|---|---|
| **Prompt-level** (simple Q&A) | Response quality, tone, trust & safety refusals |
| **RAG / Knowledge-grounded** | + retrieval accuracy, grounding/faithfulness, hallucination prevention |
| **Agentic** (tool use, orchestration) | + tool selection, action correctness, error recovery |

### Key Sources

- [Eval Scenario Library](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/evaluation-checklist) — 5 business-problem + 9 capability scenario types
- [Triage & Improvement Playbook](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/evaluation-iterative-framework) — Root cause classification
- [Common Evaluation Approaches](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/architecture/common-evaluation-approaches) — Echo, Historical Replay, Synthesized Personas
- [Eval Guidance Kit](https://aka.ms/EvalGuidanceKit) — Editable checklists and templates

## Scripts

- `skills/eval-guide/scripts/eval-runner.js` — Runs eval test sets against a live Copilot Studio agent via DirectLine API, scores responses with an LLM judge. Usage: `node eval-runner.js --token-endpoint "<URL>" --csv-dir <dir>`

## Output Formats

The toolkit generates:
- **CSV files** — Importable directly into Copilot Studio's Evaluation tab. **Exactly 2 columns: `Question`, `Expected response`** (one row per case). The testing method is assigned per row in Copilot Studio's Evaluate tab after import; all other methodology metadata (set type, category, gate, target, provenance) travels in the companion `.docx` manifest, not the CSV.
- **HTML review pages** — Interactive summaries for plan review, especially the eval-suite planner's workbook companion page. Use these instead of long chat summaries.
- **Report documents** — Eval plans, test case summaries (with the per-case manifest), triage reports
- **Conversation blueprints** — Multi-turn dialogue test structures
