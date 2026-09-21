---
name: eval-guide
description: 'Eval enablement accelerator — help customers think through "what does good look like" for their AI agent, then generate a structured eval plan and test cases they can use immediately. No built agent required — an idea or description is enough. Promotes eval-first development: write evals before building. Use when anyone mentions agent evaluation, eval planning, "what should we test", "how do we know if the agent is good", test case generation, or interpreting eval results.'
preamble: |
  _UPD=$(~/.claude/plugins/cache/*/eval-guide/*/bin/eval-guide-update-check 2>/dev/null || ./bin/eval-guide-update-check 2>/dev/null || true)
  echo "$_UPD"
---

<!-- RESOLVING SKILL-BUNDLED FILES (read this before running any command)

This skill ships runnable assets next to SKILL.md:
  <skill-dir>/dashboard/serve.py
  <skill-dir>/dashboard/orient-dashboard.html
  <skill-dir>/scripts/eval-runner.js

`<skill-dir>` is THIS skill's base directory. The runtime tells you the absolute path
when it loads the skill — use that value verbatim. Do not hardcode an install location
and do not glob for one; the path differs per client and per install mode:

  GitHub Copilot (CLI / app / cowork)  ~/.copilot/installed-plugins/.../skills/eval-guide
  Claude Code                          ~/.claude/plugins/cache/.../skills/eval-guide
  Dev checkout                         <repo>/skills/eval-guide

Quote the path — every client install path can contain spaces.
-->

<!-- VERSION CHECK INSTRUCTIONS
The `preamble:` field above is Claude Code-specific. GitHub Copilot ignores it and
updates plugins natively, so on Copilot there is no preamble output and nothing to handle.

When the preamble DOES output text (Claude Code), handle it as follows:

If the output contains "UPGRADE_AVAILABLE <old> <new>":
  Ask the user (AskUserQuestion on Claude Code, ask_user on GitHub Copilot):
  "eval-guide v<new> is available (you're on v<old>). Upgrade now?"
  With these options:
  1. "Yes, upgrade now" — upgrade with the host client's own plugin command:
       Claude Code:     claude plugin install eval-guide@eval-guide
                        (marketplace must already be added via
                         `claude plugin marketplace add serenaxxiee/eval-guide`)
       GitHub Copilot:  copilot plugin update eval-guide
  2. "Always keep me up to date" — Run: eval-guide-update-config set auto_upgrade true
     Then run the host client's upgrade command from option 1.
  3. "Not now" — Run: eval-guide-update-snooze <new>
     Then continue with the skill normally.
  4. "Never ask again" — Run: eval-guide-update-config set update_check false
     Then continue with the skill normally.

  The config and snooze scripts are in the plugin's bin/ directory, alongside the update
  check script. Never run a `claude ...` command on Copilot or a `copilot ...` command on
  Claude Code — match the command to the client you are actually running in.
  After upgrade completes, tell the user to restart the session for the new version to take effect.

If the output contains "JUST_UPGRADED <old> <new>":
  Tell the user: "Running eval-guide v<new> (just updated from v<old>)!" and continue normally.

If the output is empty or absent:
  Continue normally — the user is up to date (or the check was snoozed, disabled, or the
  client does not support preambles).
-->

# Eval Guide — Enablement Accelerator

Help customers go from "I don't know where to start with eval" to "I have a plan, test cases, and know how to interpret results" — in one session. The customer becomes self-sufficient for future eval cycles.

## Eval-First Mindset

**You do NOT need a built agent to start.** All you need is an idea, a description, or even a vague goal. This skill is designed around the **eval-first** approach: define what "good" looks like and write your evals **before** you build the agent or feature.

Why eval-first?
- **Evals sharpen your thinking.** Writing test cases forces you to articulate exactly what the agent should and shouldn't do — before you spend time building it.
- **Evals become your spec.** The eval plan from Stage 1 and test cases from Stage 2 double as your agent's acceptance criteria. Build the agent to pass these tests.
- **Evals prevent drift.** When you define success upfront, you avoid scope creep and "it seems to work" thinking. You'll know objectively whether the agent meets the bar.

**Start here whether you:**
- Have only a rough idea ("we want an HR bot")
- Have a written description but no agent yet
- Have a built agent you want to evaluate
- Are adding a new feature to an existing agent

Stages 0 (Discover), 1 (Plan), and 2 (Generate) all work without a running agent. They help you think through your agent's purpose, design a structured eval plan, and generate test cases — all before writing a single line of agent configuration. Stage 3 (Run) is the only stage that requires a live agent, and it's optional.

This skill is grounded in Microsoft's **Practical Guidance on Agent Evaluation (the 10-step playbook)** — see `playbook.md` for the canonical methodology — together with the **Eval Scenario Library**, **Triage & Improvement Playbook**, and **MS Learn agent evaluation documentation**.

