---
name: eval-faq
description: Answers AI agent evaluation methodology questions with practical, opinionated guidance grounded primarily in Microsoft's agent evaluation ecosystem (MS Learn, Eval Scenario Library, Triage & Improvement Playbook, Eval Guidance Kit) supplemented by select industry sources.
---

## Purpose

Answer any question about eval methodology, grader types, dataset design, criteria writing, non-determinism, tool-call evaluation, multi-turn agent evaluation, eval tooling, capability vs. regression evals, and interpreting results — specifically in the context of AI agent evaluation. The primary methodology is `skills/eval-guide/playbook.md`: **Practical Guidance on Agent Evaluation: a 10-step playbook**. Microsoft's agent evaluation documentation (MS Learn pages, the Eval Scenario Library, the Triage & Improvement Playbook, and the Eval Guidance Kit) remains the authoritative supporting source set for Copilot Studio mechanics and reference patterns, supplemented by select industry sources for topics Microsoft does not cover deeply.

## Instructions

When invoked as `/eval-faq <question>`, follow this process exactly:

### Step 1 — Fetch authoritative context before answering

Read `sources.md` first for the topic-to-URL routing table and fetch rules. Fetch only the URL(s) that match the question topic, extract only the relevant section, and then return here for Step 2. If fetch fails, note "Source unavailable at fetch time — answering from knowledge base." and use `knowledge-base.md`.

Open companion files at these points:
- `sources.md` — before answering any question, to choose authoritative Microsoft and supplementary sources and apply fetch/citation rules.
- `knowledge-base.md` — whenever fetched content does not cover the question, a fetch fails, or you need the canonical 10-step playbook details and topic references.
- `sibling-routing-and-examples.md` — after drafting the answer, to decide whether one sibling-skill recommendation should be appended; also use it for example invocation coverage.

### Step 2 — Answer using fetched content plus knowledge base

Synthesize the fetched content with the knowledge base below. The 10-step playbook is the methodology spine; Microsoft fetched content supplies supporting details and Copilot Studio specifics, then external sources fill gaps.

**Answer style rules — no exceptions:**
- Answer in 3-5 sentences maximum. No padding, no preamble, no "great question."
- Give opinionated, direct guidance. Never say "it depends" without immediately resolving it with a concrete recommendation.
- Use specific numbers ("start with 20-50 cases", "flag cases with <60% agreement", "run 3 trials per case").
- Ask at most one targeted clarifying question only when the answer would materially change by agent architecture, risk tier, or lifecycle stage; otherwise make a reasonable assumption and answer.
- Cite which source you used at the end of the answer.

---

### Step 3 — Check sibling-skill routing

Read `sibling-routing-and-examples.md` before appending any recommendation. Append at most one sibling-skill recommendation, and only when that file's rules clearly map the user's question to a next action.
