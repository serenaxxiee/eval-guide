---
name: eval-faq
description: Answers AI agent evaluation methodology questions with practical, opinionated guidance grounded primarily in Microsoft's agent evaluation ecosystem (MS Learn, Eval Scenario Library, Triage & Improvement Playbook, Eval Guidance Kit) supplemented by select industry sources.
---

## Purpose

Answer any question about eval methodology, grader types, dataset design, criteria writing, non-determinism, tool-call evaluation, multi-turn agent evaluation, eval tooling, capability vs. regression evals, and interpreting results — specifically in the context of AI agent evaluation. The primary methodology is `skills/eval-guide/playbook.md`: **Practical Guidance on Agent Evaluation: a 10-step playbook**. Microsoft's agent evaluation documentation (MS Learn pages, the Eval Scenario Library, the Triage & Improvement Playbook, and the Eval Guidance Kit) remains the authoritative supporting source set for Copilot Studio mechanics and reference patterns, supplemented by select industry sources for topics Microsoft does not cover deeply.

## Instructions

When invoked as `/eval-faq <question>`, follow this process exactly:

### Step 1 — Fetch authoritative context before answering

Use this topic-to-URL routing table to decide what to fetch. Fetch FIRST, then answer. Fetch only the URL(s) that match the question topic — do not fetch all URLs every time.

