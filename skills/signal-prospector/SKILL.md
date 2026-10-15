---
name: signal-prospector
description: >-
  Find companies showing buying signals (hiring, tech adoption, funding) and reach
  their decision-makers with signal-aware outreach. Use when the user says "find
  companies that are hiring for X", "find accounts using [technology]", "find
  recently funded companies", "who's showing buying signals in my market",
  "build a list off a trigger event", or "find companies expanding and get me the
  right person to contact".
metadata:
  version: "1.0.0"
---

# Signal Prospector

## Role and goal

Turn a buying signal into a working list of decision-makers. You source
companies that exhibit a signal, import them to a list, find the right people at
each, and hand off to signal-aware outreach where the signal *is* the angle.
Critically, you are honest about which signals Enginy's AI Finder can express
natively versus which need external monitoring — you never promise a signal the
tool can't produce.

## What AI Finder can express today

`preview_an_ai_finder_search` takes a natural-language `text` query (or a saved
filter) plus a `provider`. There is **no structured "signal" field** — a signal
is expressed as phrasing routed to the provider that carries that data. Native
signal coverage:

| Signal | Provider | How to express it |
|---|---|---|
| **Hiring / open roles** | `THEIRSTACK_JOBS` | "companies hiring [role] in [geo]" — sourced from job postings |
| **Technology adoption / stack** | `THEIRSTACK_TECHNOLOGY` | "companies using [technology]" |
| **Funding / investors / new funds** | `CRUNCHBASE_COMPANIES`, `CRUNCHBASE_INVESTORS` | "Series B SaaS companies funded recently" — recency precision depends on what Crunchbase surfaces |
| **Job change (person moved roles)** | `LINKEDIN` | "Heads of Sales who changed jobs in the past 90 days" |
| **Ecommerce platform / store growth** | `STORELEADS` | "Shopify DTC brands with growing traffic" |
| **Local presence** | `GOOGLE_MAPS` | "dental clinics in Austin" |

**Signals NOT natively expressible** (need external monitoring): press/news
events, product launches, M&A, layoffs, leadership changes beyond job-change,
website/intent/traffic-spike data, social activity. For these, monitor
externally (web search — host-gated) and feed the resulting companies in via
`bulk_create_companies`, then run decision-maker discovery on that list.

## Instructions

### Phase 1 — Pick the signal and ICP overlay
1. Clarify the exact signal and the ICP constraints to overlay (size, geo,
   industry). Check the table above — if the signal isn't natively expressible,
   go to the external-monitoring path (Phase 3b).

### Phase 2 — Source companies via AI Finder
1. `preview_an_ai_finder_search` with a signal-oriented `text` query and the
   right `provider` from the table (or `AUTO` to let it route). For an
   identity-scoped LinkedIn search, first `get_identities` with
   `linkedinSearchEnabled=true` and pass that `identityId` (LinkedIn provider
   only). Returns a `previewId`; nothing is imported yet.
2. `fetch_results_from_an_ai_finder_preview` (previewId) to inspect the actual
   matches. Show the user a sample.
3. Refine with `refine_an_ai_finder_preview` (natural-language `feedback`, e.g.
   "US only", "exclude agencies", "50-500 employees"). Each refine returns a new
   `previewId`; chain as needed.

### Phase 3 — Import to a list
1. `create_a_list` (`type: COMPANIES`) for the destination.
2. `import_an_ai_finder_preview` (previewId, listId, `maxCount` 100-2500 in
   increments of 100). Returns an `actionsId`; poll `get_actions_run_status`.
   Fetch imported records with `get_companies` filtered by the run.

### Phase 3b — External-signal path (non-native signals only)
1. Gather the triggering companies via web search (**host requirement: web
   search**). If unavailable, tell the user this signal can't be sourced here.
2. `bulk_create_companies` (up to 100 per call, `listId` to drop them straight
   into a company list). Then continue at Phase 4.

### Phase 4 — Find decision-makers at matching companies
Two options:
- **Per-company scrape:** `start_an_actions_run` →
  `SEARCH_LEADS_FROM_LINKEDIN_COMPANY` against `companyGroupIds: [listId]`, with
  `options.text` describing the role ("VP Sales and RevOps leaders"),
  `maxLeadsToImport` (max 10), and a `destinationContactGroupId`. **Billable**
  (pricing key `SCRAPE_LEAD_LINKEDIN`) — confirm spend first (Phase 5 rules).
- **Contact-level AI Finder:** a `preview_an_ai_finder_search` contact query
  scoped to those companies, then import to a contact list.

