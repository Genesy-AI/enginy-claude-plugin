---
name: pre-call-research-brief
description: >-
  Assemble a tight pre-call research brief on a contact and their company before
  a sales call or meeting. Use when the user says "prep me for my call with X",
  "build a pre-call brief", "research this prospect before my meeting", "what do
  I need to know before I talk to this account", "give me talking points for this
  contact", or "get me ready for the demo with this company".
version: 1.1.0
---

# Pre-Call Research Brief

## Role and goal

Before a rep gets on a call, give them everything they need on one screen: who
the person is, what the company does, the full history of what's been said
between you, the likely pains, concrete talking points, objection prep, and a
suggested next step. Pull relationship history from Enginy, fill only the gaps
that matter for the call, and optionally leave the brief on the record as a task
note. Keep it short enough to read in the two minutes before dialing.

**The meeting-existence rule.** A call only earns its slot on the calendar if it
delivers something the prospect couldn't get from an email — a fix, a decision,
an idea, or a sharper plan. If the research doesn't surface that value (no open
question worth talking through live, nothing concrete to walk them through),
say so plainly in the brief and suggest handling it async by email instead of
defaulting to a call.

## Instructions

### Phase 1 — Identify contact and company
1. Resolve the contact: `get_a_single_contact` by ID, or `get_contacts` /
   `search_contacts_with_advanced_filters` by name/email. Note the associated
   company ID.
2. `get_a_single_company` for firmographics. Capture `appUrl` for both so the
   brief links straight into Enginy.
3. Check the meeting type. Strategic/planning meetings — QBRs, quarterly
   business reviews, success reviews — aren't a fit for this skill; route those
   to **quarterly-outbound-review** instead. This skill is built for tactical
   sales/demo calls, one contact and one agenda at a time.

### Phase 2 — Fill gaps only if the record is thin
1. If key fields for the call are missing (role, seniority, company size,
   industry, recent company context), consider `SCRAPE_LEAD_FROM_LINKEDIN` and
   `SCRAPE_COMPANY_ACCOUNTIQ_FROM_LINKEDIN` (AI company insights).
2. These are **billable**. Call `get_credit_pricing` + `get_credit_balance`,
   show the cost, and confirm before `start_an_actions_run`. Poll
   `get_actions_run_status` to completion. Skip this phase entirely when the
   record is already rich enough for the call — don't spend to gold-plate a
   brief.

### Phase 3 — Relationship history
1. `get_inbox_contact_messages` (contactId) for the flattened message history —
   what you've sent and what they replied. Use `get_conversation_messages` for a
   specific thread if needed.
2. `get_tasks` filtered by `companyIds` (and search by contact) for open/pending
   items and past activity.
3. Check which campaigns the contact is in (from the contact record / campaign
   membership) so you know what messaging they've already received — don't
   repeat an angle they've seen.

### Phase 4 — Fresh external context (optional)
1. For recent news, funding, launches, or leadership changes not in Enginy, use
   web search. **Host requirement: this phase needs web search available in the
   running client.** If it isn't, say so and build the brief from Enginy data
   only — do not fabricate news.

### Phase 5 — Assemble the brief

Before writing anything, apply the meeting-existence rule from above: if
nothing in Phases 1-4 turned up a concrete fix, decision, insight, or plan
worth the prospect's time live, say so and recommend async handling instead of
prepping a call that shouldn't happen.

Prep the brief for the call's actual shape, not just its content. Default to a
30-minute meeting run as:
- **0-5 min** — intro and agenda.
- **5-15 min** — their questions and doubts.
- **15-30 min** — the Value block: the prepared segment where you bring
  something they didn't have before the call.

**Never walk in empty.** The brief's output must always include a prepared
Value block — not just talking points — so the rep has a relevant insight, a
concrete analysis, or a best practice matched to their situation ready to run
for that back half of the call.

Produce a tight, skimmable brief:
- **Who they are** — name, role, seniority, tenure, notable background.
- **Company snapshot** — what they do, size, industry, any fresh context.
- **Engagement history** — messages exchanged, last touch, current campaign(s),
  open tasks. Quote the last thing *they* said if there is a reply.
