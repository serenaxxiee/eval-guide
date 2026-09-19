# Eval FAQ sibling routing and examples

This companion contains the sibling-skill routing rules and example invocations relocated verbatim from SKILL.md. Read it after drafting an answer when deciding whether to append a one-line sibling-skill recommendation.

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
