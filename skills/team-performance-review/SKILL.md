---
name: team-performance-review
description: Compare reps and senders on outreach activity and outcomes using live Enginy data — pulls per-identity campaign analytics, conversation outcomes, and task discipline into a side-by-side scorecard with coaching notes. Use when asked "compare my reps", "how is my team doing", "who's my top performer", "which SDR is underperforming", "team performance review", "rep scorecard", "who has the most overdue tasks", "compare sender performance", or "how do my senders stack up". Fetches identities, task owners, campaigns per sender, campaign analytics, conversation outcomes, and task counts. Honest about the CRM boundary — Enginy sees outreach and tasks, not closed-won revenue.
metadata:
  version: "1.0.0"
---

# Team Performance Review

## Role & goal

You are a sales-ops analyst producing a rep/sender scorecard from the user's Enginy account. Your job: build the roster, pull each rep's outreach volume, reply/positive outcomes, and task discipline, lay them side by side, and give per-rep coaching notes that route each weakness to the sibling skill that fixes it. Be honest about the boundary: Enginy sees outreach identities and tasks, **not** CRM quota or closed-won revenue — pair with the CRM for true attribution (see Important Notes).

Always return the `appUrl` fields from responses.

---

## Instructions

### Phase 1 — Build the roster

1. List sending identities with `get_identities` (paginate) — these are the senders/reps you can attribute outreach to. Capture ID, name, `appUrl`.
2. Call `get_task_owners` to get the CRM owners tasks are assigned to. Map owners to identities where they represent the same person (names/emails); note where they don't line up cleanly.

### Phase 2 — Per-identity outreach outcomes

1. For each sender, find their campaigns with `get_campaigns` filtered by `identitySenderId`.
2. Aggregate `get_campaign_analytics` across that sender's campaigns → total volume, replies, bounces (read only fields the response returns).
3. Pull `get_identity_performance_metrics` per identity for sending volume/bounce context over the window.
4. For outcome quality, call `get_conversations_analytics` scoped by `identityIds` (and `dateRange`). Use `lastMessageSentBy: CONTACT` as a reply proxy and `conversationTags` for tagged positive/meeting outcomes where the team tags them.

### Phase 3 — Task discipline per owner

1. Call `get_task_status_counts` per owner (`ownerId`) — this returns the **due, skipped, and upcoming** buckets only.
2. For **completed** counts, call `get_tasks` with `status: completed` (and `ownerId`, date range) and read the total; "overdue" maps to the `due` bucket. Use `get_task_pending_count` for a quick pending snapshot.
3. Derive a task-completion signal per rep: completed vs. skipped/overdue.

### Phase 4 — Comparison table

Build a side-by-side scorecard, one row per rep:

| Rep | Volume sent | Reply rate | Positive/tagged outcomes | Task completion | Overdue/skipped |
|---|---|---|---|---|---|

Rank by the metric the user cares about (default: reply rate, then positive outcomes). Flag the top performer and the laggards with the deltas.

### Phase 5 — Coaching notes per rep

For each rep, translate the numbers into one honest read and route the fix:

- **Low reply rate despite good volume** → copy/messaging problem → **copywriting-analyzer**.
- **Low positive outcomes despite decent replies** → wrong targeting/ICP → **build-targeted-lead-list**.
- **High overdue/skipped tasks** → follow-up discipline problem → task hygiene (clear the `due` queue, stop skipping steps); if their sequence structure sets them up to fail, route to **outbound-campaign-architect**.
- **Low volume** → activity/capacity issue → coach on cadence; check whether campaigns are even assigned to that sender.
- **Strong across the board** → document what they do differently (channel mix, step count from `get_a_single_campaign`) and propagate it.

### Phase 6 — Honest framing

Present the scorecard as an **outreach-and-task** view, not a revenue view. Offer a manual re-run cadence (e.g. monthly) — no auto-scheduling.

---

## Enginy MCP tools used

- `get_identities`
- `get_task_owners`
- `get_campaigns`
- `get_campaign_analytics`
- `get_identity_performance_metrics`
- `get_conversations_analytics`
- `get_task_status_counts`
- `get_task_pending_count`
- `get_tasks`
- `get_a_single_campaign` (optional — read a top performer's sequence structure to propagate)

---

## Important Notes

- **CRM boundary.** Enginy attributes **outreach activity and tasks**, not quota attainment, pipeline, or closed-won revenue. Never present this scorecard as a revenue ranking — pair it with the user's CRM for true attribution. Say so explicitly in the output.
- **Task counts split across two tools.** `get_task_status_counts` returns only **due / skipped / upcoming**. There is no "completed" or "overdue" field there — completed comes from `get_tasks` with `status: completed`, and overdue is the `due` bucket. Don't claim a completed count from `get_task_status_counts`.
- **Owner ≠ identity automatically.** Task owners (CRM) and sending identities are separate concepts and may not map 1:1. State your mapping assumptions and flag reps you couldn't cleanly join.
- **Positive-outcome data is tag-dependent.** No native "meetings booked" metric — rely on `conversationTags` (if the team tags) or `lastMessageSentBy: CONTACT` as a reply proxy.
- **Metric fields:** read only what the live response returns; do not invent per-rep metrics the schema doesn't expose.
- **Read-only skill.** This review reads data and coaches; it does not change campaigns, tasks, or send anything.
- Full platform docs: https://docs.enginy.ai

---

## Examples

**1. "Compare my three SDRs this month."**
→ `get_identities` + `get_task_owners` to build roster → per sender `get_campaigns(identitySenderId)` + aggregated `get_campaign_analytics` + `get_conversations_analytics(identityIds, lastMessageSentBy CONTACT)` → per owner `get_task_status_counts` + `get_tasks(status completed)`. Scorecard shows Rep A top at 5.8% reply, Rep C at 1.9% with 40 overdue tasks. Coaching: A's copy/targeting is the template; C has a follow-up discipline problem — clear the due queue, then revisit copy via **copywriting-analyzer**.

**2. "Who's my top performer and why?"**
→ Build the table, rank by reply + positive outcomes, then read the leader's sequence with `get_a_single_campaign` to explain the edge (3-step LinkedIn-first) and recommend propagating it via **outbound-campaign-architect**.

**3. "Which rep has the worst task discipline?"**
→ `get_task_status_counts` per owner for due/skipped, `get_tasks(status completed)` for completed. Report the rep with the highest overdue+skipped ratio; note this is task follow-through, not outcomes, and pair with reply data before judging overall performance.

---

## Troubleshooting

| Symptom | Likely cause | What to do |
|---|---|---|
| Owners don't map to identities | CRM owners and senders are distinct sets | Match on name/email, state assumptions, flag unmatched reps |
| `get_task_owners` returns empty | No CRM connected / no owners configured | Fall back to `assignedId` on `get_tasks`; note task attribution is limited |
| No completed count in status counts | By design — only due/skipped/upcoming | Use `get_tasks` with `status: completed` for the total |
| Sender shows zero volume | No campaigns assigned to that `identitySenderId` | Confirm campaign assignment before calling it a low-activity rep |
| Positive-outcome column empty | Conversations aren't tagged | Use reply proxy (`lastMessageSentBy: CONTACT`); recommend tagging + CRM pairing |
| Tool rejected for permissions | OAuth re-running / missing scope | Run `mcp_whoami`; see https://docs.enginy.ai/mcp/security-troubleshooting |

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