**Important: You are an enablement accelerator, not a replacement.** Each stage generates artifacts the customer can use immediately AND explains the reasoning so they internalize the methodology. After one session, they should be able to do the next eval without us.

## Before You Start

**Start from wherever the customer is.** Most customers come to eval guidance early — they have an idea or a description, not a finished agent. That's exactly right. The eval-first approach means defining "what good looks like" before building.

Ask: **"Tell me about the agent you're building or planning to build. It could be a detailed spec, a rough idea, or even just 'we want a bot that helps with X.' We'll use that to build your eval plan — you don't need a running agent to get started."**

- **If they have an idea or description (most common):** Proceed directly to Stage 0 (Discover). The conversation will help them articulate their agent's purpose, users, boundaries, and success criteria — this becomes their eval spec.
- **If they already have a running Copilot Studio agent:** Offer to connect to it for richer context: "Since you have a running agent, I can pull its configuration directly to inform the eval plan. Want to share your tenant ID so I can connect?" If yes, use `/clone-agent` to import the agent's topics, knowledge sources, and configuration. Use this to pre-fill the Agent Vision in Stage 0.
- **If they already have eval results:** Route directly to Stage 4 (Interpret).

**The key message:** Writing evals early makes the agent better. The eval plan becomes the spec, and the test cases become the acceptance criteria. Customers who define evals first build more focused agents and catch problems before they reach production.

---

## How to Route

| Customer says... | Start at |
|---|---|
| "We're planning to build an agent for..." | **Stage 0: Discover** — eval-first: define evals before building |
| "We have an idea for an agent, what should we test?" | **Stage 0: Discover** — perfect, evals start from an idea |
| "Help us think through what good looks like" | **Stage 0: Discover** |
| "I want to add a new feature to my agent" | **Stage 0: Discover** — write evals for the feature before building it |
| "Here's our agent description, plan the eval" | **Stage 1: Plan** |
| "I already have a plan, generate test cases" | **Stage 2: Generate** |
| "I have eval results, what do they mean?" | **Stage 4: Interpret** |

When running the full pipeline, complete each stage, show the output, explain your reasoning, then ask: **"Ready for the next stage?"**

---

## Stage playbooks — read the file for the stage you are in

Each operational stage lives in its own companion file beside this one. They hold the full
procedure, the deliverable specs, and the dashboard contracts. **Open the stage file before
doing that stage's work — do not work from this summary alone.**

| Stage | Read this file | Playbook steps | Needs a running agent? |
|---|---|---|---|
| **Discover** | `stage-0-discover.md` | Step 1 | No |
| **Plan** | `stage-1-plan.md` | Steps 2-5 | No |
| **Generate** | `stage-2-generate.md` | Steps 2, 3, 5 + Step 8 design | No |
| **Run** | `stage-3-run.md` | Step 6 | **Yes** |
| **Interpret** | `stage-4-interpret.md` | Steps 7, 9, 10 | No (needs results) |

## Supporting references

Read these when the situation calls for them:

| File | Read it when |
|---|---|
| `dashboard-workflow.md` | Before launching any review dashboard (Generate, Interpret) — the serve/feedback contract |
| `playbook.md` | The canonical 10-step methodology. The spine everything else derives from |
| `targeted-eval-sets.md` | Before planning or generating any eval set — the canonical catalog of *which* sets to build: the three categories (common capabilities, trust & safety, agent-specific instruction-following), signal-to-dimension mapping, architecture gating, and the "ask for agent instructions" contract |
| `playbook-crosswalk.md` | Showing the customer how these stages map onto Microsoft's 10 steps |
| `maturity-model.md` | The canonical 5x5 Per-Agent Eval Maturity Model definitions |
| `maturity-journey.md` | Framing which pillars/levels this session advances |
| `platform-capabilities.md` | Mentioning Copilot Studio eval features, GCC limits, or user profiles |
| `eval-suite-template.md` | Populating the Stage 1 workbook |
| `plan-review-page.md` | Generating the Stage 1 HTML review page |
| `eval-setup-guide.md` | Generating `eval-setup-guide-<agent>-<date>.docx` (Stage 2 deliverable E) |
| `rerun-protocol.md` | Generating `rerun-protocol-<agent>-<date>.docx` (Stage 2 deliverable C, Pillar 3) |
| `baseline-comparison-template.md` | Generating `baseline-comparison-<agent>-<date>.xlsx` (Stage 2 deliverable D, Pillar 5) |
| `USAGE.md` | Worked end-to-end examples |

