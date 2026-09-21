# Common Targeted Eval Sets — Canonical Generation Catalog

This is the **source of truth for *which* eval sets to generate** for a given agent. `playbook.md` says *what* eval work to do and in what order (Steps 2 and 3); this file says *which concrete sets come out of those steps*. Skills point here instead of restating the catalog.

Consumers: `eval-suite-planner` (registry rows), `eval-generator` (test-case sets), `eval-guide` Stage 1 Plan and Stage 2 Generate.

---

## The three categories

Generated sets fall into three categories. The first two are largely predictable from the agent's profile and are where generation does most of its work. The third is specific to what *this* agent was told to do, and only exists if the customer supplies the agent's instructions.

| # | Category | `set_type` | What's included | Predictable from profile? |
|---|---|---|---|---|
| 1 | **Common capabilities** | `capability` | The quality dimensions most document-grounded agents share — accuracy, groundedness, format, reasoning. Generated from the agent's own knowledge sources, so the cases are real questions with real answers. | Yes |
| 2 | **Trust & safety** | `trust_safety` | What the agent must never do. Largely agent-agnostic, which makes these the sets that carry across agents with the least editing — and the strongest candidates for the Step 10 shared library. | Yes |
| 3 | **Agent-specific instruction-following** | `instruction_following` | Behaviors this agent's instructions require that no generic taxonomy would predict — for example, asking a follow-up question in a given scenario. Derived from the agent's own instructions, one set per important instruction that is testable. | **No — requires the agent's instructions** |

Categories 1 and 2 remain first-class and separate; do not collapse one into the other. Category 3 is a third first-class group in the generator's data model, but see [Workbook mapping](#workbook-mapping) — the planning workbook's controlled vocabulary has only `Capability` and `Trust & Safety`, and must not be edited.

---

## Category 1 — Common capabilities

Eight signals. Each is a candidate set; generate only the ones the agent's architecture actually supports (see [Architecture gating](#architecture-gating)). The canonical `capability_dimension` column is the vocabulary used everywhere else in the toolkit — several signals intentionally roll up into the same dimension, in which case generate them as separate **criteria** inside one set rather than inventing a new dimension.

| Signal | Diagnostic signal — what a pass proves | Canonical `capability_dimension` | Workbook `Dimension tested` |
|---|---|---|---|
| **Answer / extraction accuracy** | Numbers and facts in the answer match ground truth | `accuracy_correctness` | `Accuracy / correctness` |
| **Groundedness & citation** | Claims are supported by, and cite, the retrieved source | `faithfulness_groundedness` | `Faithfulness / groundedness` |
| **Context awareness** | The answer is tailored to the user's own context, not a generic restatement of the source | `relevancy` | `Relevancy` |
| **Relevancy / on-topic** | On-topic, complete, responsive answers | `relevancy` | `Relevancy` |
| **Format adherence** | Obeys the output structure specified in the instructions | `style_tone` | `Style & tone` |
| **Multi-document reasoning** | Synthesizes correctly across multiple sources | `reasoning_tool_use` | `Reasoning & tool use` |
| **Style & tone** | Professional, on-brand, appropriately concise voice | `style_tone` | `Style & tone` |
| **Tool-use correctness** | Right tool, right parameters, right sequence | `reasoning_tool_use` | `Reasoning & tool use` |

**Generate cases from the agent's own knowledge sources** whenever they are named or attached. A question drawn from a real policy page with a real answer is worth several synthetic ones. When sources are not available, generate plausible cases and wrap every factual span in `[VERIFY: …]`.

**Format adherence is worth calling out explicitly.** It is the signal customers most often forget, and it is cheap to test: if the instructions say "answer in three bullets with a citation line," that is a testable contract. Where the required format comes from an explicit instruction, prefer a Category 3 instruction-following set so the failure points at the instruction that was ignored.

---

## Category 2 — Common trust & safety

Each set carries exactly one `category`. The first three below come from the common catalog; `guardrails` and `prompt_injection` complete the toolkit's T&S vocabulary and are added by risk tier. The common catalog's fourth entry — *hallucination* — is deliberately **not** in this table; see the note beneath it.

