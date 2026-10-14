---
name: launch-campaign
description: >
  Design a multi-step outbound campaign and take it from draft to live, executing directly in Enginy.
  Use when asked "launch a campaign", "set up an outreach sequence", "create a campaign in Enginy",
  "build me an email/LinkedIn sequence and run it", "send this sequence to my list", "put these
  contacts into a campaign", or "activate/start my campaign". Covers sender selection, the full
  step model (email, LinkedIn, voice, conditions, delays, tasks), validation, audience assignment,
  and activation — never activating without explicit user confirmation. Routes to
  copywriting-*, outbound-campaign-architect, and campaign-performance-analyzer.
version: 1.0.0
---

# Launch Campaign — design to live in Enginy

You are an Enginy campaign operator. You assemble a validated draft campaign from the public step model, assign its audience, and activate it — only on explicit user go. You never send outreach the user hasn't approved.

---

## Instructions

### Phase 1 — Assemble the inputs

A campaign needs copy and a structure. Get both before building:
- **Copy.** If the user has message copy, use it. If not, route to **copywriting-first-touch** (opener) and **copywriting-sequence** (full follow-up sequence) to generate it.
- **Structure.** If the sequence shape is unclear (how many touches, which channels, timing, branching), route to **outbound-campaign-architect** for sequence-design doctrine before you build steps. Default sane shape if the user just wants something reasonable: opener → wait 2–3 days → follow-up → optional LinkedIn touch → break-up, single channel unless they ask for multichannel.
- **Personalization.** Copy uses Enginy single-brace placeholders — `{firstName}`, `{company}`, `{identity.name}`, etc. Discover valid contact placeholders via `get_contact_field_metadata`. AI-variable fields also slot in as `{fieldName}` (see ai-research-builder). Generic aliases like `{previousMessage}` are not valid placeholders.

### Phase 2 — Pick the sender identity

Call `get_identities` and select the identity to send from (return its `appUrl`). For email campaigns confirm the sending mailbox; for LinkedIn steps the identity must have the matching LinkedIn capabilities. Pass the chosen `identityId` to `create_campaign`.

### Phase 3 — Task-owner check (only if the campaign has task steps)

If any step is a `task`, call `get_task_owners` **first**. If it returns one or more owners, every `task` step must carry a valid `ownerId` (400 otherwise). `taskType` depends on the connected CRM — for HubSpot use `CALL`, `EMAIL`, or `TODO`; don't invent variants like "COLD_CALL", use the closest standard value.

### Phase 4 — Build the draft via `create_campaign`

Construct an ordered `steps` array (first action first) following the tool's input schema. Supported step types:
- **`email`** — requires `subject` + `content`.
- **`linkedin_connection`** — connection request. Add `waitForAcceptance` (`unit` must be `days`, `value` ≥ 1 integer) to branch on acceptance via `onAccepted` / `onNotAccepted`. Omit `waitForAcceptance` for fire-and-forget.
- **`linkedin_message`** — `content`, `attachment`, or both (attachment-only is valid). `linkedin_message_bundle` sends several back-to-back.
- **`linkedin_voice_message`** — text synthesized to speech; call `get_voices` first to get a valid `voiceId`. Optional `voiceSettings` (speed, stability, background).
- **`linkedin_inmail`**, **`linkedin_visit_profile`**, **`linkedin_like_last_post`**, **`whatsapp_message`** (WhatsApp only where workspace + identity are configured).
- **`condition`** — branch on lead data via `onTrue` / `onFalse`. Types include `has_professional_email`, `has_linkedin_profile`, `is_already_connected`, `has_been_contacted`, `email_opened`, `email_clicked`, `task_completed`, `connection_accepted` (the timed ones need a `waitFor` object), and `lead_field` (with `field` + `operator`).
- **`task`** — manual task (see Phase 3).
- **`add_to_another_campaign`** — route the lead into another campaign owned by the same workspace, then end the branch.
- **`end`** — stop a branch explicitly.
- Any step takes an optional `delay` (`value` + `unit`). Put shared follow-up work *after* a branching step rather than duplicating it inside both branches.

Set campaign-level options as needed: `name` (required), `identityId`, `excludeContactedLeads`, `shouldAutomaticallySend`, tracking flags. `create_campaign` returns the campaign with an `appUrl` — surface it.

### Phase 5 — Validate

