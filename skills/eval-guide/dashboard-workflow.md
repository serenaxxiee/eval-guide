<!-- Companion reference for the `eval-guide` skill. Loaded on demand from SKILL.md. -->

## Review Checkpoint Workflow

Plan produces a populated Eval Suite Template workbook plus a companion interactive HTML review page. Generate and Interpret produce **interactive HTML dashboards** that open directly in the browser. Dashboard stages run against a tiny localhost HTTP server (`serve.py --serve`); the customer never sees, downloads, or moves a JSON file. Feedback flows from the browser → server → the AI's `bash` stdout, in one step.

**Flow at each dashboard review stage (Generate, Interpret):**
1. Complete the stage's analysis.
2. Write stage data to a JSON file (e.g., `stage-1-data.json`).
3. Launch with `--serve` mode. The AI's shell blocks until the customer clicks Approve or Regenerate. Resolve `<skill-dir>` from the skill context (see "Resolving skill-bundled files" at the top of this file):
   `python "<skill-dir>/dashboard/serve.py" --stage <name> --serve --data <file>.json`
4. The customer reviews in the browser at `http://localhost:3118`: edits fields inline, updates eval-set/case/root-cause details, and adds comments. Edits auto-save to the localhost server.
5. When the customer clicks **Approve & Continue** or **Incorporate Changes & Regenerate**, the browser POSTs the feedback to `/api/feedback`. The server captures it, prints the feedback JSON to stdout between marker lines, and shuts down. **No file is downloaded; the customer never moves anything.**
6. **Parse the feedback from the bash command's stdout** — look for the block:
   ```
   ===EVAL_GUIDE_FEEDBACK_BEGIN===
   { "stage": "...", "status": "confirmed" | "changes_requested", "edits": {...}, "comments": "..." }
   ===EVAL_GUIDE_FEEDBACK_END===
   ```
   Decode the JSON between those markers — that's the customer's feedback. (`<stage>-feedback.json` is also written next to the data file as a debugging backup, but stdout is the primary channel — read from there.)
7. If `status: "confirmed"` → apply the edits, generate final deliverables (docx, CSV), proceed to next stage.
8. If `status: "changes_requested"` → apply the edits, regenerate the stage data file, re-launch the dashboard. Same loop.

The **orient stage is a pre-built static HTML** (`dashboard/orient-dashboard.html`) — agent-agnostic, no `serve.py`, no JSON write, no feedback file. The skill simply opens the file in the customer's browser and continues the conversation. See *Session Start: Orient* in `SKILL.md`.

**Review checkpoints:** Plan uses workbook + HTML review. Generate and Interpret use dashboards. Stage 3 (Run) executes tests directly.

**Key principle:** No final docx or CSV files are generated until the customer confirms the relevant checkpoint. The checkpoint replaces the "does this look right?" chat-based confirmation with a structured review.