| Question topic | Fetch this URL | Section to extract | Notes |
|---|---|---|---|
| Scenario types, business-problem vs capability scenarios, what cases to write, dataset structure | `https://github.com/microsoft/ai-agent-eval-scenario-library` | Business-Problem scenarios, Capability scenarios, eval-set-template | 5 business-problem + 9 capability scenario types |
| Quality signals, policy accuracy, source attribution, personalization, action enablement, privacy | `https://github.com/microsoft/ai-agent-eval-scenario-library` | Quality signals section and method mapping tables | Quality signal to evaluation method mapping |
| Red-teaming, adversarial testing, attack surface reduction, XPIA, encoding attacks, ASR metrics | `https://github.com/microsoft/ai-agent-eval-scenario-library` | Red-teaming section: Probe-Measure-Harden framework | Red-team ASR thresholds: <2% harmful, <1% PII, <5% jailbreak |
| Evaluation method selection, keyword match vs compare meaning vs general quality | `https://github.com/microsoft/ai-agent-eval-scenario-library` | resources/evaluation-method-selection-guide.md | 4 evaluation methods with selection criteria |
| Eval generation, writing eval cases from a prompt template, synthesizing test sets | `https://github.com/microsoft/ai-agent-eval-scenario-library` | resources/eval-generation-prompt.md | Template for generating eval cases |
| Agent profile template, defining agent scope for eval | `https://github.com/microsoft/ai-agent-eval-scenario-library` | resources/agent-profile-template.yaml | Agent profile definition for scoping evals |
| Score interpretation, what scores mean, risk tier-based thresholds, hard/soft gates, readiness decisions, SHIP/ITERATE/BLOCK | `https://github.com/microsoft/triage-and-improvement-playbook` | Layer 1: Score Interpretation, readiness decision tree | Supporting source for Step 4/6/7 readiness decisions |
| Failure triage, debugging eval failures, root cause analysis, diagnostic questions | `https://github.com/microsoft/triage-and-improvement-playbook` | Layer 2: Failure Triage, 26 diagnostic questions | 5-question eval verification, 7 eval setup failure sub-types |
| Remediation, fixing failures, instruction budget, actions per failure pattern | `https://github.com/microsoft/triage-and-improvement-playbook` | Layer 3: Remediation Mapping | Actions mapped to failure patterns |
| Pattern analysis, cross-signal patterns, trend analysis, concentration analysis | `https://github.com/microsoft/triage-and-improvement-playbook` | Layer 4: Pattern Analysis | 7 cross-signal patterns, trend analysis |
| Root cause types, eval-setup problem vs agent-quality problem, eval setup issue vs agent config vs platform limitation | `https://github.com/microsoft/triage-and-improvement-playbook` | Root Cause Types section | Supporting taxonomy mapped to Step 7's two root buckets |
| Non-determinism handling, run variance, flaky results | `https://github.com/microsoft/triage-and-improvement-playbook` | Non-determinism section | 3 runs minimum, +/-5% normal, +/-10% investigate |
| 4-stage iterative framework, Define, Set Baseline & Iterate, Systematic Expansion, Operationalize | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/evaluation-iterative-framework` | Full framework — all 4 stages | Supporting MS Learn lifecycle/cadence source under the 10-step playbook |
| Eval checklist, readiness checklist, pre-launch verification | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/evaluation-checklist` | Full checklist | Maps to Eval Guidance Kit documents |
| Grader types, code-based vs LLM-judge vs human graders, common evaluation approaches | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/architecture/common-evaluation-approaches` | Echo, Historical Replay, Synthesized Personas; grader types | 3 approaches + 3 grader categories |
| 7 test methods, General Quality, Compare Meaning, Capability Use, Keyword Match, Text Similarity, Exact Match, Custom | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-overview` | 7 test methods section | General Quality sub-dimensions: Relevance, Groundedness, Completeness, Abstention |
| Test set creation, building eval datasets in Copilot Studio | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-create` | Test set creation methods | Generate, import, or manually write test cases |
| Test set editing, user profiles, connections, modifying test methods | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-edit` | Manage user profiles and connections, edit test methods | Multi-profile eval for simulating different users; GCC limitations |
| Running evals, viewing results, test results interpretation | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-results` | Run tests and view results | 89-day result retention; export results immediately |
| Agent evaluation overview, why use automated testing, test chat vs eval | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-intro` | About agent evaluation | GCC limitations: no user profiles, no Text similarity method |
| Rubric refinement workflow, aligning AI grading with human judgment | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/kit-rubrics-refinement-workflow` | 8-step workflow: Run, Review, Grade, Refine, Save, Re-run, Repeat | Alignment matrix, Standard vs Full refinement views, example marking |
| Rubric best practices, tips for rubric refinement | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/kit-rubrics-best-practices` | Best practices for refinement | Quality over quantity for examples; don't chase 100% alignment |
| Rubric reference guide, grade definitions, rubric structure | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/kit-rubrics-reference` | Rubrics reference | Grade scale definitions, rubric components |
| Copilot Studio Kit overview, kit capabilities | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/kit-overview` | Kit overview | Parent page for all Kit features including rubrics |
| 11 scenario validation themes, evaluation frameworks | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/architecture/evaluation-frameworks` | 11 scenario validation themes | |
| Defining eval purpose, what to evaluate, scoping eval | `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/evaluation-define-purpose` | Full page | |
| Eval Guidance Kit, checklist documents, framework PowerPoint | `https://aka.ms/EvalGuidanceKit` | Checklist, Framework, failure-log-template | Resolves to GitHub PowerPnPGuidanceHub |
| pass@k vs pass^k metrics, non-determinism statistics, 0% pass@100 interpretation | `https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents` | pass@k, pass^k, capability evals sections | Supplementary: Microsoft non-determinism guidance is primary |
| Capability vs regression evals, eval-driven development | `https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents` | Capability evals, regression evals sections | Supplementary industry context under the 10-step playbook |
| LLM-as-judge calibration, position bias, verbosity bias, self-enhancement bias | `https://eugeneyan.com/writing/llm-evaluators/` | Biases and calibration sections | Supplementary: bias percentages not in Microsoft sources |
| Critique shadowing, judge prompt design, error analysis methodology | `https://hamel.dev/blog/posts/llm-judge/` | Judge prompt design, calibration | Supplementary: deep LLM judge methodology |
| Eval platforms, tooling comparison, Braintrust, LangSmith | `https://www.braintrust.dev/articles/top-5-platforms-agent-evals-2025` | Platform comparison | Supplementary: lightweight tooling reference |
| Any question not clearly matching above | Fetch `https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/evaluation-overview` as primary source, supplement with relevant knowledge base section | | Default fallback is MS Learn |

**Fetch rules:**
- Always attempt the fetch for rows without "Do NOT fetch." If it fails (404, timeout, irrelevant content), fall back to the knowledge base below and note "Source unavailable at fetch time — answering from knowledge base."
- **Microsoft sources take priority.** When a topic is covered by both Microsoft and external sources, use Microsoft content as the primary answer and external content only as supplementary detail.
- Citation format for Microsoft: "Per Microsoft's Eval Scenario Library:", "Per the Triage Playbook:", "Per MS Learn agent evaluation guidance:"
- Citation format for external: "Additional industry context from [source]:" — always after Microsoft content.
- Never block on a failed fetch. A degraded answer is better than no answer.
- Extract only the section relevant to the question. Do not summarize the whole page.

### Step 2 — Answer using fetched content plus knowledge base

Synthesize the fetched content with the knowledge base below. The 10-step playbook is the methodology spine; Microsoft fetched content supplies supporting details and Copilot Studio specifics, then external sources fill gaps.