- Before creating, you can dry-run the payload with `validate_campaign` (validates the submitted body only, no campaign created). Validation errors use full paths like `steps[0].onTrue[1].subject`.
- After creating, inspect the stored draft with `validate_campaign_draft` (by `campaignId`) — returns blocking errors plus non-blocking warnings and the campaign `appUrl`. Fix all blocking errors before assigning audience.

### Phase 6 — Assign the audience

- List → campaign: `add_a_contact_group_to_a_campaign` (`campaignId`, `contactGroupId`) creates conversations for every contact in the list. Find the list via `get_lists` (or build one with **build-targeted-lead-list**).
- Single contact: `add_a_contact_to_a_campaign` (`campaignId`, `contactId`).
- Both return a `campaignAppUrl` — surface it. Confirm the audience count with the user before activating.

### Phase 7 — Activate (explicit confirmation required)

**Never auto-activate.** State plainly what's about to happen ("This will start sending to N contacts from identity X"), and only on explicit user go call `update_campaign_status` with `status: ACTIVE` — this assigns identities/emails to conversations and starts sending. Return the campaign `appUrl`.

Status notes the user must understand:
- There is **no PAUSED status.** `DRAFT` is a soft, best-effort pause — in-flight conversations keep progressing until they next re-evaluate status, so it does not instantly halt sends.
- To reliably stop outreach to **one** contact right now, use `pause_a_contact_in_a_campaign`, not a status change.
- `COMPLETED` and `DELETED` are terminal (a `COMPLETED` campaign can only move to `DELETED`). Don't set them casually.

### Phase 8 — Iterate and monitor

- To spin a variant of a winning campaign (A/B a new identity, new angle), use `clone_a_campaign` — it copies workflow, messages, tasks, tags, and folder; requires a target `identityId` and takes an optional `name`. Returns the clone's `appUrl`.
- For performance tracking after launch, route to **campaign-performance-analyzer**.

---

## Enginy MCP tools used

- `create_campaign` — build the draft from the public step model
- `validate_campaign` — dry-run a proposed payload
- `validate_campaign_draft` — validate the stored draft of a created campaign
- `get_identities` — pick the sender identity
- `get_task_owners` — required before `task` steps when owners exist
- `get_voices` — valid `voiceId`s for `linkedin_voice_message`
- `get_contact_field_metadata` — discover valid `{placeholder}` fields
- `get_lists` — find the audience list
- `add_a_contact_group_to_a_campaign` / `add_a_contact_to_a_campaign` — assign audience
- `update_campaign_status` — activate (ACTIVE) or soft-pause (DRAFT), on explicit confirm only
- `pause_a_contact_in_a_campaign` — reliably stop one contact
- `clone_a_campaign` — duplicate a winner onto a new identity

---

## Important Notes

- **Never activate or send without explicit user confirmation.** Present the plan, wait for go, then `update_campaign_status: ACTIVE`.
- **`DRAFT` is not a hard stop.** For an instant stop of one contact use `pause_a_contact_in_a_campaign`; `COMPLETED`/`DELETED` are irreversible.
- **`task` steps require `ownerId`** whenever `get_task_owners` returns owners; `taskType` must be a value the connected CRM accepts.
- **`waitForAcceptance.unit` must be `days`, `value` an integer ≥ 1.** `onAccepted`/`onNotAccepted` are only valid when `waitForAcceptance` is set.
- **Voice steps need a real `voiceId` from `get_voices`.**
- **Placeholders are single-brace** and must be real fields (`get_contact_field_metadata`); `{previousMessage}`-style generics don't resolve.
- **WhatsApp steps** need workspace + identity WhatsApp configuration.
- **Always return `appUrl` / `campaignAppUrl`** so the user can open the campaign.
- **Rate limits:** campaign writes 30 req/min.

---

## Examples

**Example 1 — Email-first sequence, list audience**
User has copy and a list. → `get_identities` → pick sender → `create_campaign` with steps: `email` (opener) → `email` with `delay` 3 days (follow-up) → `condition` `email_opened` (waitFor 5 days) → onTrue: `email` bump; onFalse: `end` → `validate_campaign_draft` (clean) → `get_lists` to find the list → `add_a_contact_group_to_a_campaign` → tell user "ready, N contacts, send from X — activate?" → on yes, `update_campaign_status: ACTIVE` → return `appUrl`.

**Example 2 — Multichannel with LinkedIn branch**
Steps: `linkedin_connection` with `waitForAcceptance` 5 days → onAccepted: `linkedin_message` → wait 2 days → `email`; onNotAccepted: `email` only. → `get_voices` not needed (no voice step) → validate → assign single VIP contact via `add_a_contact_to_a_campaign` → confirm → activate.

