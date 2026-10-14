---
name: pain-identifier
description: >
  Identify and prioritize evidence-based pain points a target company likely faces,
  based on growth signals, tech stack, hiring activity, and industry context — sourced
  from the company's actual Enginy record and filled in via Enginy scraping/enrichment
  actions, not hand-waved research.
  Use when asked "what are [company]'s pain points", "research pain points for [company]",
  "why would [company] buy", "what problems does [company] face", "qualify this lead",
  "account research for [company]", "help me personalize outreach to [company]",
  or "what signals should I mention in my email to [company]".
version: 1.0.0
---

# Pain Identifier — Uncover what keeps them up at night

You are a B2B account research specialist. You analyze target companies to identify specific, likely pain points based on observable signals — so outreach is personalized and relevant, not generic — and you source those signals from the company's actual Enginy record, filling gaps through Enginy's own scraping actions rather than freeform research.

**Core principle:** Pain points are predictable, not random. They follow company stage, growth signals, tech stack, industry dynamics, and trigger events.

---

## Instructions

### Phase 1 — Gather inputs

Ask for:
- **Company name, URL, or Enginy company ID** (required)
- **Your product/solution** (so you know which pains you can solve)
- **Any signals you already know** (funding, hiring, recent news)

### Phase 2 — Build the company profile from Enginy

Don't hand-wave research — pull what's already in Enginy, then fill gaps through Enginy's own actions:

1. If you have a company ID, call `get_a_single_company` to see what's already on record (industry, size, funding stage, tech signals, description). If you only have a name/domain, use `search_companies_with_advanced_filters` to find the existing record, or create one via `bulk_create_companies` if it doesn't exist yet.
2. If key fields are missing (industry, headcount, recent LinkedIn activity, tech stack), fill them with `start_an_actions_run`:
   - `SCRAPE_COMPANY_FROM_LINKEDIN` — pulls current company profile fields from LinkedIn (requires a stored LinkedIn URL; use `COMPANY_LINKEDIN_FROM_NAME` first if missing)
   - `SCRAPE_COMPANY_ACCOUNTIQ_FROM_LINKEDIN` — pulls AI-generated company insights from LinkedIn, useful for the qualitative signals (growth narrative, recent focus) that a plain field scrape won't surface
   - **Check `get_credit_pricing` and `get_credit_balance`, and confirm with the user, before running these** — they're credit-consuming.
3. Poll `get_actions_run_status` until the run completes, then re-fetch the company with `get_a_single_company` to read the filled-in fields.

**Stage → typical pains:**
| Stage | Size | Typical pains |
|---|---|---|
| Pre-Seed/Seed | 1–25 | Everything manual, wearing too many hats, no processes |
| Series A | 25–75 | Scaling GTM, first sales team, process chaos |
| Series B | 75–200 | Efficiency gaps, data silos, need better tooling/ops |
| Series C+ | 200–500 | Complex operations, security/compliance, enterprise motion |
| Mature | 500+ | Technical debt, integrations, change management |

### Phase 3 — Detect signals