| Signal | Diagnostic signal — what a pass proves | `category` | Workbook `Dimension tested` |
|---|---|---|---|
| **Out-of-scope handling** | Declines, or says it doesn't know, when the answer isn't in the sources — instead of fabricating | `out_of_scope` | `Out-of-scope handling` |
| **Sensitive-data / PII** | Does not surface PII or confidential data, or over-disclose beyond scope | `sensitive_data` | `Sensitive-data handling` |
| **Compliance-scope behavior** | Avoids definitive legal, HR, or medical advice and routes to a human | `compliance` | `Compliance-specific` |
| **Guardrails** | Refuses harmful, illegal, or policy-violating requests | `guardrails` | `Guardrails` |
| **Prompt injection / jailbreak** | Holds its instructions under adversarial input | `prompt_injection` | `Prompt injection / jailbreak` |

### Note — where hallucination lives

Source catalogs sometimes file "hallucination" — *explicitly probing questions absent from the grounding data to catch invented answers* — under trust & safety. **This toolkit does not.** Hallucination is a **faithfulness/groundedness capability failure** and is caught there (`playbook.md` Step 2). Generate those absent-data probes as a `capability` set on `faithfulness_groundedness`, not as a T&S set. Keep the split clean:

- **Capability / `faithfulness_groundedness`** — absent-data probes and unanswerable questions that catch *invented* answers. This is where the hallucination cases go.
- **Trust & safety / `out_of_scope`** — requests that are outside the agent's remit, where the required behavior is declining and routing, not answering well.

The two use similar-looking questions and grade different things. A set that fabricates a plausible answer fails faithfulness; a set that answers a question it should have refused fails out-of-scope.

At least one trust & safety set is mandatory in every generated kit, regardless of risk tier.

---

## Category 3 — Agent-specific instruction-following

**One set per testable instruction.** Read the agent's instructions and turn each enforceable behavior into its own small set, so a failure points at the instruction that was ignored. This is the category a generic taxonomy cannot produce, and it is usually the highest-signal part of the kit.

| Example instruction | What the set checks |
|---|---|
| *"Ask a clarifying question when the request is ambiguous."* | The agent asks rather than guessing when a required detail is missing |
| *"Always cite the policy section you used."* | Every answer carries a citation, in the required form |
| *"Hand off to a human for anything payroll-related."* | The agent routes instead of answering, on every payroll trigger |

### Testability filter

Mine the instructions and keep only instructions that pass all three checks:

1. **Observable in a single response or a bounded conversation** — the behavior shows up in output a grader can read.
2. **Has a trigger you can write a question for** — you can construct an input that *should* fire the behavior.
3. **Has a discriminating negative** — you can construct an input where the behavior should *not* fire, so the set catches over-application as well as omission.

Drop instructions that fail the filter (persona flavor text, aspirational language, internal implementation notes) and say which ones you dropped and why. Don't silently discard them.

### Set shape

Each instruction-following set is small and sharp:

- **2–4 cases per instruction.** At least one positive trigger, at least one near-miss or negative control where the behavior should not fire.
- **Method:** usually `General quality` or `Custom`, with the instruction restated as the pass condition. Use `Keyword match` when the instruction mandates a literal string (a URL, a team name, a citation format).
- **Quote the instruction verbatim** in the set's `source_instruction` field and in the criterion statement. The whole point is traceability from failure back to instruction.
- **Gate type:** usually a soft target, promoted to a hard gate when the instruction is a routing or refusal obligation (*"hand off anything payroll-related"* is effectively a guardrail).

### Overlap rule

Some instructions restate a common capability or a trust & safety behavior. When that happens, **the instruction-following set wins** — generate one set, tagged `instruction_following`, and note the dimension it also covers in the set's `also_covers` field. Do not generate the same behavior twice under two categories.

---

## Asking for agent instructions

Category 3 cannot be generated without the agent's instructions, so **ask for them once, explicitly, before generating.** Ask early enough that the answer changes the output, and never block on it.

**When to ask:** immediately after input mode is detected and before any test case is written.

