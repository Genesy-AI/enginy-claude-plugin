---
name: pipeline-analysis
description: >
  Runs a full sales pipeline analysis — pipeline value, stage conversion, stuck deals,
  forecast, rep performance, and sales velocity — starting from your Enginy outreach data
  and extending into your CRM for deal-stage/revenue analysis. Use this skill whenever the
  user wants to analyze their pipeline — even if they just say "analyze my pipeline",
  "how's my pipeline looking", "show me my deals", "analyze my campaign pipeline",
  "forecast this month", or "who's my top rep". Works from Enginy campaigns, CSV, HubSpot,
  or Salesforce. Delivers a markdown analysis by default, with an optional interactive
  dashboard where the client supports artifacts.
version: 1.0.0
---

# Pipeline Analysis

You are an expert sales analyst. The user's pipeline data can come from Enginy (the
outreach → conversation pipeline), and/or a CRM (HubSpot, Salesforce) or a CSV (the
deal → revenue pipeline). Your job is to extract the data, run a full analysis, and deliver
it — as a markdown report by default, or an interactive dashboard where the client supports
artifacts.

Always respond in the user's language.

---

## Phase 1 — Data Ingestion

Start with Enginy (Source A) — it's the source of truth for the outreach-to-conversation
stages. For deal-stage and closed-won revenue analysis, add the user's CRM or a CSV
(Sources B–D). See "The Enginy boundary" below before deciding which sources you need.

### Source A — Enginy (start here)
Enginy owns the top of the pipeline: campaigns, replies, meetings booked, and follow-up
tasks. Pull it in this order:
1. `get_campaigns` — list the campaigns (filter by `status` ACTIVE / PENDING / DRAFT /
   COMPLETED, or `search` by name). Return each campaign's `appUrl`.
2. `get_campaign_analytics` — per campaign (`pathParams.campaignId`, optional
   `startDate` / `endDate`) for sent / open / reply / bounce volumes and daily trend.
3. `get_conversations_analytics` — reply and meeting outcomes across campaigns (filter by
   `campaignIds`, `dateRange`, or `lastMessageSentBy` to isolate prospect-replied threads).
4. `get_tasks` — the manual follow-up load (filter by `status`, `taskType`, `campaignIds`,
   `assignedId`) to see where reps owe follow-ups and where work is piling up.

Treat each campaign as a top-of-funnel "stage set": contacted → opened → replied →
meeting booked. This is the pipeline Enginy can measure directly.

### The Enginy boundary (be honest about it)
Enginy tracks the **outreach → conversation** pipeline: who was contacted, who replied,
who booked a meeting, and what follow-ups are due. It does **not** track deal stages,
amounts, or closed-won revenue. Any analysis of deal value, stage conversion to close,
forecast in dollars, or win rate needs the user's CRM (Source B/C) or a deal CSV
(Source D). Enginy's `EXPORT_TO_CRM` and `SYNC_LEAD_WITH_CRM` actions bridge records
between the two systems — export Enginy contacts into the CRM, or sync the latest CRM
field values back into Enginy — so the same contact can be followed across both pipelines.
State this boundary to the user when they ask for revenue/forecast analysis on
Enginy-only data.

### Source B — CSV
The user pastes or uploads a CSV. Expected columns (flexible naming, normalize on ingest):
- Deal name → `name`
- Stage → `stage`
- Amount / ARR → `amount` (numeric, strip currency symbols)
- Close date → `close_date` (parse to ISO date)
- Owner / Rep → `owner`
- Created date → `created_date` (parse to ISO date)
- Company / Account → `company`
- Probability → `probability` (optional, numeric 0-100)

If column names differ, infer from context. If probability is missing, assign defaults
based on stage name (see Stage Probability Defaults below).

### Source C — HubSpot MCP
Use the HubSpot MCP to fetch open deals (deal-stage / revenue pipeline):
- Fetch deals with properties: `dealname`, `dealstage`, `amount`, `closedate`,
  `hubspot_owner_id`, `createdate`, `hs_probability`, `associated_company`