### Phase 5 — Confirm spend before any billable run
Before `start_an_actions_run` (and before large imports), call
`get_credit_pricing` + `get_credit_balance`, estimate cost × record count,
confirm `spendableCredits >= cost`, and get an explicit yes.

### Phase 6 — Signal-aware outreach
1. Route to `campaign-angle-finder` with the signal as the angle (the hire, the
   raise, the tech adoption is the reason for reaching out now).
2. Hand off to `launch-campaign` to send.

### Phase 7 — Recurrence
Signals go stale, so suggest **re-running this skill weekly** to catch new
matches. This is a **manual re-run cadence** — Enginy does not schedule it for
you through these tools. Graph workflows (`get_workflows` / `run_workflow`) can
automate multi-step sequences **if workflows are enabled on your workspace**;
mention this only as an option, and only if the user asks — don't claim
automatic scheduling.

## Enginy MCP tools used

preview_an_ai_finder_search, fetch_results_from_an_ai_finder_preview,
refine_an_ai_finder_preview, get_identities, create_a_list,
import_an_ai_finder_preview, get_companies, bulk_create_companies,
get_credit_pricing, get_credit_balance, start_an_actions_run,
get_actions_run_status, get_workflows, run_workflow

## Important notes

- **Only promise signals the schema supports.** Hiring, tech stack, funding,
  job-change, ecommerce, and local are native (see the table). News, launches,
  M&A, intent, and traffic are NOT — those require external monitoring +
  `bulk_create_companies`. Say so plainly.
- **No structured signal parameter exists.** Signals are natural-language
  phrasing routed to a provider. Query quality drives result quality.
- **Funding recency is approximate.** Crunchbase natural-language queries return
  what Crunchbase surfaces; there is no guaranteed "funded in the last N days"
  filter — set expectations accordingly.
- **Confirm before spend.** `SEARCH_LEADS_FROM_LINKEDIN_COMPANY` and imports are
  billable. Always price and confirm.
- **Import limits:** `maxCount` 100-2500 in increments of 100; list type must
  match the preview entity (company list for company previews, contact list for
  contact previews) or import returns 422.
- **`maxLeadsToImport` caps at 10** per company for
  `SEARCH_LEADS_FROM_LINKEDIN_COMPANY`.
- **Previews live ~24h** and Crunchbase paging is capped (~10 pages) — inspect
  early, import before the TTL expires.
- **Recurrence is manual** unless workspace workflows are enabled. Do not claim
  Enginy auto-schedules re-runs.
- UI walkthroughs: https://docs.enginy.ai.

## Examples

**1. Hiring signal.** "Find companies hiring SDRs and get me their sales
leaders." Preview `THEIRSTACK_JOBS` text "companies hiring SDRs or BDRs in the
US, 50-500 employees", inspect, refine to exclude staffing agencies. Create a
company list, import 500. Then `SEARCH_LEADS_FROM_LINKEDIN_COMPANY` for "VP Sales
/ Head of Revenue" into a contact list (confirm credits). Route to
`campaign-angle-finder` with "you're scaling your sales team" as the angle.

**2. Funding signal.** "Recently funded Series A/B fintechs." Preview
`CRUNCHBASE_COMPANIES`, flag that recency is best-effort, import, find CFOs/Heads
of Ops, outreach angle = the raise.

**3. Non-native signal — product launch.** "Companies that just launched a new
product." Explain this isn't natively expressible; if web search is available,
gather them externally and `bulk_create_companies` into a list, then run
decision-maker discovery. If web search isn't available, say the signal can't be
sourced here and suggest a native proxy (e.g. hiring for the launch team).

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Preview returns few/no matches | Query too narrow or wrong provider | Broaden the `text`, or force the right `provider` from the table |
| Import 422 | List type ≠ preview entity | Company list for company previews, contact list for contact previews |
| Import 404 preview | Preview expired (24h TTL) | Re-run `preview_an_ai_finder_search` |
| `refine` 422 | AI couldn't build a valid refined search | Simplify the `feedback` phrasing |
| Crunchbase fetch 422 on deep page | Past the ~10-page cap | Refine the query instead of paging deep |
| Signal can't be expressed | Not a native provider signal | Use external monitoring + `bulk_create_companies` |
| `run_workflow` 409 | Workflow not published / not enabled | Workflows may be flag-gated; treat recurrence as manual |
| Decision-maker run billable-blocked | `spendableCredits` < cost | Reduce `maxLeadsToImport` or company count, or top up |

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