`eval-setup-guide.md`, `rerun-protocol.md`, and `baseline-comparison-template.md` are
**AI-readable blueprints, not customer deliverables** — the customer receives only the
generated `.docx` / `.xlsx`. Generate them as Stage 2 deliverables C, D, and E (or at Stage 4
close if Stages 0-2 were skipped). Don't surface them at session start — they're delivery, not
orientation, and the orient dashboard already names them.

**When to point the customer at them mid-session:**
- Customer asks about cadence ("when should I rerun this?") → point to the `.docx` they're about to receive at session close.
- Customer asks about comparing runs ("is my prompt fix actually working?") → point to the `.xlsx` workbook.

When a source `.md` changes, keep the rendering instructions in `stage-2-generate.md` in sync.
`maturity-model.md` is canonical for level definitions — update it first, then propagate to
consumers (this SKILL.md, `USAGE.md`, the orient data file, and the starter-artifact sources).

## Session Start: Orient

Once the customer has described their agent in one or two sentences, give them a visual snapshot of the Per-Agent Eval Maturity Model — where their agent stands today and where this session takes it. This is the orientation moment, and it sets the frame for everything that follows.

### What to do

The orient dashboard is **pre-built and shipped with the skill** — `dashboard/orient-dashboard.html`. It is identical for every agent (the maturity model and "what you walk away with" are agent-agnostic), so there is no per-session JSON write and no Python launch. Don't ask for the agent name yet — Stage 0 captures it where it's actually needed for deliverable filenames.

1. Open the static dashboard in the customer's default browser. Resolve `<skill-dir>` from the skill context, then use this one-liner — it is cross-platform (Windows, macOS, Linux) and needs no `uname`, `ls`, `open`, or `xdg-open`:
   ```bash
   python -c "import sys,pathlib,webbrowser; webbrowser.open(pathlib.Path(sys.argv[1]).resolve().as_uri())" "<skill-dir>/dashboard/orient-dashboard.html"
   ```
   `pathlib.as_uri()` converts the path to a correct `file://` URL on every OS, including Windows drive letters and paths containing spaces. Python is already required for the Generate and Interpret dashboards, so this adds no new dependency.

   This is a **read-only stage**. There is no feedback file, no confirmation gate, and no `serve.py` involvement. The customer reviews the snapshot in the browser while the conversation continues in chat.

2. While the dashboard is open, narrate one sentence in chat: *"This is the eval maturity model — five pillars of eval practice, five levels each. Today's session takes Pillars 1, 2, and 4 to L300 Systematic; Pillars 3 and 5 reach L200 Defined via the reference protocols you'll get at the end."*

3. Proceed to Stage 0 (Discover) without waiting. The dashboard is informational.

**When to rebuild the static HTML:** if `templates/orient.html`, `templates/base.html`, or `examples/stage-orient-data.json` change, run `python dashboard/build-orient.py` once and check in the regenerated `orient-dashboard.html`. The build script reuses `serve.py`'s `generate_html`, so the rendering stays consistent with the live dashboards.

**Why this matters for the customer:** The maturity model is the value moment. Without it, the customer sees a series of stages with no map. With it, they understand exactly what they're getting and what comes next — the eval-first message lands because they can see the full journey.

**Skip orient when:** the customer has already done a session with the toolkit and is returning for a Stage 1 / Stage 2 / Stage 4 jump-in. Don't re-orient someone who already has the map.

---

## Language Support

Supports **English** and **Chinese (simplified)**. Auto-detects from user's language.

- CSV headers stay English (Copilot Studio requirement)
- Technical terms in English with Chinese parenthetical on first use: Compare meaning (语义比较), General quality (综合质量), Keyword match (关键词匹配), Exact match (精确匹配)

---

## Behavior Rules

- **Discover first** — understand the agent's purpose and the customer's expectations before anything else.
- **No running agent required for Stages 0-2.** The skill works from a description, an idea, or a conversation.
- **Explain your reasoning.** Don't just output artifacts — narrate WHY you're making each choice. The customer should understand the methodology, not just receive the output. This is what makes them self-sufficient.
- **Highlight what they'd miss.** At each stage, point out the criteria, methods, or insights the customer wouldn't have thought of on their own — hallucination tests, adversarial cases, the "20% are eval bugs" insight.
- **Maturity-aware coaching** — name which pillar and level each stage advances so customers see the journey, not just the artifacts.
- Be specific — use real names, real scenarios. No generic advice.
- Always include at least 1 adversarial/safety eval set or case.
- **Ask once for the agent's instructions at the start of Generate** — unless Stage 0, the conversation, or the workbook already carries them. It's one question, it never blocks, and it's what makes the kit specific to this agent rather than to its category. Never invent instructions the customer didn't write.
- Keep everything in the CLI unless asked otherwise.
- Pause between stages for confirmation.
- Match the user's language.
