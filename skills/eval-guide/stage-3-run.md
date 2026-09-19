<!-- Companion reference for the `eval-guide` skill. Loaded on demand from SKILL.md. -->

## Stage 3: Run (requires a running agent)

Stage 3 turns the eval set into evidence. Run your CSVs against the live agent and record the results. 10–30 minutes (depends on test count and auth setup).

### What you walk away with

- **`eval-results-<agent>-<date>.csv`** — pass/fail per case, score per LLM method, judge rationale.
- **`eval-results-<agent>-<date>.json`** — same data, programmatic-friendly.
- **A baseline pass rate and gate status by eval set** — the number every future change is compared against.

### Skip this stage if

- **Your agent isn't built yet.** The deliverables from Stages 0–2 are the eval jumpstart; come back when the agent is running.
- **You already have eval results** (prior run, internal/external testing tool). Skip to Stage 4.

### Set expectations before you run

**First-run pass rate is usually 40–70%, not 80%+.** Customers who get 50% on the first run sometimes spiral; they shouldn't. The valuable signal is *which categories* pass and fail, not the headline number. Stage 4 turns the failures into ranked action.

**LLM-judge methods are non-deterministic** — `Compare meaning` and `General quality` show ±5% variance between runs. If a result lands borderline, run it again and take the median.

### Two paths — pick one

| Path | When it's right | Setup cost |
|---|---|---|
| **Copilot Studio UI Evaluation tab** *(default — start here)* | Most customers, especially incidental users. Import `eval-<set-type>-<set-slug>-<date>.csv`, run, view results in the UI. Use this unless you need automation. | Agent auth only. |
| **`eval-runner.js` (CLI)** | You need to automate, run from CI, or use LLM-judge methods the UI doesn't expose. | Node, DirectLine token endpoint, `ANTHROPIC_API_KEY` (real $ — Claude API costs apply). |

### How to run (CLI path)

```bash
node eval-runner.js --token-endpoint "<URL>" --csv-dir .
```

Or use `/chat-with-agent` for individual questions via the Copilot Studio SDK.

**Scoring methods:**
- `Compare meaning` → semantic equivalence (0.0–1.0, LLM judge)
- `General quality` → relevance / groundedness / completeness / abstention (0.0–1.0, LLM judge)
- `Keyword match` → code-based string matching (free, deterministic)
- `Exact match` → code-based string equality (free, deterministic)

Required: `ANTHROPIC_API_KEY` for LLM-judge methods. Code-based methods run free.

### How to get value from it

- **Don't panic at the first-run pass rate.** 40–70% is normal. Read hard gates, eval-set pass rates, and regression/direction instead of the headline.
- **Export results immediately to CSV.** Copilot Studio retains run results for only 89 days. You need the CSV for long-term tracking and for Stage 4 interpretation.
- **Run twice if borderline.** LLM-judge scoring is non-deterministic; re-run and take the median.
- **Run hard-gated Trust & Safety sets and impacted capability regression sets first.** If a hard gate fails, the release decision is blocked until it is fixed or explicitly waived by the accountable owner.

### Output

Results table printed to terminal + `eval-results-<agent>-<date>.csv` and `.json` written to disk.

---