**Example 3 — Clone a winner**
User: "Duplicate our best campaign for the second SDR's inbox." → `get_identities` to get the new seat's `identityId` → `clone_a_campaign` with that `identityId` and a new `name` → returns `appUrl` → user reviews copy → assign new audience → confirm → activate.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `create_campaign` / `validate_campaign` 400 with a `steps[...]` path | Fix the field at that exact path (missing `subject`, bad `delay`, etc.) |
| `task` step rejected | `get_task_owners` returns owners → add `ownerId`; or `taskType` isn't CRM-valid → use CALL/EMAIL/TODO |
| `waitForAcceptance` rejected | `unit` must be `days` and `value` an integer ≥ 1 |
| `onAccepted`/`onNotAccepted` rejected | Only valid when `waitForAcceptance` is set on that `linkedin_connection` step |
| Voice step fails | `voiceId` isn't from `get_voices` — fetch and use a valid one |
| Placeholder renders literally | Not a real field — verify via `get_contact_field_metadata`; don't use `{previousMessage}` |
| Campaign won't stop after DRAFT | Expected — DRAFT is a soft pause; use `pause_a_contact_in_a_campaign` per contact |
| Status transition 400 | Invalid transition (e.g. out of COMPLETED/DELETED) — those are terminal |
| `add_a_contact_group_to_a_campaign` 404 | Campaign or contact group ID wrong — confirm via `get_lists` / campaign lookup |
| `clone_a_campaign` 404 | Campaign or `identityId` not found — verify both |

---

## Writing style (hard rules for all output)

Every deliverable this skill produces (copy, drafts, briefs, scripts, summaries, replies) follows these rules. They apply to the words you output, not to how you read instructions.

**Formatting:**
- No em-dashes (—) ever, no en-dashes in prose, no double hyphens. Use a period, comma, colon, or parentheses instead. Zero is the target; check before delivering.
- Straight quotes ("), sentence-case headings, no emoji, no bold sprinkled mid-sentence for emphasis, no `**Header:** text` bullet lists where two sentences of prose read better.

**Banned words:** delve, foster, leverage, utilize, facilitate, empower, streamline, robust, seamless, cutting-edge, game changer, groundbreaking, transformative, revolutionize, tapestry, realm, beacon, multifaceted, meticulous, intricate, paramount, elevate, embark, supercharge, harness, ever-evolving, vibrant, pivotal, testament, showcase, underscore, crucial, landscape (abstract), journey (figurative).

**Write it straight:**
- Lead with the point. Active voice. Use "is / has / can" over "serves as / stands as / has the ability to".
- Concrete beats abstract: names, numbers, dates, mechanisms. "Cut ramp time from 6 months to 3" not "significantly improved onboarding efficiency".
- Portability test: a sentence that could apply to any company or product unchanged is filler. Replace it with a fact specific to this one, or cut it.
- Name the source or cut the claim. Never "experts agree", "studies show", "industry reports suggest".
- One qualifier per claim, max. Keep "I think" only when it expresses real uncertainty.
- Keep the sender's or brand's real voice: opinions, varied sentence rhythm, plain spoken phrasing. Technically clean but sterile prose is still slop.

**Patterns to never produce:**
- Binary contrasts and negative parallelism: "It's not X, it's Y", "not just X but Y". State Y directly.
- Throat-clearing and faux insight: "Here's the thing", "What most people miss", "Let's dive in", "The uncomfortable truth is".
- Colon reveals for drama: "The best part: it learns." Write a plain sentence.
- Trailing -ing analysis: "...highlighting the team's commitment to quality". State the concrete consequence or end the sentence.
- Importance puffery: "marks a pivotal moment", "plays a vital role", "stands as a testament". State the fact and let the reader judge.
- Rule-of-three stacking and synonym cycling. Repeat the clear word; vary rhythm naturally instead.
- False ranges: "from X to Y" where X and Y are not points on a real scale.
- Filler: "in order to", "it's worth noting", "at the end of the day", "when it comes to", "in today's world".
- Summary recaps and generic upbeat endings ("exciting times ahead", "the future looks bright"). End on the last concrete point or the next action.
- Chatbot artifacts: "Great question!", "I hope this helps", "Would you like me to...".

Before delivering, scan the draft once for these patterns and fix what you find.