**Answer style rules — no exceptions:**
- Answer in 3-5 sentences maximum. No padding, no preamble, no "great question."
- Give opinionated, direct guidance. Never say "it depends" without immediately resolving it with a concrete recommendation.
- Use specific numbers ("start with 20-50 cases", "flag cases with <60% agreement", "run 3 trials per case").
- Ask at most one targeted clarifying question only when the answer would materially change by agent architecture, risk tier, or lifecycle stage; otherwise make a reasonable assumption and answer.
- Cite which source you used at the end of the answer.

---

## Knowledge Base

Use the sections below as your primary reference when fetched content does not cover the question, or to supplement fetched content with additional details.

### Canonical methodology: 10-step playbook

The core methodology is `skills/eval-guide/playbook.md`: **Practical Guidance on Agent Evaluation: a 10-step playbook**. Use the MS Learn pages below as supporting sources, not the spine.

1. **Plan the eval effort** — eval objective, agent-level risk tier using five risk factors, named owner.
2. **Build capability eval sets** — one set per capability dimension: accuracy/correctness, faithfulness/groundedness, relevancy, style/tone, reasoning/tool use. Hallucination is a faithfulness capability failure.
3. **Build trust & safety eval sets** — separate refusal/policy/sensitive-data/prompt-injection/compliance sets.
4. **Define pass-rate targets and gates** — explicit target plus hard gate or soft target per set.
5. **Specify human inputs** — rubrics, ground truths, golden answers, and source-to-ground-truth dependencies.
6. **Run the baseline** — record per-set/case results with timestamp and agent version.
7. **Iterate to diagnose failures** — every failure is either an eval-setup problem or an agent-quality problem; maintain a failure-pattern log.
8. **Regression suite** — partition sets into regression sets and gate-only sets with cadence/alerts.
9. **Optimization loop** — production signals -> clusters -> fix location -> ship -> re-evaluate against the regression suite.
10. **Identify and save reusable assets** — promote reusable sets/rubrics/patterns into Required, Recommended, or Opt-in shared library tiers.

**Supporting MS Learn lifecycle source:** The MS Learn iterative framework is still useful for lifecycle/cadence questions and maps into the playbook, but it is no longer the canonical methodology for this toolkit.

### Risk tier, gates, workbook, and manifest

- **Risk tier** is agent-level and uses five factors: reach, criticality of error, autonomy/blast radius, regulatory exposure, and data sensitivity. It drives targets, gate strictness, required trust & safety coverage, and human-review needs.
- **Hard gate** means must pass before deploy; **soft target** means tracked but non-blocking.
- Prefer the **workbook/manifest** over inference: set type, category, method, gate, target, intended use, cadence, human-review inputs, source dependencies, grader-validation notes, and reusable-asset flags live in the Eval Suite Template workbook and manifest, not in the Copilot Studio import CSV.

### Step 9 optimization loop and Step 10 reusable assets

Step 9 turns production signals into improvements: thumbs-down (highest signal), escalations, manual overrides, support tickets, and qualitative feedback -> cluster -> decide fix location (agent config/retrieval/tools, rubric/expected answer, or new eval cases) -> ship -> re-evaluate against the Step 8 regression suite. A production failure with no matching eval case is a coverage gap, not proof that the prompt is bad.

Step 10 promotes reusable assets into a shared eval library with three tiers: **Required** (org-wide deploy gate), **Recommended** (applies to most agents in a class), and **Opt-in** (borrow when relevant). Good candidates are trust & safety sets, tone/citation/refusal rubrics, failure-pattern templates, and production-derived edge cases.

### Scenario types

Per Microsoft's Eval Scenario Library, scenarios divide into two categories:

**5 Business-Problem scenarios** (test whether the agent solves the real user problem):
- **Information Retrieval** — Agent finds and delivers the right information from knowledge sources.
- **Troubleshooting** — Agent diagnoses and resolves user issues through guided steps.
- **Request Submission** — Agent completes a transactional request on the user's behalf.
- **Process Navigation** — Agent guides users through multi-step workflows.
- **Triage & Routing** — Agent correctly classifies and routes requests to the right handler.

**9 Capability scenarios** (test a specific isolated ability):
- Knowledge Grounding, Tool Invocations, Trigger Routing, Compliance, Trust & Safety / Red-Teaming, Tone, Graceful Failure, Regression.

**Anti-pattern:** Skewing your dataset 80%+ toward happy-path cases. Per the Scenario Library, balance across business-problem and capability scenarios for meaningful coverage. Target roughly 50% happy-path, 30% edge cases, 20% adversarial.

### Source-library quality dimensions

Microsoft's Eval Scenario Library includes five reusable quality dimensions that can inform eval-set design, but the toolkit records them as workbook registry rows rather than treating them as the primary planning artifact:

1. **Policy Accuracy** — Does the agent follow business rules and policies correctly?
2. **Source Attribution** — Does the agent ground claims in retrieved documents and cite them?
3. **Personalization** — Does the agent adapt responses to user context and preferences?
4. **Action Enablement** — Does the agent empower users to take the next step?
5. **Privacy Protection** — Does the agent avoid exposing sensitive information?

Each dimension can map to methods such as Keyword Match, Compare Meaning, Capability Use, or General Quality, but the selected method and governance belongs to the eval set in the workbook registry.

### 7 test methods (Copilot Studio)

Per MS Learn agent evaluation guidance, seven test methods cover different evaluation needs:

1. **General Quality** — LLM-judge evaluation across sub-dimensions: Relevance, Groundedness, Completeness, Abstention. Use for open-ended quality assessment. Target 80-90% pass rate.
2. **Compare Meaning** — Semantic similarity between agent response and expected answer. Use when the meaning matters but exact wording does not.
3. **Capability Use** (labeled "Tool use" in UI) — Validates the agent invoked the correct tools with correct parameters. Use for agentic workflows with tool calls.
4. **Keyword Match** — Checks for presence or absence of specific keywords. Use for compliance, policy adherence, and must-include/must-not-include checks.
5. **Text Similarity** — Lexical/embedding-based similarity scoring. Use when response phrasing matters.
6. **Exact Match** — Strict string equality. Use for classification, routing labels, and structured outputs.
7. **Custom** — Define your own evaluation criteria with evaluation instructions and labeled outcomes. Components: (1) **Evaluation instructions** — a plain-language rubric describing what to check (e.g., "Does the response follow our HR escalation policy?"), (2) **Labels** — named outcomes the judge assigns (e.g., "Compliant" / "Non-Compliant"), each mapped to pass or fail. Works for both single-response and conversation test sets. Use Custom when pass/fail requires judgment that Keyword Match or Compare Meaning cannot capture — compliance checks, tone/brand voice, safety policies, classification accuracy. **CSV import caveat:** Custom test cases cannot be imported via CSV — create them directly in the Copilot Studio evaluation UI.

### Evaluation approaches

Per MS Learn (common-evaluation-approaches), three approaches for generating test interactions:

- **Echo** — Replay exact user inputs and compare outputs to expected results. Simplest; good for regression testing.
- **Historical Replay** — Use real production conversation logs as eval inputs. Best signal for production-realistic coverage.
- **Synthesized Personas** — Generate diverse simulated user personas to create varied test interactions. Best for coverage expansion when production logs are limited.

### Score interpretation and triage (Triage Playbook)

Per the Triage Playbook, score interpretation follows a 4-layer framework:

**Layer 1 — Score Interpretation:** Apply risk tier, workbook-defined gates, grader-validation caveats, and the readiness decision tree:
- **SHIP** — All hard gates pass, soft targets are acceptable, and the owner accepts any residual risk.
- **ITERATE** — Some eval sets miss soft targets or show regressions; targeted fixes needed.
- **BLOCK** — A hard gate fails, especially trust & safety; do not ship regardless of aggregate pass rate.

**Layer 2 — Failure Triage:** When scores are low, run the 5-question eval verification first (is the eval itself correct?) before blaming the agent. Then apply 26 diagnostic questions across 6 domains to identify the root cause. Seven eval setup failure sub-types cover common grader/dataset bugs.

**Layer 3 — Remediation Mapping:** Each failed eval set should map to a specific fix location. Watch for the instruction budget problem — adding instructions to fix one failure pattern can degrade another.

**Layer 4 — Pattern Analysis:** Look for concentration (failures clustered in specific scenario types), cross-signal correlations (7 documented cross-signal patterns), and trends over time.

**Step 7 root buckets:** Every failure is exactly one of: (1) **Eval-setup problem** — the response is acceptable and the eval/ground truth/rubric/method is wrong, or (2) **Agent-quality problem** — the eval caught a real issue. The Triage Playbook's Eval Setup / Agent Configuration / Platform Limitation categories are useful operational subtypes mapped onto those two buckets. Always rule out eval setup first — many early "failures" are grader or dataset bugs, not agent bugs.

### Non-determinism

Per the Triage Playbook: agents are non-deterministic. Run a minimum of 3 trials per case. Score variance of +/-5% across runs is normal. Variance of +/-10% or more requires investigation — either the eval is flaky or the agent has a genuine instability.

**Additional industry context from Anthropic:** pass@k ("succeeded at least once in k runs") vs. pass^k ("succeeded every time in k runs") diverge massively at scale. At k=10 with 70% per-trial success: pass@k is approximately 97%, pass^k is approximately 3%. The same agent looks excellent or catastrophic depending on which metric you report. For customer-facing agents, pass^k is the right question. A 0% pass@100 is almost always a task specification problem, not an agent problem — fix the task definition before blaming the model.

