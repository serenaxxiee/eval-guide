<!-- Companion reference for the `eval-guide` skill. Loaded on demand from SKILL.md. -->

## Stage 0: Discover

Help the customer articulate what their agent is supposed to do and what "good" looks like. This is the most important stage — it shapes everything downstream.

### What you walk away with

- **A 1-page Agent Vision** — purpose, users, knowledge sources, core capabilities, boundaries (what the agent must NOT do), success criteria, role-based access, and the **eval objective + risk tier + owner** (playbook **Step 1**). Written down, not assumed.
- **The eval objective** — one sentence naming what "good" looks like and what decisions the evals will inform. It anchors every later choice.
- **The agent's risk tier** — HIGH / MEDIUM / LOW, classified from the **five risk factors**: reach, criticality of error, autonomy / blast radius, regulatory exposure, data sensitivity. The risk tier drives pass-rate targets, gate strictness, required trust & safety categories, and minimum adversarial coverage downstream.
- **A named owner** — one person accountable for authoring the eval, reviewing results, and signing off.
- **Stakeholder alignment** — or, more often, a *surfaced disagreement* between builder and PM about scope. 10 minutes of structured questions catches what would otherwise cost weeks of rework.
- **The spec every later stage depends on.** The Plan stage's eval plan, the Generate stage's test cases, and the Interpret stage's pass/fail judgment all trace back to what gets named here.

### When this stage is wrong for you

- You already have a written PRD, agent spec, or design doc that covers all 7 questions below. Bring it and skip to Stage 1.
- You have eval results in hand and need triage now — go straight to Stage 4.
- Your agent is a 50-topic monster. One Stage 0 pass won't fit; run Stage 0 per top-level capability.

### What to do — extract Vision, apply safe defaults, proceed to Stage 1

**Don't ask Q1–Q7 in chat.** This was the old flow; it tested as an interrogation and customers tuned out. The new flow: extract everything you can from the customer's kickoff description, fill the gaps with **domain-keyed safe defaults**, summarize in 5–6 lines, and proceed straight to the Plan dashboard. The customer corrects in chat ("actually, peer comp comparison isn't a boundary for us") or via the dashboard's General Comments box. Nothing is locked until they confirm in the dashboard.

#### Step 1 — Pre-extract from the kickoff

From the customer's 1–4 sentence description, extract:
- **Purpose** — usually the first clause ("Personalized HR support…")
- **Users** — usually implied ("employees," "customers," "internal teams")
- **Capabilities** — usually a list ("benefits, training, policies")
- **Knowledge sources** — sometimes named, often categorized ("official company resources" → SharePoint TBD)
- **Tone hints** — sometimes explicit ("trusted HR colleague," "efficient")
- **Personalization hints** — words like "personalized," "your," "based on your role"

If the kickoff is too thin (one sentence with no domain hint), ask **one** clarifying question — *"Two more sentences on what it does and who uses it would help me draft a Vision faster"* — then resume.

#### Step 2 — Apply safe defaults by domain

Domain detection runs on keywords in the kickoff description. Pick the matching default set:

| Domain trigger keywords | Default boundaries (what NOT to do) | Default risk tier |
|---|---|---|
| **HR / ESS / employee / benefits / policy / leave / payroll** | Legal advice; medical advice; salary negotiation; performance review interpretation; HR investigation details; peer compensation comparison; PII about other employees | **HIGH** (data sensitivity + regulatory exposure) |
| **Customer support / refunds / billing / accounts** | Refunds beyond policy; account-specific data outside this user's scope; legal-binding promises; competitor product recommendations | **HIGH** (reach + criticality: customer trust + financial) |
| **Knowledge / documentation / FAQ / wiki** | Content beyond the named knowledge sources; opinions framed as facts; regulated advice (legal/medical/financial) | **MEDIUM** (defaults higher if regulated content domain) |
| **IT / helpdesk / troubleshooting** | Remote-execute actions on user systems; reset credentials without verification; security advice that bypasses policy | **MEDIUM** (HIGH if security/privacy adjacent) |
| **Agentic / tool-using / "submits" / "schedules" / "books"** | Irreversible actions without confirmation; actions outside user's authorization scope; anything requiring approval the agent can't get | **HIGH** (autonomy / blast radius: writes to systems) |
| **No domain detected** | "Outside the named knowledge sources" + "anything the user-cohort isn't authorized for" + 1 generic safety guardrail | **MEDIUM** (default cautious) |

