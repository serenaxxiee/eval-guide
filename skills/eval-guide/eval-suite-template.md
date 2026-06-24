# Eval Suite Planning & Logging Workbook Template

Use this blueprint whenever a skill needs to generate the customer-facing eval planning workbook. The workbook turns the 10-step Practical Guidance on Agent Evaluation into one populated `.xlsx` file per agent.

**Template preservation rule:** If the physical blank template workbook is available in the session, copy that workbook and populate only existing input cells and data rows. Do **not** modify worksheet names, column headers, formulas, styles, widths, data validations, README text, or the `Dropdown Lists` tab. If the blank template is not available, ask the user for it instead of creating a different workbook shape.

## Workbook name

`eval-suite-<agent-name>-<YYYY-MM-DD>.xlsx`

The planner also generates a companion HTML review page next to the workbook:

`eval-suite-<agent-name>-<YYYY-MM-DD>-review.html`

The HTML page is an interactive review surface only. It must not replace the workbook or modify the template structure. Use `plan-review-page.md` for the HTML contract.

## Tabs

1. `README`
2. `1 . Planning`
3. `2 . Eval Suite Registry`
4. `3 . Run Log`
5. `4 . Reusable Library`
6. `Dropdown Lists`

## README

Static instructional tab. Preserve exactly. Do not edit this sheet when populating a workbook.

- Title: `Eval Suite Planning & Logging Template`
- Subtitle: `Project Phoenix | Agent Quality & Evaluation - companion to the 10-step Practical Guidance on Agent Evaluation`
- `How to use this workbook`
- `The tabs and the steps they cover`
- `Two core distinctions to keep straight`

The two distinctions must say:

- **Capability vs Trust & Safety** — capability measures how well the agent does its job; trust & safety measures what it must refuse, avoid, or route. Hallucination is a faithfulness/groundedness capability failure, not a trust & safety category.
- **Gate vs Regression** — gate means must pass before pilot/production; regression means run on a cadence to catch drift.
- **Two kinds of failure (Step 7)** — every failure is either an eval-setup problem or an agent-quality problem.

## 1 . Planning

Populate the existing yellow/input cells for Step 1. Do not insert rows, columns, or new sections.

### Agent identity

| Field | Populate with |
|---|---|
| Agent name | Agent name from the user, dashboard, or inferred description |
| Description / job to be done | One-sentence Agent Vision summary |
| Lifecycle stage | Idea / design / build / pilot / production / update |
| Eval owner (builder) | Named accountable eval owner; ask if missing |
| Date / version of this plan | Current date and agent/version label if known |

### Risk classification (5 factors)

Rows:

| Factor | What to assess | Your assessment |
|---|---|---|
| Reach | Number/type of users - internal team, company-wide, external customers, regulated audiences. | Low / Medium / High + short rationale |
| Criticality of error | Consequence of a wrong/harmful response - financial loss, legal exposure, safety, reputation. | Low / Medium / High + short rationale |
| Autonomy & blast radius | Text a human reviews, or tool calls / actions? Are actions reversible? | Low / Medium / High + short rationale |
| Regulatory & compliance exposure | Regulated domain? HIPAA, GDPR, SOX, attorney-client, fiduciary, etc. | Low / Medium / High + short rationale |
| Data sensitivity | PII, PHI, confidential business data, source code, other sensitive inputs/outputs. | Low / Medium / High + short rationale |
| -> Risk tier (overall) | Autonomy & regulatory exposure often dominate in enterprise contexts. | Low / Medium / High risk tier |
| Risk tier rationale | Why this tier - the 1-2 factors that drove it. | Concise rationale |

### Owners & roles

Rows:

| Role | Responsibility | Named owner |
|---|---|---|
| Agent builder | Runs the eval suite, drives iteration. | Required; ask if missing |
| Domain expert | Authors rubrics, ground truths, golden answers (Step 5). | Ask or set `TBD - name before baseline` |
| Eval reviewer | Inspects failures case-by-case in early iterations (Step 7). | Ask or set `TBD - name before baseline` |
| Risk / compliance owner | Signs off for higher-tier agents; required for regulated deployments. | Required for high-risk agents; ask if missing |

### Deployment gates / sign-off criteria

Rows:

| Item | Definition | Decision |
|---|---|---|
| Min pass rate - Capability | In v5 guidance, most capability sets use launch floors plus regression/direction rather than standing absolute targets. | Fill with launch floor / high-risk capability floor / regression-governance note, not a generic per-scenario target |
| Min pass rate - Trust & Safety | T&S sets are typically absolute hard gates at a high bar. | Derived from risk tier; ask if deployment gate is known |
| Mandatory risk coverage | Which T&S categories MUST be covered before deploy. | Derived from risk tier/domain |
| Named approver | Who signs off. | Risk/compliance owner or eval owner |
| Required evidence | Baseline report, failure log, regression green, etc. | Derived from lifecycle stage |
| Policy / compliance requirements | Company policy or regime the eval program must satisfy. | Ask if regulated/compliance domain is detected |

## 2 . Eval Suite Registry

Populate one row per eval set, not one row per individual test case or legacy planning artifact. The registry is the workbook's Steps 2-5 artifact.

Columns:

