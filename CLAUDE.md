# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **content/plugin repository**, not an application. It ships an AI-agent evaluation toolkit for Copilot Studio in three parallel forms:

- **GitHub Copilot plugin** (CLI, app, cowork) — skills under `skills/*/SKILL.md`, registered via the [Agent Plugins 1.0](https://github.com/agentplugins/agent-plugins-spec) manifest at `plugin.json` (repo root) and indexed by `.github/plugin/marketplace.json`.
- **Claude Code plugin** — the same skills, registered via `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`.
- **GitHub Copilot prompts** — equivalent prompts under `.github/prompts/*.prompt.md`, with always-on instructions in `.github/copilot-instructions.md`.

The two plugin manifests coexist deliberately: Copilot reads the root `plugin.json` (and also accepts `.claude-plugin/`), Claude Code reads only `.claude-plugin/`. Both point at the same `skills/` directory, so there is one copy of every skill.

There is no build, no test runner, no lint step. Changes are validated by running the skills end-to-end against an agent description.

`AGENTS.md` is the master cross-tool instruction file. `README.md` is the canonical user-facing docs. When changing user-visible behavior, keep all three (`README.md`, `AGENTS.md`, `.github/copilot-instructions.md`) in sync — they each describe the same routing table for a different audience.

## High-level architecture

### The 6 skills form a pipeline, not a flat catalog

`/eval-guide` is the orchestrator that walks the customer through the operational workflow (Discover → Plan → Generate → Run → Interpret) over Microsoft's ***Practical Guidance on Agent Evaluation* — 10-step playbook**. The canonical methodology spine is `skills/eval-guide/playbook.md`; every skill points to it rather than restating the methodology. The companion `skills/eval-guide/targeted-eval-sets.md` is the canonical catalog of *which* eval sets come out of Steps 2/3 — treat it the same way. The other 5 skills are extracted stages that can be invoked directly:

| Skill | Role |
|---|---|
| `eval-guide` | Orchestrator. Owns the dashboard workflow and stage transitions. |
| `eval-suite-planner` | Stage 1 standalone. Produces eval-suite workbook plus an interactive HTML review page. |
| `eval-generator` | Stage 2 standalone. Produces test-case CSVs / conversation blueprints. |
| `eval-result-interpreter` | Stage 4 standalone. SHIP/ITERATE/BLOCK verdict from results. |
| `eval-triage-and-improvement` | Stage 4 deep-dive. Interactive remediation. |
| `eval-faq` | Methodology Q&A grounded in Microsoft's eval ecosystem. |

When editing one stage's behavior, check whether the corresponding standalone skill needs the same change. The orchestrator and standalones share methodology but have separate SKILL.md files.

### The dashboard is the review checkpoint

`/eval-guide` uses interactive review surfaces instead of asking "does this look right?" in chat. Stage 1 Plan produces a populated workbook plus `eval-suite-<agent>-<date>-review.html`. Generate and Interpret use the dashboard server flow:

1. Skill writes stage data to `stage-N-data.json`.
2. Skill launches `python skills/eval-guide/dashboard/serve.py --stage <name> --data stage-N-data.json`.
3. `serve.py` injects the JSON into a template (`dashboard/templates/<stage>.html` composed into `base.html`), writes a standalone HTML file next to the data file, and opens it in the browser.
4. The user edits inline; on Confirm/Request Changes, a feedback JSON downloads. They save it next to the data file.
5. `serve.py` detects `<stage>-feedback.json` on disk and exits.
6. Skill reads the feedback file. **No `.docx` or `.csv` is generated until the user confirms via the dashboard.**

Stage names map to file names for served dashboards: `generate` (2), `interpret` (4). Stage 3 (Run) executes tests directly with no dashboard. The planner HTML review page follows `skills/eval-guide/plan-review-page.md`; served dashboard templates live in `dashboard/templates/`, with example stage data in `dashboard/examples/`.

### Eval execution path (Stage 3)

`skills/eval-guide/scripts/eval-runner.js` is a Node script that talks to a live Copilot Studio agent over DirectLine, runs CSV test cases, and uses the Anthropic SDK as an LLM judge for `Compare meaning` / `General quality` methods. It requires `ANTHROPIC_API_KEY` and either `--token-endpoint` or `--directline-secret`.

```bash
node skills/eval-guide/scripts/eval-runner.js --token-endpoint <url> --csv-dir <dir>
```

This is the only stage that requires a running agent.

### Versioning and self-upgrade

Every `/eval-guide` invocation runs `bin/eval-guide-update-check` from the SKILL.md `preamble:` block. The script compares `VERSION` against `serenaxxiee/eval-guide@main` on GitHub and prints `UPGRADE_AVAILABLE <old> <new>` / `JUST_UPGRADED <old> <new>` / nothing. The skill body has explicit handling instructions for each output. State (config, snooze, just-upgraded marker) lives in `~/.eval-guide/`.

`preamble:` is a Claude Code feature. GitHub Copilot ignores it and updates plugins natively via `copilot plugin update eval-guide@eval-guide`, so on Copilot there is no preamble output to handle. The version-check block in `skills/eval-guide/SKILL.md` is client-aware — never emit a `claude ...` command on Copilot or a `copilot ...` command on Claude Code.

When bumping a release, update the version in **all five** places so the clients agree:

| File | Field |
|---|---|
| `VERSION` | whole file |
| `plugin.json` | `version` |
| `.claude-plugin/plugin.json` | `version` |
| `.github/plugin/marketplace.json` | `plugins[0].version` and `metadata.version` |
| `.claude-plugin/marketplace.json` | `plugins[0].version` and `metadata.version` |

Then merge to `main` so the remote update check picks up the new version.

## Cross-client portability rules

The skills run on Copilot (Windows/macOS/Linux) and Claude Code. Two rules keep them working everywhere:

1. **Never hardcode or glob an install path.** Resolve skill-bundled assets (`dashboard/serve.py`, `dashboard/orient-dashboard.html`, `scripts/eval-runner.js`) from the skill's base directory, which the runtime supplies when the skill loads. Copilot installs to `~/.copilot/installed-plugins/...`, Claude Code to `~/.claude/plugins/cache/...`, and a dev checkout lives anywhere. A `ls ~/.claude/... | head -1` style fallback silently resolves to *another client's copy* when both are installed, launching a stale dashboard.
2. **Never use shell-specific launchers.** No `uname`, `xdg-open`, `open`, `head -1`, or `case` — they break on Windows. Open a bundled file in the browser with the cross-platform one-liner `python -c "import sys,pathlib,webbrowser; webbrowser.open(pathlib.Path(sys.argv[1]).resolve().as_uri())" "<path>"`, which already handles drive letters and spaces.

## Conventions to preserve

These are non-obvious invariants enforced across all 6 skills — keep them consistent when editing:

- **CSV format for Copilot Studio import**: exactly 2 columns — `Question`, `Expected response` (one row per case). The testing method is assigned per row in Copilot Studio's Evaluate tab **after** import — it is NOT a CSV column. All other methodology metadata (set type, trust&safety category, gate, target, regression class, provenance) travels in the workbook/manifest, never the CSV. Group test cases by eval set into separate CSV files. (A 3-column `-with-methods` variant may be emitted for human readability only — it is never the import format.)
- **Valid testing methods**: `General quality`, `Compare meaning`, `Text similarity`, `Exact match`, `Keyword match`, `Capability use`, `Custom`. The first five are the Copilot Studio core set; the last two extend it.
- **Stages 0–2 must work without a running agent.** Description-based mode is the default; live-agent mode is an enhancement when the Copilot Studio plugin is also installed. Don't introduce code paths that require live agent connectivity in those stages.
- **Architecture-aware scoping**: planner output should change based on whether the agent is prompt-level / RAG / agentic. Don't generate tool-routing tests for a simple FAQ bot.
- **Three generated-set categories**: common capabilities (`set_type=capability`), trust & safety (`trust_safety`), and agent-specific instruction-following (`instruction_following` — one set per testable instruction from the agent's own instruction block). The canonical catalog, signal-to-dimension mapping, and workbook mapping live in `skills/eval-guide/targeted-eval-sets.md`; don't restate them in skills.
- **Plan and Generate each ask once for the agent's instructions.** One question, offered as a choice, never blocking, never bundled with other questions, and skipped entirely if the conversation / attachments / workbook / Agent Vision already carry the instruction block. If the customer skips, generate the first two categories and state the gap — never invent instructions.
- **The planning workbook's `Dropdown Lists` tab stays untouched.** It has no `Instruction-following` value, so instruction-following rows map onto the existing `Capability` / `Trust & Safety` values with `Set category: Agent-specific instruction-following` recorded in `Notes`. The richer `set_type` label lives only in non-template artifacts (generator data model, `stage-2-data.json`, `.docx` manifest, CSV filename).
- **Every eval plan must include at least one adversarial / safety scenario.**
- **Explain reasoning, don't just produce artifacts.** The skills are pitched as enablement accelerators — the user should learn the methodology, not just receive output. Calibrate verbosity accordingly.
- **The Per-Agent Eval Maturity Model** (5 pillars × 5 levels, L100→L500) is the outcome scorecard (`maturity-model.md`), mapped onto the playbook: P1=Step 1, P2=Steps 2–5, P3=Steps 6+8, P4=Steps 7+9, P5=Step 8; Step 10 is the org-wide bridge. A `/eval-guide` session moves Pillars 1, 2, and 4 from L100 to L300, and Pillars 3 and 5 to L200 (via the rerun-protocol and baseline-comparison starter artifacts). Keep that scoping consistent.
