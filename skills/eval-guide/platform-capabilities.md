<!-- Companion reference for the `eval-guide` skill. Loaded on demand from SKILL.md. -->

## Platform Capabilities to Leverage (March 2026)

When coaching customers, mention these Copilot Studio evaluation features at the appropriate stage:

| Feature | When to mention | What it does |
|---|---|---|
| **Custom test method** | Stage 1 (Plan) | Lets customers define domain-specific evaluation criteria with custom labels (e.g., "Compliant" / "Non-Compliant"). Ideal for compliance, tone, or policy checks that don't fit standard methods. |
| **Comparative testing** | Stage 4 (Interpret) | Side-by-side comparison of agent versions. Use after making fixes to verify improvements without regressions. |
| **Theme-based test sets** | Stage 2 (Generate) | Creates test cases from production analytics themes — real user questions grouped by topic. Best for agents already in production. |
| **Production data import** | Stage 2 (Generate) | Import real user conversations as test cases. Higher fidelity than synthetic test cases. |
| **Rubrics (Copilot Studio Kit)** | Stage 1 (Plan) | Custom grading rubrics with 1-5 scoring and refinement workflow to align AI grading with human judgment. For advanced customers with mature eval practices. |
| **User feedback (thumbs up/down)** | Stage 4 (Interpret) | Makers can flag eval results they agree/disagree with. Captures grader alignment signals over time. |
| **Set-level grading** | Stage 4 (Interpret) | Evaluates quality across the entire test set (not just individual cases). Gives an overall quality picture and supports multiple grading approaches for more holistic results. Use this to report aggregate quality to stakeholders. |
| **User profiles** | Stage 2 (Generate) / Stage 3 (Run) | Assign a user profile to a test set so the eval runs as a specific authenticated user. Use this when the agent returns different results based on who is asking — e.g., a director can access different knowledge sources than an intern. Ask in Stage 0: "Does your agent behave differently depending on who the user is?" If yes, plan separate test sets per role. **Limitations:** (1) Multi-profile eval only works for agents WITHOUT connector dependencies. (2) Tool connections always use the logged-in maker account, not the profile — mismatch causes "This account cannot connect to tools" error. (3) Not available in GCC. Docs: [Manage user profiles](https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-edit#manage-user-profiles-and-connections). |
| **CSV template download** | Stage 2 (Generate) | Copilot Studio provides a downloadable CSV template under Data source > New evaluation. Recommend customers download it first to verify format before importing generated CSVs. |
| **89-day result retention** | Stage 3 (Run) / Stage 4 (Interpret) | Test results are only available in Copilot Studio for 89 days. **Always export results to CSV** after each run for long-term tracking. Critical for customers establishing baselines and tracking improvement over time. |

**Don't overwhelm.** Only mention features relevant to the customer's maturity level. A customer in Stage 0 doesn't need to hear about rubric refinement workflows.

**GCC (Government Community Cloud) limitations:** If the customer is in a GCC environment, flag these restrictions early:
- **No user profiles** — they can't assign a test account to simulate authenticated users during evaluation
- **No Text Similarity method** — all other test methods work normally
These are documented at [About agent evaluation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-intro). Don't let them design an eval plan around features they can't use.

**Important caveat to share:** Agent evaluation measures correctness and performance — it does NOT test for AI ethics or safety problems. An agent can pass all eval tests and still produce inappropriate answers. Customers must still use responsible AI reviews and content safety filters. Evaluation complements those — it doesn't replace them.

---