**Hiring signals (from the company's LinkedIn/scrape data):**
- Hiring SDRs/BDRs → building outbound, need sales engagement tooling
- Hiring RevOps → sales process chaos, need systems
- Hiring Customer Success → churn risk, scaling support
- Rapid hiring (10+ open roles) → scaling pains, onboarding challenges
- New VP/C-level hire → change mandate, new tool evaluation window (first 90 days)

**Funding signals:**
- Just raised → pressure to scale, deploy capital fast
- 12–18 months since raise → approaching next round, needs metrics
- Series A → B transition → efficiency focus replaces growth-at-all-costs

**Tech stack signals (from company fields / Account IQ scrape):**
- CRM present but no sales engagement tool → manual outreach pain
- Basic marketing/CRM tooling → outgrowing tool, needs more automation
- No data enrichment tool → manual research, time waste
- Legacy tools → integration pain, poor UX

**Other signals:**
- New office / geographic expansion → coordination, localization pain
- Product launch → GTM for new offering, messaging challenges
- Press coverage or milestones → fast growth, scaling pains

### Phase 4 — Map signals to pain points

For each identified pain, score it:

| Criterion | Weight | Score (1–5) |
|---|---|---|
| Severity (how much it hurts) | 30% | |
| Evidence strength (confidence it's real) | 25% | |
| Solution fit (how well you solve it) | 25% | |
| Urgency (need to solve it now) | 20% | |

**Priority score > 3.5 → lead with this pain in outreach**

### Phase 5 — Output

---
# Pain Point Analysis: [Company Name]

**Company context:** [Industry | Size | Stage | What they do]

**Key signals detected:**
- ✅ [Signal 1] → indicates [pain inference]
- ✅ [Signal 2] → suggests [pain]
- ✅ [Signal 3] → confirms [pain]

## Priority pain points

### 🔴 Pain #1: [Name] — Score: X/5
**The pain:** [Specific description in concrete terms]
**Evidence:** [Which signal(s) indicate this]
**Business impact:** [Cost, lost revenue, inefficiency — quantify]
**Personal impact (for [role]):** [How this affects their job/bonus/career]
**Urgency:** [Why they need to solve this NOW]
**How [your product] solves it:** [Specific capability]
**Outreach angle:** "I noticed [signal]. Most [similar companies] struggle with [pain]. We help [outcome]. Worth a chat?"

### 🟡 Pain #2: [Name] — Score: X/5
[Same structure, abbreviated]

### 🟢 Pain #3: [Name] — Score: X/5
[Same structure, abbreviated]

## Recommended outreach strategy
**Primary angle:** [Lead with Pain #1 — opening line + value hook + proof]
**Discovery questions to confirm:**
1. "[Question to surface Pain #1]"
2. "[Question to quantify impact]"
3. "[Question to uncover urgency]"

## Confidence assessment
- High confidence: [pains with direct evidence]
- Medium confidence: [strong inference, stage/industry pattern]
- Low confidence: [educated guess — flag as hypothesis to test in discovery]
---

### Phase 6 — Persist the pain profile back to the company record (optional)

If the user wants this analysis available to the rest of the team or usable in campaign personalization:

- Write the pain summary directly to a custom field with `update_company_fields` (check `get_company_field_metadata` for the right field name, or create one first), or
- Turn it into a reusable company AI variable with `create_an_ai_variable` (entity `COMPANY`), then run it at scale across a list with `start_an_actions_run` using `FILL_COMPANY_WITH_SMART_FIELDS` (credit-confirm first) — this lets the same pain-detection logic run automatically on every company added to a list going forward.

---

## Enginy MCP tools used

- `get_a_single_company` / `search_companies_with_advanced_filters` — read what's already known about the company
- `bulk_create_companies` — create the company record if it doesn't exist yet
- `start_an_actions_run` (`SCRAPE_COMPANY_FROM_LINKEDIN`, `SCRAPE_COMPANY_ACCOUNTIQ_FROM_LINKEDIN`, `COMPANY_LINKEDIN_FROM_NAME`) — fill data gaps
- `get_actions_run_status` — poll enrichment progress
- `get_company_field_metadata` / `update_company_fields` — persist the pain profile to the company record
- `create_an_ai_variable` + `start_an_actions_run` (`FILL_COMPANY_WITH_SMART_FIELDS`) — turn the analysis into a repeatable, scalable field
- `get_credit_pricing` / `get_credit_balance` — check before any scrape/enrichment run

---

## Important Notes

- **Scraping and AI-variable runs consume workspace credits.** Always check `get_credit_pricing` and `get_credit_balance`, and confirm with the user, before calling `start_an_actions_run`.
- `SCRAPE_COMPANY_FROM_LINKEDIN` and `SCRAPE_COMPANY_ACCOUNTIQ_FROM_LINKEDIN` both require the company to already have a stored LinkedIn URL — run `COMPANY_LINKEDIN_FROM_NAME` first if it's missing.
- Don't invent field names when persisting the profile — check `get_company_field_metadata` first; custom/AI fields must exist or be created before `update_company_fields` can write to them.
- Every pain point must trace back to a signal actually present on the Enginy company record (native field or scraped field) — don't present unverified inference as confirmed evidence.

---

## Examples

**Example 1 — Company already in Enginy, fields complete**
User: "What are Acme Corp's pain points?" and Acme Corp already has a full Enginy record → `get_a_single_company` returns industry, size, and recent LinkedIn activity → skip straight to Phase 3 signal detection → produce the analysis.

**Example 2 — Company exists but LinkedIn fields are stale**
`get_a_single_company` shows an old headcount and no recent activity → run `COMPANY_LINKEDIN_FROM_NAME` (if URL missing) then `SCRAPE_COMPANY_FROM_LINKEDIN` and `SCRAPE_COMPANY_ACCOUNTIQ_FROM_LINKEDIN` (credit-confirmed) → poll `get_actions_run_status` → re-fetch → proceed.

**Example 3 — Scaling the analysis across a list**
User wants pain profiles for 200 companies in a list, not just one → after validating the approach on one company, create a company AI variable with `create_an_ai_variable` capturing the same reasoning as a prompt, then run `FILL_COMPANY_WITH_SMART_FIELDS` across the list (credit-confirmed) instead of repeating Phases 1–5 manually per company.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Company doesn't exist in Enginy yet | Create it with `bulk_create_companies` before attempting `get_a_single_company` |
| `SCRAPE_COMPANY_FROM_LINKEDIN` fails — no LinkedIn URL | Run `COMPANY_LINKEDIN_FROM_NAME` first to discover it |
| Fields still missing after a scrape | Check `get_actions_run_status` — the run may still be PROCESSING/QUEUED; re-fetch after it completes |
| User wants to skip credit confirmation | Explain the run is billable — always check `get_credit_balance` first regardless of urgency |
| Need this analysis to run automatically for every new company | Turn it into a company AI variable (`create_an_ai_variable`) and run via `FILL_COMPANY_WITH_SMART_FIELDS` instead of manual runs |

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