- Resolve owner IDs to names via the owners endpoint
- Filter: only fetch deals where `pipeline = default` (or ask user which pipeline)
- Normalize to the standard schema above

### Source D — Salesforce MCP
Use the Salesforce MCP to query opportunities (deal-stage / revenue pipeline):
```sql
SELECT Name, StageName, Amount, CloseDate, Owner.Name, CreatedDate,
       Probability, Account.Name
FROM Opportunity
WHERE IsClosed = false
```
Normalize to the standard schema above.

### Stage Probability Defaults
If probability is not provided, use these defaults. Adapt if the user's stages differ:
```
Prospecting / Discovery    → 10%
Qualification              → 20%
Demo / Meeting scheduled   → 30%
Proposal sent              → 50%
Negotiation                → 70%
Contract sent              → 85%
Closed Won                 → 100%
Closed Lost                → 0%
```

---

## Phase 2 — Data Validation

Before analysis, flag any data quality issues inline (don't block the analysis):
- Deals with no amount → flag as "amount missing", exclude from value calculations
- Deals with close date in the past and still open → flag as "overdue"
- Deals with no owner → group under "Unassigned"
- Negative amounts → flag and exclude
- Duplicate deal names → flag, keep both

Report flagged records as a small warning section at the top of the dashboard.

---

## Phase 3 — Run the Analysis

Compute the following metrics from the normalized data:

### 3.1 Pipeline Overview
- **Total pipeline value** — sum of all open deal amounts
- **Weighted pipeline value** — sum of (amount × probability) for each deal
- **Deal count** — total number of open deals
- **Average deal size** — total value / deal count
- **Median deal size**
- **Pipeline coverage ratio** — total pipeline / monthly quota (ask user for quota if not provided, or skip)

### 3.2 Stage Breakdown
For each stage:
- Deal count
- Total value
- Weighted value
- Average deal size
- % of total pipeline value
- Conversion rate stage-to-stage (if historical closed/lost data is available)

### 3.3 Stuck Deals (At-Risk)
A deal is **stuck** if:
- It has been in the current stage for more than 2× the average time in that stage, OR
- Close date is more than 14 days in the past and still open, OR
- Created more than 90 days ago and still in an early stage (Prospecting / Qualification)

For each stuck deal, surface:
- Deal name + company
- Stage
- Days in current stage
- Amount
- Owner
- Recommended action (based on stage — see Action Templates below)

### 3.4 Forecast
Group deals by close date into:
- **This month** — sum of weighted values closing this calendar month
- **Next month** — sum of weighted values closing next calendar month
- **This quarter** — sum of weighted values closing this quarter
- **Beyond** — everything else

For each period: deal count, weighted value, raw value, and list of top 5 deals by amount.

Flag deals closing this month with probability < 30% as "at risk of slipping".

### 3.5 Rep Performance
For each owner / rep:
- Deal count
- Total pipeline value
- Weighted pipeline value
- Average deal size
- Number of stuck deals
- Deals closing this month (count + value)
- Rank by weighted pipeline value

### 3.6 Sales Velocity
- **Average sales cycle length** — average days from created_date to today for open deals
  (use closed_won deals if available for a more accurate figure)
- **Average days per stage** — mean time spent in each stage across all deals
- **Velocity by rep** — average cycle length per owner
- **Deals at risk of missing close date** — close date within 7 days, probability < 50%

---

## Phase 4 — Present the Analysis

**Default output is a structured markdown analysis** — the KPIs, stage/at-risk/forecast/rep/
velocity tables, and the priority actions from Phase 5, written inline. This works in every
client.

**Optional: interactive dashboard.** Where the user asked for it AND the client supports
artifacts, render a single interactive React artifact with the structure below. Use Recharts
for all charts and Tailwind utility classes for layout. It must work with the actual computed
data — no mock data. If the client can't render artifacts, deliver the markdown analysis
instead; don't refuse.

### Dashboard Layout

```
┌─────────────────────────────────────────────────────┐
│  HEADER: Pipeline Analysis — [date range] — [source] │
│  Data quality warnings (if any)                      │
├──────────┬──────────┬──────────┬────────────────────┤
│ KPI Card │ KPI Card │ KPI Card │ KPI Card           │
│ Total    │ Weighted │ Deals    │ Avg Deal Size      │
│ Pipeline │ Pipeline │ Count    │                    │
├──────────┴──────────┴──────────┴────────────────────┤
│ TABS: Overview │ Stages │ At-Risk │ Forecast │ Reps │ Velocity │
├─────────────────────────────────────────────────────┤
│ [Tab content — charts + tables]                      │
└─────────────────────────────────────────────────────┘
```

### Tab Contents

**Overview tab**
- Bar chart: pipeline value by stage
- Pie/donut chart: deal count by stage
- Summary table: stage name | deals | value | weighted value | % of pipeline

**Stages tab**
- Funnel visualization: deals flowing stage to stage
- Table: stage | count | total value | avg deal size | avg days in stage

**At-Risk tab**
- Summary: X stuck deals, $Y at risk
- Table with colored risk indicators:
  - Red: close date overdue
  - Orange: stuck > 2× avg stage time
  - Yellow: early stage > 90 days
- Columns: deal | company | owner | stage | days stuck | amount | reason | recommended action

**Forecast tab**
- Grouped bar chart: weighted vs raw value per period (this month / next month / quarter / beyond)
- Table per period: deals closing, count, weighted value
- "At risk of slipping" list highlighted in orange

**Reps tab**
- Horizontal bar chart: weighted pipeline by rep
- Table: rep | deals | total value | weighted value | avg deal size | stuck deals | closing this month

**Velocity tab**
- Bar chart: avg days per stage
- Table: rep | avg cycle length | deals closing this month | at-risk deals

### Styling rules
- Use a clean, professional color palette: blues and greens for positive metrics,
  orange/red for at-risk items
- KPI cards: large number, label, and a subtle trend indicator if comparable data exists
- Tables: sortable columns (click header to sort), alternating row colors
- All monetary values formatted as currency (€ or $ based on user's data)
- Dates formatted as DD/MM/YYYY for European users, MM/DD/YYYY for US

---

## Phase 5 — Recommendations

After the dashboard, output a short prioritized action list (max 8 items) in this format:

```
## Priority Actions

1. [URGENT] Deal X (Company Y) — close date passed 12 days ago, $45K at risk.
   → Owner: follow up today, update stage or mark lost.

2. [THIS WEEK] 3 deals stuck in Proposal Sent for 30+ days.
   → Re-engage them through a follow-up campaign in Enginy (use one of the campaigns
     returned by `get_campaigns`, or create a new one — never reference a campaign name
     that isn't in the user's workspace).

3. [FORECAST] Pipeline coverage for this month is 1.2× quota — below the 3× healthy ratio.
   → Prioritize moving 5 deals from Demo to Proposal this week.
```

Tailor recommendations to what's visible in the data. Never invent metrics not present.

---

## Action Templates by Stage

Use these when recommending actions for stuck deals:

| Stage | Recommended Action |
|---|---|
| Prospecting | Re-qualify or disqualify — no activity in 30+ days suggests poor fit |
| Qualification | Schedule a discovery call — ask 3 BANT questions to move forward |
| Demo scheduled | Send pre-demo prep email + confirm attendance |
| Proposal sent | Send a follow-up sequence, offer to answer objections on a call |
| Negotiation | Escalate to manager or offer a limited-time incentive |
| Contract sent | Direct call to legal/finance contact to unblock signature |

---

## Handling Missing Data Gracefully

- If `probability` is missing: derive from stage using defaults — note this in the dashboard header
- If `created_date` is missing: skip velocity calculations — note this
- If only one rep: skip rep performance tab, merge into overview
- If no close dates: skip forecast tab — note this
- Never crash or refuse to analyze — always produce the best analysis possible with available data

---

## Enginy MCP tools used

- `get_campaigns` — list campaigns (filter by `status` / `search`); the outreach stage sets
- `get_campaign_analytics` — per-campaign sent / open / reply / bounce volumes and daily trend
- `get_conversations_analytics` — reply and meeting outcomes across campaigns
- `get_tasks` — the manual follow-up load per rep / campaign
- `start_an_actions_run` — `EXPORT_TO_CRM` / `SYNC_LEAD_WITH_CRM` to bridge Enginy contacts and the user's CRM
- `get_actions_run_status` — poll an export/sync run until it reaches a terminal status
- `get_credit_pricing` / `get_credit_balance` — check cost before any billable action run

---

## Important Notes

- **Enginy covers outreach → conversation, not closed-won revenue.** Deal stages, amounts,
  forecast in currency, and win rate need the user's CRM (HubSpot / Salesforce) or a deal
  CSV. State this boundary whenever the user asks for revenue analysis on Enginy-only data.
- **`EXPORT_TO_CRM` and `SYNC_LEAD_WITH_CRM` need a connected CRM integration** in the
  workspace. They don't expose a credit-mapped price entry, but confirm intent with the
  user before running, and poll `get_actions_run_status` until terminal.
- **`get_conversations_analytics` is rate-limited** to 10 requests/minute — batch campaign
  IDs into one call rather than looping per campaign.
- **Always surface `appUrl` fields** from `get_campaigns`, `get_campaign_analytics`, and
  `get_tasks` so the user can open the campaign or task in Enginy.
- **Artifact output requires host support.** The React dashboard is optional — default to
  the markdown analysis, which works everywhere.
- **Never invent a campaign name, metric, or deal.** Only reference campaigns returned by
  `get_campaigns` and metrics present in the data.

---

## Examples

**Example 1 — Enginy-only outreach pipeline**
User: "How's my outbound pipeline looking?" with no CRM connected. → `get_campaigns` →
`get_campaign_analytics` per campaign → `get_conversations_analytics` for replies/meetings →
`get_tasks` for follow-up load → deliver a markdown analysis of the contacted → replied →
meeting funnel → note that closed-won/revenue needs a CRM.

**Example 2 — Full pipeline (Enginy + CRM)**
User has HubSpot connected and wants a revenue forecast. → Pull Enginy top-of-funnel
(Source A) → pull open deals from HubSpot (Source C) → run stage breakdown, forecast, rep
performance, velocity → render the dashboard artifact (client supports it) → priority
actions.

**Example 3 — Bridge Enginy replies into the CRM**
User wants replied-contacts pushed to Salesforce. → `get_conversations_analytics`
(`lastMessageSentBy: CONTACT`) to find replied threads → confirm with the user →
`start_an_actions_run` (`EXPORT_TO_CRM`) on those contacts → poll `get_actions_run_status`
→ then run the deal-stage analysis from the CRM.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| User asks for revenue/forecast on Enginy-only data | Explain the boundary — Enginy has outreach→conversation, not deal value; ask for CRM access or a deal CSV |
| Client can't render a React artifact | Deliver the markdown analysis (the default) instead |
| `get_conversations_analytics` hits a 429 / rate limit | It's capped at 10 req/min — pass all `campaignIds` in one call rather than looping |
| Campaign analytics look empty | Check the `startDate` / `endDate` window and the campaign `status` — a DRAFT campaign has no sends |
| `EXPORT_TO_CRM` / `SYNC_LEAD_WITH_CRM` fails | The workspace has no connected CRM integration — connect one before bridging records |
| Reps in Enginy don't match CRM owners | Use `SYNC_LEAD_WITH_CRM` (optionally `fullResync`) to re-match records, then reconcile owner names |

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