**How to ask** — one question, offered as a choice, not an open-ended interview:

> *"Want to paste your agent's instructions (the system prompt / instruction block from Copilot Studio)? I'll use them to generate a third category of eval sets — one set per testable instruction, so a failure points at the exact instruction the agent ignored. Without them I'll still generate common capability and trust & safety sets, which cover most of the kit."*
>
> Options: **Paste the instructions** · **Point me at a file or attachment** · **Skip — generate common sets only**

**If instructions are supplied:** confirm what you mined before generating — list the testable instructions you extracted and the ones you dropped with reasons. Then generate all three categories.

**If the customer skips:** generate Categories 1 and 2, and state plainly what they are missing — *"Generated common capability and trust & safety sets. No instruction-following sets, because I don't have your agent's instructions — those are the ones that catch behaviors specific to how you told this agent to act. Re-run with your instruction block whenever you want them."* Record the gap so the manifest and the human-review checklist both show it.

**Don't ask twice.** If the conversation, attachments, workbook, or Agent Vision already contain the instruction block, use it and say so rather than asking.

**Don't ask for anything else in the same breath.** This is one question, not a questionnaire.

---

## Architecture gating

Generate only the sets the agent's architecture can actually exercise. Do not generate tool-routing tests for a simple FAQ bot.

| Architecture | Category 1 signals to generate | Category 2 | Category 3 |
|---|---|---|---|
| **Prompt-level** (simple Q&A, no retrieval) | Relevancy, style & tone, format adherence | Out-of-scope, guardrails; add others by risk tier | All testable instructions |
| **RAG / knowledge-grounded** | + answer/extraction accuracy, groundedness & citation, context awareness, multi-document reasoning | + sensitive-data, compliance as applicable | All testable instructions |
| **Agentic** (tools, connectors, actions) | + tool-use correctness | + prompt injection; sensitive-data usually mandatory | All testable instructions, especially routing and confirmation obligations |

---

## Workbook mapping

The planning workbook's `Dropdown Lists` tab is a controlled vocabulary and **must not be edited** (`eval-suite-template.md`). It has no `Instruction-following` value. Map instead:

- **`Category`** — choose `Capability` for instruction-following sets, unless the instruction is a refusal or routing obligation, in which case choose `Trust & Safety`.
- **`Dimension tested`** — the closest existing dropdown value, using the mapping columns in the tables above.
- **`Eval Set Name`** — lead with the instruction, e.g. `Instruction: cite the policy section used`.
- **`Purpose / diagnostic signal`** — quote the instruction verbatim and name what a failure diagnoses.
- **`Notes`** — record `Set category: Agent-specific instruction-following` plus the source instruction, so the category survives the round-trip into `eval-generator` even though the registry has no column for it.

The richer `set_type: instruction_following` label is carried in the generator's internal data model, `stage-2-data.json`, the `.docx` manifest, and the CSV filename — none of which are template-bound.

---

## Naming

One import CSV per set, named by category:

| Category | Filename pattern |
|---|---|
| Common capabilities | `eval-capability-<dimension-slug>-<YYYY-MM-DD>-for-import.csv` |
| Trust & safety | `eval-trust-safety-<category-slug>-<YYYY-MM-DD>-for-import.csv` |
| Instruction-following | `eval-instruction-following-<instruction-slug>-<YYYY-MM-DD>-for-import.csv` |

Every import CSV is the standard 2-column `Question`, `Expected response` format regardless of category. Category, dimension, gate, target, regression class, and source instruction travel in the `.docx` manifest, never in the CSV.

---

## Sync rule

When this catalog changes, update this file first, then propagate to:

- `skills/eval-generator/SKILL.md` and `.github/prompts/eval-generator.prompt.md`
- `skills/eval-suite-planner/SKILL.md` and `.github/prompts/eval-suite-planner.prompt.md`
- `skills/eval-guide/SKILL.md` (Stage 1 Plan, Stage 2 Generate) and `.github/prompts/eval-guide.prompt.md`

Do not restate the catalog in those files — reference this one and keep only the behavior each skill needs.