- **Likely pains** — route to the `pain-identifier` doctrine to reason from
  role + company to probable problems. Frame as hypotheses, not facts.
- **Talking points** — 3-5 specific openers/angles tied to the pains and history,
  for the 5-15 min segment.
- **Value block** — the prepared 15-30 min segment: one relevant insight,
  concrete analysis, or best practice matched specifically to this prospect's
  situation. This is the part of the brief that makes the call worth having —
  don't skip it even when talking points feel sufficient.
- **Objection prep** — anticipated pushback and responses; reference the
  `reply-handler` doctrine for handling live replies.
- **Suggested next step** — the concrete ask for this call.

### Phase 6 — Persist and hand off (optional)
1. If the user wants it saved, `create_task` on the contact (`leadId`) or company
   (`companyId`) with the brief in `notes`, a `dueDate` for the call, and
   `type`/`subject`. Return the task `appUrl`.
2. Return `build_inbox_link` (scoped to the contact via `leadId`) so they can
   jump to the live thread, plus the contact and company `appUrl`s.

## Enginy MCP tools used

get_a_single_contact, get_contacts, search_contacts_with_advanced_filters,
get_a_single_company, get_credit_pricing, get_credit_balance,
start_an_actions_run, get_actions_run_status, get_inbox_contact_messages,
get_conversation_messages, get_tasks, create_task, build_inbox_link

## Important notes

- **Confirm before spend.** Phase 2 scraping is billable; always price and
  confirm before `start_an_actions_run`. Most briefs need no enrichment — skip
  it when the record is already sufficient.
- **No value, no call.** If the brief can't identify something the prospect
  couldn't get from an email, flag it and suggest async instead of prepping a
  call anyway. Strategic/planning meetings (QBRs, success reviews) route to
  `quarterly-outbound-review`, not this skill.
- **Always ship a Value block.** A brief that's only talking points is
  unfinished — the 15-30 min segment needs a prepared insight, analysis, or
  best practice, not improvised commentary.
- **Web research is host-gated.** If the client has no web search, build from
  Enginy data and say the external-news section is unavailable. Never invent
  news, funding, or headlines.
- **Pains and objections are hypotheses.** They come from role/company reasoning
  (`pain-identifier`), not confirmed facts — label them as such.
- **Don't repeat sent messaging.** Read campaign membership and inbox history so
  talking points don't echo what the prospect already ignored.
- **`build_inbox_link` builds a URL only** — it does not fetch messages. Use
  `get_inbox_contact_messages` for the actual history.
- UI walkthroughs: https://docs.enginy.ai.

## Examples

**1. Warm demo prep.** User has a demo in 20 minutes with a contact ID. Pull
contact + company, read the inbox thread (they replied "interested, but worried
about onboarding time"), see they're in the "Q3 Outbound" campaign. No
enrichment needed. Brief leads with the onboarding objection, gives 3 talking
points on time-to-value, and suggests booking a technical follow-up. Save as a
task due today, return the inbox link.

**2. Cold-ish account, thin record.** Contact has only name + company. Confirm
~X credits for `SCRAPE_LEAD_FROM_LINKEDIN` + `SCRAPE_COMPANY_ACCOUNTIQ_FROM_LINKEDIN`,
run, poll. Add web-search news (recent funding round). Brief covers who they are,
the raise as an opener, likely scaling pains, and a discovery-call ask.

**3. No web search available.** Same as above but the client lacks web search.
Build the full Enginy-based brief and flag: "External news section skipped — web
search isn't enabled here."

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `get_inbox_contact_messages` returns nothing | No thread, or history is archived | Retry with `archived: true`; the contact may be truly cold |
| Company fields sparse after scrape | LinkedIn page thin or no URL on record | Add a company LinkedIn URL first, or note the gap in the brief |
| Web-search step unavailable | No web search in the client | Build from Enginy data; flag the missing section |
| `create_task` 400 | Missing required `type`/`subject` | Provide both; attach brief in `notes` |
| Duplicate/irrelevant talking points | Didn't read campaign history | Check campaign membership before drafting angles |
| Enrichment stuck | Worker backlog (stale `lastUpdatedAt`) | Keep polling `get_actions_run_status` |

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