| Column | Populate with |
|---|---|
| ID | Stable ID such as `CAP-ACC-001`, `TS-PII-001` |
| Eval Set Name | Human-readable set name |
| Category | `Capability` or `Trust & Safety` |
| Dimension tested | Capability dimension or T&S category |
| Purpose / diagnostic signal | What this set detects |
| Target pass rate | For T&S: absolute pass-rate gate. For capability: launch floor or `Regression / direction after baseline` unless high-risk capability needs a hard floor. |
| Target rationale | Why this governing instrument fits the risk tier, criticality, and v5 Step 4 guidance |
| Gate type | Use the closest existing dropdown value: `Hard gate`, `Soft target`, or `Hard floor + soft target`. Explain launch-floor / regression-direction nuance in `Notes` because the template has no separate governing-instrument column. |
| Intended use | `Gate`, `Regression`, or `Both` |
| Run cadence | `Per-change`, `Nightly`, `Weekly`, `Milestone-only`, etc. |
| Human input type | `Grading rubric`, `Ground truth`, `Golden answer`, `Rubric + ground truth`, `Rubric + eval set` |
| Human input author | Named SME/risk/content owner or `TBD - name before baseline` |
| Grounding source dependency | Source system/document that must stay in sync |
| Source change -> review? | `Yes` or `No` |
| Run Cadence | Same cadence as run cadence column; preserve because the template contains both columns |
| Reusable asset? | `Yes - candidate` or `No - agent-specific` |
| Reuse tier | `Required`, `Recommended`, or `Opt-in` when reusable |
| Set status | `Draft`, `Active`, or `Deprecated` |
| Notes | Assumptions, open questions, owner decisions, Step 4 governing-instrument nuance, and Step 6 grader-validation requirement. Since the template has no grader-validation columns, record grader type and validation plan here without adding columns. |

## 3 . Run Log

Leave mostly blank when only planning is complete. Add an initial placeholder row for each planned baseline set when useful. Step 7 baseline results are recorded here after Step 6 grader validation is complete.

Columns:

| Column | Populate with |
|---|---|
| Run ID | `BASELINE-001` for planned baseline rows |
| Eval Set ID | Registry ID |
| Eval Set Name | Registry name |
| Run date | Blank until run, or planned date if known |
| Agent version | Plan version / agent version if known |
| Run type | `Baseline` for first planned run |
| Result (pass rate) | Blank until run |
| Target | Target from Registry |
| Target met? | Blank until run |
| Cases passed / total | Blank until run |
| Failure classification | Blank until run; later use Eval-setup problem / Agent-quality problem / Mixed / N/A, matching v5 Step 7 |
| Failure pattern identified | Blank until run |
| Actionable next step | `Run baseline` for placeholder rows |
| Action owner | Eval owner |
| Status | `Open` |

## 4 . Reusable Library

Populate candidate reusable assets identified during planning. Prioritize trust & safety sets, rubrics, and failure-pattern templates that are not agent-specific.

Columns:

| Column | Populate with |
|---|---|
| Asset ID | Stable ID such as `LIB-TS-001` |
| Asset name | Reusable asset name |
| Asset type | Trust & safety eval set / Grading rubric / Failure-pattern template / Production edge case |
| Source agent | Agent name |
| Tier | Required / Recommended / Opt-in |
| Owner | Proposed library owner |
| Version | Initial version, e.g. `v0.1` |
| Last reviewed | Current date or blank |
| Review cadence | Quarterly / On policy change / etc. |
| Applies to (agent categories) | Agent categories that can reuse it |
| Notes | Promotion rationale and assumptions |

## Dropdown Lists

Preserve exactly. Do not edit this sheet when populating a workbook; changing it modifies the template's controlled vocabularies and validations.

| Category | Dimension | GateType | IntendedUse | Cadence | HumanInput | SourceReview | Reusable | ReuseTier | Promotion | SetStatus | RunType | TargetMet | FailureClass | ActionStatus | AssetType |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Capability | Accuracy / correctness | Hard gate | Gate | Per-change | Grading rubric | Yes | No - agent-specific | Required | Agent-specific | Draft | Baseline | Yes | Eval-setup problem | Open | Trust & safety eval set |
| Trust & Safety | Faithfulness / groundedness | Soft target | Regression | Nightly | Ground truth | No | Yes - candidate | Recommended | Flagged for promotion | Active | Iteration | No | Agent-quality problem | In progress | Grading rubric |
|  | Relevancy | Hard floor + soft target | Both | Weekly | Golden answer |  |  | Opt-in | Promoted | Deprecated | Regression | Partial | Mixed | Resolved | Failure-pattern template |
|  | Style & tone |  |  | Milestone-only | Rubric + ground truth |  |  |  |  |  | Gate check |  | N/A - passed | Won't fix | Production edge case |
|  | Reasoning & tool use |  |  |  | Rubric + eval set |  |  |  |  |  |  |  |  |  |  |
|  | Guardrails |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Out-of-scope handling |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Sensitive-data handling |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Prompt injection / jailbreak |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Compliance-specific |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

## Question policy

Skills may ask targeted questions before populating this workbook. Ask only for fields that materially affect the workbook and cannot be inferred safely:

1. Eval owner / named approver.
2. Lifecycle stage and target deployment decision.
3. Whether the agent uses tools/connectors or only answers from knowledge.
4. Regulated/compliance obligations.
5. Authoritative sources and their owners.

If the user wants speed or cannot answer, populate `TBD - confirm before baseline` rather than blocking.

## What not to generate in this workbook

- Do not add a scenario plan sheet.
- Do not add a quality-signals sheet.
- Do not create one row per test case.
- Do not add columns for grader validation, production signals, optimization loops, or scenario metadata. Use existing `Notes`, `Run Log`, and `Reusable Library` fields instead.
