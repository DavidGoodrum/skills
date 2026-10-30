---
name: prototype-brief
description: Compare a prototype (usually HTML) with a vision/PRD document, find conflicts and gaps, ask the user how to resolve them, then write the prototype brief for AI-DLC.
---

# Prototype brief

Turn a click-through prototype plus a written document (usually a vision document or PRD) into a single **prototype brief** that an AI-DLC Inception agent can use alongside both originals. The value is in finding where the two disagree or are silent, getting the product owner to decide, and recording the decisions.

## Inputs

- **Prototype**: normally a standalone HTML file. Screenshots, Figma exports or recordings also work, with the limits noted below.
- **Document**: a vision document, PRD or similar. Sometimes several.
- If either is missing, check the conversation and attachments first, then ask for it. Do not proceed from the prototype alone.

## Workflow

### 1. Read both completely

Read the whole document. For an HTML prototype, read the JavaScript (state, handlers, timers, validation, copy), not just the markup and CSS. If the file is large, read it in chunks rather than skimming.

### 2. Build a behavior model of the prototype

Capture:
- Screens and states, and the transitions between them.
- Settings, their defaults and their dependencies (what is disabled or hidden when another setting changes).
- Validation rules and exact error copy.
- Timers (idle timeouts, auto-dismiss, polling) and which screens they do or do not apply to.
- Failure and offline states.
- **Demo-only scaffolding**: control bars, outcome dropdowns, magic IDs or PINs, hard-coded names, customers, sites and timestamps, simulated failures, retry that always succeeds. These must not become requirements.
- **Unreachable behavior**: trace state variables. A flag that is initialized but never assigned means the screens behind it may be dead code. State this as something read from the code and suggest the user click through to confirm.

For non-HTML prototypes (screenshots, design exports) list what cannot be known from pictures (timers, validation, transitions, error paths) and turn those into questions.

### 3. Extract the document's commitments

Scope in and out, rules and numbers, constraints, "must not change" statements, non-functional targets, risks, and any open questions already recorded.

### 4. Compare and classify

Sort every finding into one of these buckets and give each an ID:
- **M#**: mismatch, where prototype and document disagree.
- **G#**: gap, where both are silent but a builder will need a decision (data contracts, limits, failure behavior, security handling, time and time zone, multiple matches, staleness).
- **V#**: inconsistency inside the document itself (contradictions, garbled text, wording that conflicts with its own example).
- **P#**: oddity inside the prototype (a screen missing a timeout, behavior that looks accidental).

Also check the prototype against the document's "must not change" and regression-risk statements. A prototype behavior that changes existing product behavior is high priority even if the document never mentions it.

For each mismatch note what each source says, and a suggested handling. Default precedence, unless the user says otherwise: **the document wins on scope, intent and constraints; the prototype wins on interaction detail and on-screen copy.** Never resolve a conflict silently.

### 5. Present findings, then ask

Give a short summary in chat first: counts per bucket and the few items that matter most.

Then ask the user how to resolve the important items with AskUserQuestion (up to four questions per call):
- Put the highest-impact items first: anything that changes existing product behavior, scope, security or approvals, or data handling.
- Make each question concrete: quote the two conflicting statements, put a recommended option first, and say what each option leads to.
- Do not ask what the documents already answer, and do not ask about low-impact details. List those as open items in the brief instead.
- If the user is not present (unattended run), take the most reasonable reading, label it an assumption in the brief, and do not guess on irreversible or product-wide changes.

After the answers, **check what each decision implies** and raise any consequence the user may not have noticed (for example: a decision that changes today's behavior for existing customers, adds a data download to every device, or contradicts a "must not change" section). Record these as follow-ups (N#) and ask again if they need a decision. Do not lecture; state the consequence once.

If an answer is ambiguous, restate your interpretation as an assumption in the brief.

### 6. Write the brief

Write a markdown file named `<feature>-prototype-brief.md` with these sections:

1. **How to use the inputs**: the precedence rule, and that the prototype is a demo, not a spec. Items marked DECIDED are settled; the rest are unconfirmed.
2. **Demo-only elements**: list of scaffolding not to turn into requirements.
3. **Mismatches (M#)**: table with columns: #, topic, prototype, document, handling. Mark resolved items **DECIDED** and state the decision.
4. **Gaps (G#)**: bullets, each phrased as a decision needed in Inception. Mark resolved ones.
5. **Issues inside the document (V#)**: including which are resolved and what wording to fix.
6. **State and behavior inventory**: screens and flows (including reachable and unreachable paths), settings and dependencies, validation rules with exact copy, error and offline states, a timer table, and a list of what the prototype does not show (logic, sync, analytics, backend, audit, other platforms).
7. **Decision log**: dated, numbered, one line each, followed by follow-ups (N#) that the decisions create, with their status.

Keep it factual and scannable. Use tables for comparisons. Quote copy exactly. Do not invent numbers, thresholds or requirements that neither source contains.

### 7. Check, deliver, explain

- Verify every claim against the prototype code or the document text. For reachability or dead-code claims, say they were read from code.
- Deliver the file with SendUserFile.
- Finish with a short summary: what was decided, what is still open, and which edits the **source document** needs so it no longer contradicts the decisions (recommend updating the document itself rather than relying on the brief alone).
- Tell the user to hand the AI-DLC process the prototype, the document and the brief together, and to update the brief if they decide any open items later.

## Rules

- Do not treat demo scaffolding as requirements.
- Do not resolve conflicts silently; every disagreement is either decided by the user or listed as open.
- Do not expand scope: record what the sources say, plus the questions they leave open.
- If the user later changes a decision, update the decision log and every row it touched so the brief stays consistent.
