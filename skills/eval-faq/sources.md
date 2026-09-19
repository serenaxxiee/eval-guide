# Eval FAQ sources and fetch routing

This companion contains the source-routing and fetch-rule material relocated verbatim from SKILL.md. Read it before answering any /eval-faq question, fetch the matching source(s), then return to SKILL.md Step 2.

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
