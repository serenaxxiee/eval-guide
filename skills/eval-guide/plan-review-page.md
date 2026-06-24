# Eval Plan HTML Review Page

Use this blueprint whenever `/eval-suite-planner` or Stage 1 of `/eval-guide` generates a plan from the Eval Suite Planning & Logging Template.

The primary plan artifact remains the populated `.xlsx` workbook. The HTML page is the customer-facing review surface so the chat response can stay short and the user can explore the plan interactively.

## File naming

Generate the HTML page next to the workbook:

`eval-suite-<agent-name>-<YYYY-MM-DD>-review.html`

## Chat behavior

Do not paste the workbook summary, eval-set tables, or human-review checklist into chat. After the workbook and HTML page are created, respond with only:

1. the workbook path;
2. the HTML review page path;
3. any blocker or required manual action, if one exists.

If HTML generation fails, say so plainly and still provide the workbook path. Do not fall back to a long narrative unless the user explicitly asks for the text version.

## Template integrity

The HTML page is a companion artifact. It must not replace, reshape, or write back to the workbook. The workbook still follows `eval-suite-template.md`: no sheet, header, dropdown, README, style, validation, or structure changes.

## Required page content

Use the populated workbook data as the source of truth.

Include:

- **Hero summary**: agent name, eval objective, risk tier, lifecycle stage/version, workbook filename.
- **Metric cards**: capability eval-set count, Trust & Safety eval-set count, hard-gate count, baseline placeholder count, reusable candidate count, and `TBD - confirm before baseline` count.
- **Risk and Step 4 governance panel**: five-factor risk drivers, T&S hard gates, capability launch floors / regression-direction, and any high-risk capability floor.
- **Interactive eval-set explorer**:
  - filter chips for `All`, `Capability`, `Trust & Safety`, `Hard gates`, `Needs owner`, `Reusable`;
  - one card per eval-set registry row;
  - show ID, name, category, dimension, target, gate type, intended use, cadence, human input owner, source dependency, source-review trigger, reusable tier, status, and notes;
  - use expandable details for long rationale/notes.
- **TBD action list**: grouped by owner/source/compliance/tool-autonomy gaps, with copy-to-clipboard for the item text.
- **Baseline readiness checklist**: the human review checkpoints from the planner, as checkboxes persisted in `localStorage`.
- **Reusable assets panel**: cards from `4 . Reusable Library`.
- **Next step callout**: validate graders before trusting baseline scores, then use `/eval-generator` from the workbook registry.

## Interactivity

Use plain HTML, CSS, and JavaScript only; no external libraries or network calls.

Required interactions:

- client-side filtering of eval-set cards;
- expandable/collapsible details;
- persisted checklist state with `localStorage`;
- optional search by eval-set name, ID, category, dimension, owner, or notes;
- optional copy buttons for workbook path and TBD items.

Open the page in the browser when the environment supports it.

## Mandatory Clawpilot theme

Every generated HTML page must include this theme detection script before any other JavaScript:

```html
<script>
  (() => {
    const param = new URLSearchParams(window.location.search).get("clawpilotTheme");
    const theme =
      param || (window.matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light");
    document.documentElement.setAttribute("data-theme", theme);
  })();
</script>
```

Every generated HTML page must include these CSS variables exactly in its `<style>` block:

```css
:root {
  color-scheme: light;
  --cp-bg: #f7f4ef;
  --cp-bg-elevated: #fcfbf8;
  --cp-surface: #ffffff;
  --cp-surface-soft: #f5f5f5;
  --cp-border: #dedede;
  --cp-border-strong: #919191;
  --cp-text: #242424;
  --cp-text-muted: #5c5c5c;
  --cp-text-soft: #6f6f6f;
  --cp-accent: #b11f4b;
  --cp-accent-hover: #9a1a41;
  --cp-accent-soft: rgba(177, 31, 75, 0.08);
  --cp-accent-fg: #ffffff;
  --cp-success: #16a34a;
  --cp-danger: #dc2626;
  --cp-warning: #f59e0b;
  --cp-link: #0078d4;
  --cp-shadow: 0 18px 48px rgba(0, 0, 0, 0.12);
  --cp-overlay: rgba(255, 255, 255, 0.8);
  --cp-panel: rgba(255, 255, 255, 0.86);
  --cp-panel-strong: rgba(255, 255, 255, 0.96);
  --cp-sheen: rgba(255, 255, 255, 0.55);
  --cp-highlight: rgba(177, 31, 75, 0.12);
}
html[data-theme="dark"] {
  color-scheme: dark;
  --cp-bg: #3d3b3a;
  --cp-bg-elevated: #343231;
  --cp-surface: #292929;
  --cp-surface-soft: #2e2e2e;
  --cp-border: #474747;
  --cp-border-strong: #5f5f5f;
  --cp-text: #dedede;
  --cp-text-muted: #919191;
  --cp-text-soft: #b0b0b0;
  --cp-accent: #fd8ea1;
  --cp-accent-hover: #fb7b91;
  --cp-accent-soft: rgba(253, 142, 161, 0.14);
  --cp-accent-fg: #1a1a1a;
  --cp-success: #4ade80;
  --cp-danger: #f87171;
  --cp-warning: #fbbf24;
  --cp-link: #4da6ff;
  --cp-shadow: 0 18px 48px rgba(0, 0, 0, 0.32);
  --cp-overlay: rgba(41, 41, 41, 0.88);
  --cp-panel: rgba(41, 41, 41, 0.72);
  --cp-panel-strong: rgba(41, 41, 41, 0.96);
  --cp-sheen: rgba(255, 255, 255, 0.04);
  --cp-highlight: rgba(253, 142, 161, 0.12);
}
```

For all component styles, use only `var(--cp-*)` color variables. Do not hardcode hex/rgb/hsl colors outside the required theme block.

Typography:

- Font: `"Segoe UI", Aptos, Calibri, -apple-system, BlinkMacSystemFont, sans-serif`
- Monospace: `Consolas, "Courier New", Courier, monospace`

Shape and spacing:

- Use `0.625rem` radius for controls and `16px` for cards.
- Use subtle card shadows: `0 0 2px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.14)`.
- Use consistent 4px-based spacing.

## Validation

Before handing off, verify:

- the HTML file exists and is self-contained;
- the page has no external CSS, JS, font, or image references;
- the workbook path shown in the page matches the generated workbook;
- all filter buttons work;
- checklist state persists after refresh;
- the final chat response is path-focused and not a duplicate of the page content.
