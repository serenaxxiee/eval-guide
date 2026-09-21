# Eval Guide — Copilot Instructions

This repository is an AI agent evaluation toolkit for Copilot Studio. It provides structured methodologies for planning, generating, running, and interpreting AI agent evaluations.

## Quick Reference

- **Prompt files** are in `.github/prompts/` — attach the relevant one when helping with eval tasks
- **Full methodology** is the 10-step playbook in `skills/eval-guide/playbook.md` (canonical spine); `AGENTS.md` at the repo root is the cross-tool summary
- **Which eval sets to generate** is catalogued in `skills/eval-guide/targeted-eval-sets.md` — three categories (common capabilities, trust & safety, agent-specific instruction-following) plus the "ask for agent instructions" contract
- **Eval runner script** is at `skills/eval-guide/scripts/eval-runner.js`

## Skill Routing

| Task | Prompt File |
|---|---|
| Full eval lifecycle | `eval-guide.prompt.md` |
| Create eval plan | `eval-suite-planner.prompt.md` |
| Generate test cases | `eval-generator.prompt.md` |
| Interpret results | `eval-result-interpreter.prompt.md` |
| Triage failures | `eval-triage-and-improvement.prompt.md` |
| Methodology Q&A | `eval-faq.prompt.md` |

## Key Conventions

- CSV format for Copilot Studio import: exactly 2 columns — `Question`, `Expected response`. The testing method is assigned per row in the Copilot Studio Evaluate tab after import (not a CSV column); other methodology metadata lives in the companion `.docx` manifest
- Valid test methods (assigned in the UI): `General quality`, `Compare meaning`, `Text similarity`, `Exact match`, `Keyword match` (core five), plus `Capability use` and `Custom` (extensions)
- Always explain reasoning — users should learn the methodology, not just receive artifacts
- For planner output, prefer the interactive HTML review page over long chat summaries; keep chat to artifact paths and blockers
- Include at least 1 adversarial/safety eval set or case in every eval plan
- Generated sets fall into three categories: common capabilities, trust & safety, and agent-specific instruction-following (one set per testable instruction). Plan and Generate each **ask once** for the agent's instructions so the third category can be built — one question, never blocking; if skipped, generate the first two and state the gap. Never invent instructions
- Group test cases by eval set into separate CSV files
- Stages 0-2 (Discover, Plan, Generate) work without a running agent