The default tier is a starting point keyed on domain. Confirm it against all five risk factors — **reach** (who and how many use it), **criticality of error** (financial/legal/safety/reputational consequence), **autonomy / blast radius** (does it only draft text a human reviews, or take irreversible actions?), **regulatory exposure** (HIPAA/GDPR/SOX/fiduciary/attorney-client), **data sensitivity** (PII/PHI/confidential/source code). In enterprise contexts autonomy and regulatory exposure often dominate, so bump the tier up when either is present even if reach is small.

**Default success criteria** (always include unless customer overrides):
- Most user questions answered directly (deflection / self-service rate)
- Out-of-scope questions routed clearly to the right human or resource (graceful handoff)
- Zero privacy / boundary breaches

**Default knowledge sources** when only categorized:
- *"some SharePoint sites"* / *"internal docs"* → flag as `Multiple SharePoint sites (TBD — name in Plan dashboard)` so the customer can fill names without us blocking on it.

**Auto-detect role-based access:** if the customer's description contains "your," "personalized," "based on your," "role-specific," "tailored to," set `role_based_access: true` and infer 2–3 likely personalization axes from the agent's domain (HR/ESS → location, tenure, plan; customer support → account tier, region; etc.). Customer corrects if wrong.

#### Step 3 — Drop aspirational-language capabilities silently

Marketing-language capabilities like *"empower employees," "explore opportunities," "streamline X"* don't survive the concreteness check. Drop them from Core Capabilities and add a one-line note in the Vision summary: *"Note: dropped 'explore opportunities' as aspirational — not a testable feature. Tell me if it's actually a concrete capability and I'll add it back."*

This is silent removal with a flagged note, not a question. Customer can flag if they disagree.

#### Step 4 — Show the Vision summary in chat (5–6 lines, no questions)

Display the pre-extracted Vision compactly:

```
Agent Vision: [Name]

Eval objective: [one sentence — what "good" means + what decision the evals inform]
Purpose:        [one sentence from kickoff]
Users:          [extracted or default]
Knowledge:      [named sources, or "TBD — confirm in Plan dashboard"]
Capabilities:   [3–5 from kickoff, aspirational dropped]
Boundaries:     [domain default set, listed]
Success:        [default 3 criteria]
Role-based:     [auto-detected: yes/no, with axes]
Risk tier:      [domain default: HIGH/MEDIUM/LOW] — driven by 5 factors (reach, criticality, autonomy, regulatory, data sensitivity)
Owner:          [named accountable owner, or "TBD — name before deploy"]
```

Then: *"This is what I extracted from your description, with safe defaults for [HR/ESS/etc.] domain agents filling the gaps. **Speak up now if any of this is wrong** — boundaries, risk tier, eval objective, or capabilities especially. I'm proceeding to draft the eval plan; you'll review the full criteria + matrix in the Plan dashboard."*

**Don't gate on customer confirmation.** Write `stage-0-data.json` and proceed to Stage 1 immediately. The customer either replies with corrections (which you incorporate before launching the dashboard) or stays silent (proceed). The Plan dashboard is the real review surface.

#### Why this works

- **Pre-extraction + defaults** covers ~80% of what the chat questions extracted, with zero customer chat input beyond the kickoff.
- **Defaults are domain-keyed**, so they're rarely wrong for common agent types (HR, customer support, IT, knowledge).
- **The Plan dashboard is the correction surface** — visual, all-at-once, lets the customer fix Vision-level issues alongside criteria-level edits in one pass.
- **Customer can always correct in chat** before the dashboard launches, but isn't forced to.

#### When this approach is wrong (revert to gap-question batch)

- The kickoff description is genuinely too thin — one sentence with no domain keywords. Ask one clarifying question to get enough material for safe defaults.
- The customer is in a regulated-but-uncommon domain (medical devices, financial services, government) where the default boundaries don't fit. After step 2, ask: *"Domain looks like [X] — your boundaries are usually [Y]. Anything specific I should add for your context?"*
- The customer has explicitly said the agent is novel / experimental and they want to talk through it. Default to conversation mode for these — but they're a small minority.

---