### Red-teaming

Per Microsoft's Eval Scenario Library, red-teaming uses the **Probe-Measure-Harden** framework:

1. **Probe** — Run adversarial attacks including prompt injection, XPIA (cross-prompt injection attacks), encoding attacks, and role-playing exploitation.
2. **Measure** — Track Attack Success Rate (ASR) metrics per category.
3. **Harden** — Fix vulnerabilities, add guardrails, re-probe.

**Red-team thresholds:** ASR <2% for harmful content, <1% for PII leakage, <5% for jailbreak. Integrate red-teaming into CI/CD — point-in-time testing misses regressions from prompt changes and model upgrades.

**Multi-turn adversarial patterns:** Single-turn tests are insufficient for deployed conversational agents. Three attack patterns require multi-turn evaluation: (1) Context manipulation — requests shift gradually across turns, (2) Permission escalation — false admin claims introduced across conversation, (3) Role-playing escalation — fictional framing established early then escalated. Include at least 2-3 multi-turn adversarial scenarios in any eval suite.

### Grader types

Per MS Learn (common-evaluation-approaches), three grader categories:

- **Code-based / deterministic graders** (regex, string matching, JSON schema validation, length checks): Fast, cheap, unambiguous. Run these first. If a deterministic check can answer your question, do not reach for an LLM judge.
- **LLM-judge graders** (LLM judges output against written criteria): Use for quality checks requiring judgment — tone, completeness, factual grounding, relevance. Write criteria in plain language before writing grader code.
- **Human graders**: Slowest and highest quality. Use only for calibration — verifying that automated graders agree with expert humans at least 80% of the time (Cohen's kappa > 0.6).

**Grading hierarchy (cheapest to most expensive):** Run code-based checks first, then LLM judges on passing cases, then human review on a calibration sample. Per the Scenario Library, the 4 evaluation methods (Keyword Match, Compare Meaning, Capability Use, General Quality) map to these grader categories.

**Calibration threshold:** If your LLM judge and a human expert agree on fewer than 80% of cases (kappa < 0.6), your criteria are ambiguous. Rewrite criteria before trusting scores.

### Dataset design

Per the Eval Scenario Library, use the `eval-set-template.md` to structure your dataset. Use the `eval-generation-prompt.md` template to generate cases from an agent profile.

- Start with 20-50 cases for a focused task. Per the Scenario Library, cover all relevant business-problem scenarios before expanding to capability scenarios.
- Use the agent profile template (`agent-profile-template.yaml`) to define scope before writing cases.
- Every production incident should become a dataset case within 24 hours.
- Datasets are living artifacts. A frozen, cadence-run dataset is a regression set; milestone-only trust & safety sets are gate-only sets.
- When pass rate hits 100%, the dataset has saturated — promote to regression suite and write harder cases.

**CSV and scoring conventions:** Copilot Studio import CSVs are exactly two columns: `Question`, `Expected response`. Assign the testing method in the Copilot Studio UI after import; keep set_type, category, method, gate, target, regression class, human-review flag, and source/ground-truth provenance in the manifest (`.docx` report + `stage-N-data.json`). Standardize scoring across the suite; for most agents, binary pass/fail is the correct default.

### Criteria writing

- Criteria must be specific enough that two people reading them independently would agree on pass or fail. Per the Triage Playbook's Layer 2, ambiguous criteria are a top eval setup failure sub-type.
- Bad: "the response is helpful." Good: "the response is under 300 characters and mentions the refund policy by name."
- Write criteria before writing code. If you cannot write a testable criterion, you do not understand what the agent should do.
- **One dimension per score.** Do not combine factuality, tone, and conciseness into a single score. Multi-dimension composite scores hide regressions.
- **Avoid Likert scales (1-5).** Use binary pass/fail. Binary forces clarity. If you must use multi-point, cap at 3: fail / partial / pass.
- **Version your grader prompts.** A grader change produces incomparable scores. Track grader versions alongside dataset versions.

### Eval-driven development

Per the 10-step playbook, evaluation starts at **Step 1 — Plan the eval effort** before the agent is built:

- Write capability and trust & safety eval sets (Steps 2-3) that define the target behavior before the agent can fulfill them.
- Define targets and hard/soft gates (Step 4) before interpreting scores.
- Run the baseline (Step 6) — low scores on new capability sets are expected and useful.
- Iterate via Step 7 until hard gates pass and release criteria are met; then keep Step 8 regression monitoring in place.

**Anti-pattern:** Writing evals after building the feature. That produces evals calibrated to what you built, not what you intended.

### Transcript reading and error analysis

Per Step 7 and the Triage Playbook (Layer 2), never trust a score you have not manually verified. The first question is whether the failure is an eval-setup problem: Is the test set correct? Is the grader measuring the right thing? Is the expected answer actually right? Is the agent getting the right context? Is the eval environment matching production?

**Axial coding process for failure analysis:**
1. Run your eval. Collect all failures.
2. Read each failure. Write a one-sentence label for the root cause.
3. Group labels into 3-5 categories (use the Step 7 buckets first, then the Triage Playbook's diagnostic domains as operational subtypes).
4. Count frequency per category. Sort descending.
5. Fix the highest-frequency category first. Re-run. Repeat.

Per Step 7, always include "eval-setup problem" as a category — many failures in a new eval are grader, rubric, stale ground-truth, or manifest bugs rather than agent-quality problems.

**Additional industry context from Hamel Husain:** The axial coding methodology and "highest ROI activity in AI engineering" framing come from Hamel Husain's error analysis work. His key insight: most practitioners skip categorization and jump to "fix the prompt," missing structural patterns.

### Tool-call evaluation

Per the Eval Scenario Library's Tool Invocations capability scenario and MS Learn's Capability Use test method:

- **Three questions per tool invocation:** (1) Was it the right tool? (2) Were arguments correct and complete? (3) Was the invocation necessary?
- Do not grade tool-call sequences rigidly. Grade outcomes, not paths. If the agent reached the right answer via a different tool sequence, that should pass.
- Unnecessary tool calls are a cost and latency issue in production. Catch them in eval.

### Multi-turn and trajectory evaluation

Per MS Learn's evaluation approaches, multi-turn workflows require conversation-level evaluation, not turn-level:

- **Trajectory scoring:** Evaluate the sequence of steps as a whole. Did the agent take the shortest reasonable path? Did it recover from intermediate errors?
- **Environment state verification:** Ground truth is the state of the external environment, not what the agent claims. A booking agent passes if the reservation exists in the database.
- **Compounding errors:** A mistake at step 2 may not be visible in the final output. Run evals with detailed logging at each step.
- **Stateful interaction evaluation:** A turn-level pass rate of 90% can hide a conversation-level failure rate of 40%.

### Eval for agentic workflows

Per MS Learn's evaluation frameworks (11 scenario validation themes):

- Test each component individually first, then evaluate end-to-end. Component-level failures compound in pipelines.
- Orchestration-level failures are the most common missed failure mode. A pipeline where all components score 95% individually can still fail end-to-end at 40-60%.
- Use simulated environments for eval. Never run evals against production systems.
- Monitor intermediate outputs with validators at each pipeline step.

### Simple Q&A vs. multi-step agent: what changes in eval

The evaluation approach differs significantly based on agent complexity:

| Dimension | Simple Q&A agent | Multi-step / agentic workflow |
|---|---|---|
| **Primary metric** | Response accuracy (Compare Meaning, General Quality) | Task completion — did the end-to-end job get done? |
| **Grading unit** | Single turn: one input, one output | Conversation or trajectory: full sequence of steps |
| **Key eval-set focus** | Grounded answers and policy accuracy | Action enablement, tool invocation, and Q&A correctness |
| **Test method mix** | Heavy on Compare Meaning + General Quality | Add Capability Use for tool calls, Keyword Match for intermediate checkpoints |
| **Failure modes to watch** | Wrong answer, hallucination, refusal | Compounding errors, wrong tool selection, unnecessary steps, partial completion |
| **Edge cases** | Ambiguous queries, out-of-scope questions | Mid-workflow failures, tool timeouts, user corrections mid-conversation |
| **Eval complexity** | Low — deterministic input/output pairs work well | High — must evaluate intermediate steps AND final outcome |

**Practical guidance:**
- **Start with Q&A-style eval even for agentic workflows.** Verify the agent produces correct final answers before evaluating the path it takes. A wrong answer via the right tools is still wrong.
- **Add tool-call eval (Capability Use) only after response quality is stable.** Per the Scenario Library, tool invocation testing checks three things: right tool, right arguments, necessary invocation.
- **Grade outcomes, not paths.** Two valid tool sequences can produce the same correct result. Per the Eval Scenario Library, do not grade tool-call sequences rigidly.
- **Watch for the orchestration gap.** Per MS Learn's evaluation frameworks, components scoring 95% individually can fail 40-60% end-to-end. Always run conversation-level evaluation, not just turn-level.
- **Budget more test cases.** A Q&A agent might need 20-30 cases for meaningful signal. A multi-step workflow with 3+ tools needs 50-100 to cover tool combinations and failure recovery paths.

### Swiss cheese model of eval coverage

No single eval method catches every failure. Per the Eval Scenario Library's 4 evaluation methods and the Triage Playbook's multi-layer approach:

- Code-based graders catch structural failures but miss semantic ones.
- LLM judges catch semantic failures but have systematic biases.
- Human review catches subtle judgment failures but is too slow for full coverage.
- Production monitoring catches real-world distribution failures.
- **Layer all four.** Run deterministic checks first (cheapest), then LLM judges, then human calibration, then production monitoring.

### LLM-as-judge calibration

Per MS Learn's General Quality test method, LLM judges evaluate across sub-dimensions (Relevance, Groundedness, Completeness, Abstention). Calibrate judges against these defined dimensions.

**Additional industry context from Eugene Yan (bias data):**
- **Position bias:** GPT-3.5 biased toward first option 50% of the time; Claude-v1 biased 70%. Mitigate by evaluating both orderings.
- **Self-enhancement bias:** GPT-4 rates own outputs 10% higher; Claude-v1 rates own outputs 25% higher. Never use a model to judge its own outputs.
- **Verbosity bias:** Both models preferred longer responses >90% of the time. Include explicit length-independence instructions in judge prompts.

**Additional industry context from Hamel Husain (critique shadowing):**
When building LLM judges from scratch, use the 7-step Critique Shadowing methodology: (1) Identify one expert, (2) Create diverse dataset, (3) Collect binary pass/fail with written critiques, (4) Fix obvious errors, (5) Build judge prompts iteratively using expert examples, (6) Error analysis on disagreements, (7) Build specialized judges for specific failure modes. Target >90% agreement with domain expert before production use.

### Knowledge grounding (for RAG agents)

Per the Eval Scenario Library's Knowledge Grounding guidance:

- Knowledge grounding score measures whether each factual claim is supported by retrieved context.
- A 75% grounding score means roughly 1 in 4 claims may not be traceable to documents. Set threshold at 90%+ for high-stakes factual tasks.
- Low grounding score almost always means the retrieval step is failing, not the generation step. Fix chunking and retrieval before tuning the prompt.

### Production continuity

Per Step 9 of the 10-step playbook, eval is not a pre-launch gate — it is a continuous optimization loop:

- Integrate evals into CI/CD. Run the full suite on every PR that changes system prompts, tool definitions, or agent behavior.
- Production signals flow from thumbs-down (highest signal), escalations, manual overrides, support tickets, and qualitative feedback into clustered patterns.
- **The optimization loop:** production signals -> clusters -> decide fix location (agent config/retrieval/tools, rubric/expected answer, or new eval coverage) -> ship -> re-evaluate against the Step 8 regression suite.
- Ship with monitoring, not just evals. The eval tells you the agent worked on test cases. Monitoring tells you it works on real user inputs.

**When the agent passes evals but fails in production:** Per the Triage Playbook, this is almost always a distribution mismatch. Pull 20 recent production failures. Check whether any would fail against your current eval dataset. If none would, your dataset needs production cases, not a better prompt.

### Interpreting results

Per the Triage Playbook's readiness decision tree:

- **SHIP**: All hard gates pass and soft-target misses are accepted/documented.
- **ITERATE**: Capability sets miss targets or production patterns need fixes. Use Step 7 failure triage to diagnose.
- **BLOCK**: Any hard gate fails, especially trust & safety, regardless of aggregate pass rate.

Per the Triage Playbook's Layer 4 (Pattern Analysis): look for failure concentration in specific scenario types, cross-signal correlations, and trends over time. When a grader's verdict disagrees with your intuition, investigate — either the grader is wrong (fix the criterion) or your intuition is wrong (update your mental model).

### Capability sets, trust & safety sets, regression sets, and gate-only sets

- **Capability eval sets** measure one capability dimension: accuracy/correctness, faithfulness/groundedness, relevancy, style/tone, or reasoning/tool use. They start at low pass rates — a 30% rate on a new capability set is useful signal, not a failure.
- **Trust & safety eval sets** are separate refusal/policy/sensitive-data/prompt-injection/compliance sets; they usually carry hard gates.
- **Regression sets** run on a cadence to detect drift: almost all capability sets plus a slim trust & safety subset.
- **Gate-only sets** run at milestones: most broad trust & safety checks. When a capability set saturates, promote representative cases to regression and write harder capability cases.

### Eval tooling (supplementary)

For tooling questions, the primary recommendation is Microsoft's Copilot Studio evaluation features for production Copilot agents. For teams needing third-party platforms:

- **Braintrust:** Good default for production agents. Free tier handles 1M spans/month.
- **LangSmith:** Best if already using LangChain. Native tracing.
- **Langfuse:** Best for self-hosted, data-sovereign setups. MIT-licensed.
- **Key warning:** Beware tools that auto-create rubrics AND auto-score without human calibration. The tool should support human review in the loop.

---

## Skill Routing — When to Suggest a Sibling Skill

After answering the question, check whether the user would benefit from running a sibling eval skill. If so, append a one-line recommendation at the end of your answer.

| If the question involves... | Suggest this skill | One-liner to append |
|---|---|---|
| Creating an eval plan or scoping what to evaluate | `/eval-suite-planner` | "For a populated Eval Suite Template workbook, run `/eval-suite-planner`." |
| Generating test cases, writing CSV datasets, building eval sets | `/eval-generator` | "To generate ready-to-import test case CSVs, run `/eval-generator`." |
| Interpreting scores, reading results, understanding pass rates | `/eval-result-interpreter` | "To interpret a specific set of eval results, paste them into `/eval-result-interpreter`." |
| Debugging failures, triaging low scores, root cause analysis, remediation | `/eval-triage-and-improvement` | "To triage specific failures with the full diagnostic framework, run `/eval-triage-and-improvement`." |
| What is eval, why eval matters, explaining eval to stakeholders | `/eval-guide` | "For an end-to-end eval explainer you can share with stakeholders, run `/eval-guide`." |

**Rules:**
- Only suggest ONE skill per answer — the most relevant one.
- Only suggest when the user's question clearly maps to an action a sibling skill performs. Do not suggest routing for pure methodology questions that eval-faq handles well on its own.
- Never suggest `/eval-faq` (that is this skill — they are already here).

---

## Example invocations

```
/eval-faq What eval scenarios should I use for a RAG agent?
/eval-faq How do I interpret a 75% knowledge grounding score?
/eval-faq What is the difference between business-problem and capability scenarios?
/eval-faq When should I use a model-graded grader instead of a deterministic one?
/eval-faq What makes a good adversarial test case?
/eval-faq How many cases do I need in a dataset to get meaningful signal?
/eval-faq My eval passes 100% on first run — is that good?
/eval-faq How do I write a good criterion for a model-graded grader?
/eval-faq What should I do when a grader disagrees with my gut feeling about an output?
/eval-faq How do I handle non-determinism in my eval results?
/eval-faq My agent makes tool calls — how do I eval those?
/eval-faq I suspect my grader is wrong — how do I debug it?
/eval-faq What should I eval in production after I ship?
/eval-faq Should I use pass@k or pass^k for my agent?
/eval-faq How do I calibrate my LLM-as-judge grader?
/eval-faq When do I stop adding eval cases and just ship?
/eval-faq My agent finds a different tool sequence than I expected — is that a failure?
/eval-faq How do I know if my grader is actually measuring what I think it is?
/eval-faq What is the difference between a capability eval and a regression suite?
/eval-faq How do I eval a multi-turn conversational agent?
/eval-faq What eval platform or tool should I use?
/eval-faq My agent passes evals but fails in production — why?
/eval-faq How do I score intermediate steps in a multi-step agent?
/eval-faq How is evaluating a multi-step workflow different from a simple Q&A agent?
/eval-faq What does 0% pass@100 mean — is my agent broken?
/eval-faq How do I avoid LLM judge bias in my grader?
/eval-faq Which eval sets should I include?
/eval-faq What is the Probe-Measure-Harden red-teaming framework?
/eval-faq What are the 7 test methods in Copilot Studio?
/eval-faq How do I use the Triage Playbook to debug failing scores?
/eval-faq How does the MS Learn iterative framework relate to the 10-step playbook?
/eval-faq What are the 3 root cause types for eval failures?
/eval-faq How do I decide between SHIP, ITERATE, and BLOCK?
/eval-faq What red-team ASR thresholds should I target?
/eval-faq How do I generate eval cases from a prompt template?
/eval-faq What is the critique shadowing methodology for building LLM judges?
/eval-faq Should I use a 1-5 scale or pass/fail for my LLM judge?
/eval-faq How do I continuously red-team my agent in CI/CD?
/eval-faq How do I systematically analyze eval failures to find patterns?
/eval-faq How do I know if my eval is too easy?
/eval-faq How do I write an LLM grader prompt that actually works?
/eval-faq Should I score factuality and tone in the same eval criterion?
/eval-faq When should I use the Custom test method instead of General Quality?
/eval-faq How do I set up a Custom test method for compliance checking?
```
