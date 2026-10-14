---
name: enrich-and-score-lead
description: >-
  Enrich a prospect (email, phone, LinkedIn, company data) and produce an ICP-fit
  score, single-record or in bulk. Use when the user says "enrich this lead",
  "fill in the missing contact info", "find this person's email and phone",
  "score this contact against our ICP", "how well does this prospect fit",
  "grade my list against our ideal customer profile", or "enrich and qualify these
  contacts".
version: 1.1.0
---

# Enrich and Score Lead

## Role and goal

You take a prospect that exists in Enginy (or that you create) from thin to
decision-ready: fill the gaps that block outreach (email, phone, LinkedIn,
company firmographics), then attach an explainable ICP-fit tier so the user
knows whether to pursue. You run enrichment through billable actions runs, so
you never spend credits without showing the cost and getting an explicit yes.
Works on one contact or a whole list.

## Instructions

### Phase 1 — Locate (or create) the record
1. If given a contact ID, call `get_a_single_contact`. Otherwise search with
   `get_contacts` (`search` by name/email) or `search_contacts_with_advanced_filters`
   for anything richer. For bulk, resolve a list ID via `get_lists`.
2. If the prospect does not exist, create it with `create_a_new_contact`
   (supply `firstName`, `lastName`, and either `linkedInProfileUrl` or
   `companyName` + `professionalEmail` so downstream enrichment has an anchor).
   Merge-on-create is on by default.
3. Return the `appUrl` (and `companyAppUrl`) so the user can open the record.

### Phase 2 — Gap check
1. Read the record's current fields. Use `get_contact_field_metadata` /
   `get_company_field_metadata` if you need the exact field names for this
   workspace.
2. List what is missing and pick only the actions that fill a real gap. Do not
   re-enrich a field that is already populated and verified. Typical gaps →
   actions:
   - No email → `ENRICH_WITH_EMAIL`
   - No phone → `ENRICH_WITH_PHONE`
   - Email present but unverified → `VERIFY_LEAD_EMAIL`
   - Phone present but unverified → `VERIFY_LEAD_PHONE`
   - Has LinkedIn URL, thin profile → `SCRAPE_LEAD_FROM_LINKEDIN`
   - No LinkedIn URL but has name + company → `LINKEDIN_FROM_NAME_LASTNAME_COMPANY`
   - Company record thin → `SCRAPE_COMPANY_FROM_LINKEDIN` or
     `SCRAPE_COMPANY_ACCOUNTIQ_FROM_LINKEDIN` (AI insights)

### Phase 3 — Enrich (billable — confirm first)
1. **Always** call `get_credit_pricing` and `get_credit_balance` before
   starting. Map each planned action to its pricing key (e.g. `ENRICH_WITH_EMAIL`
   → `ENRICH_LEAD_EMAIL`, `SCRAPE_LEAD_FROM_LINKEDIN` → `SCRAPE_LEAD_LINKEDIN`,
   `SCRAPE_COMPANY_FROM_LINKEDIN` → `SCRAPE_COMPANY_LINKEDIN`). Multiply by the
   number of records. Confirm `spendableCredits >= total cost`.
2. Show the user the action list and the estimated credit spend, and get an
   explicit go-ahead. Never start a billable run silently.
3. Start with `start_an_actions_run`. **One target kind per run** — provide
   exactly one of `contactIds`, `companyIds`, `contactGroupIds`, or
   `companyGroupIds`. Contact-side actions (enrich/verify/scrape lead) and
   company-side actions (scrape company) target different kinds, so they need
   **separate runs**. For a waterfall, set `ENRICH_WITH_EMAIL` options
   `stopType` (`VERIFIED_EMAIL` to stop once a verified email is found, `PHONE`,
   or `NONE` to try every provider) and `speed` (`SLOW` = thorough, `FAST`).
4. Poll `get_actions_run_status` with the returned `actionsId` until
   `overallStatus` is terminal (COMPLETED / FAILED / CANCELLED / PARTIAL). Read
   `statusCounts` for per-record outcomes — `alreadyUpToDate` means the worker
   skipped a current record; `blocklisted` means the target is on the workspace
   blocklist. If it stalls at PROCESSING with an old `lastUpdatedAt`, that is a
   worker backlog, not your problem to retry immediately.

### Phase 4 — Score against ICP
1. Get the user's ICP definition. If they don't have one, route to the
   `icp-definer` skill first — a good score needs a real profile.
2. **Prefer nested scoring layers over one mega-variable.** A single "score
   everything at once" prompt is hard to trust and hard to debug. Enginy's
   authoring standard is to chain single-decision variables, each doing one job
   and feeding the next by its `{fieldName}`:

   ```
   industry_classification (oneOf)  →  tech_stack_fit (oneOf)  →  icp_tier (oneOf)
   ```

   Each layer is independently testable and filterable, and a wrong tier is easy
   to localize to the layer that caused it. For a simple ICP a single `icp_tier`
   variable is still fine — reach for the chain when the profile has multiple
   independent dimensions (industry AND size AND tech AND signal).
3. **Name variables in lowercase `snake_case` scoped to their job** —
   `icp_tier`, `tech_stack_fit`, `funding_recency`, `industry_classification`.
   Scoped names keep a growing set of scoring fields legible and make the chain
   self-documenting.
4. **Split the classification from its reasoning.** For each layer keep the
   filterable label and its explanation as one variable via `type: oneOf` +
   `outputSchema.provideExplanation: true` (the explanation rides alongside the
   `oneOf` value, so you can still filter cleanly on the label while keeping the
   reason for QA). Create each with `create_an_ai_variable`:
   - `entity: CONTACT` (or `COMPANY` if scoring accounts)
   - `type: oneOf`, `values: ["A","B","C"]` (or Tier 1/2/3 — whatever the user
     wants), `provideExplanation: true`.
   - `prompt` built from the user's ICP criteria; a downstream layer references
     upstream layers by their `{fieldName}`. **Placeholders must be real field
     names** — pull them from `get_contact_field_metadata` /
     `get_company_field_metadata`. Do not invent placeholders.
5. **For detection/matching layers, bias toward recall over precision.** When a
   layer's job is to *detect* a trait or *match* a signal (does this company do
   X, does it fit segment Y), instruct the prompt to over-include rather than
   miss a valid prospect — "when uncertain, include rather than exclude." A
   missed prospect is gone for good; a false positive is caught by the
   downstream `icp_tier` layer that filters it out. Precision is the final
   tier's job, not the detector's.
6. Run each layer with `start_an_actions_run` → `FILL_LEAD_WITH_SMART_FIELDS`
   (or `FILL_COMPANY_WITH_SMART_FIELDS`), passing the variable name(s) in
   `options.fields` (run an upstream layer before the layer that depends on it).
   This is billable — pricing key `FILL_LEAD_WITH_SMART_FIELDS_AVERAGE`; confirm
   spend as in Phase 3. Poll to completion.
7. Read back the tier and explanation from the record.

### Phase 5 — Act on the score
- **High fit** → offer to route into `build-targeted-lead-list` (group the
  A-tier records) and `launch-campaign`.
- **Poor fit** → surface the explanation. If the user agrees it is a hard miss,
  optionally suppress it: `add_blocklist_entries_by_value` with
  `reason: NOT_TARGET` and the right `type` (`EMAIL`, `LINKEDIN_URL`, `DOMAIN`,
  or `COMPANY_LINKEDIN_URL`). Confirm before blocklisting — it excludes the
  target from future runs.

## Enginy MCP tools used

get_a_single_contact, get_contacts, search_contacts_with_advanced_filters,
create_a_new_contact, get_lists, get_contact_field_metadata,
get_company_field_metadata, get_credit_pricing, get_credit_balance,
start_an_actions_run, get_actions_run_status, create_an_ai_variable,
add_blocklist_entries_by_value

## Important notes

- **Confirm before spend, every time.** Enrichment, verification, LinkedIn
  scraping, and running AI variables are all billable. Always
  `get_credit_pricing` + `get_credit_balance` and get an explicit yes before
  `start_an_actions_run`. `EXPORT_TO_CRM` / CRM sync have no credit-mapped entry.
- **One target kind per actions run.** You cannot mix contact-only and
  company-only actions in a single run. Split them.
- **Enrichment is best-effort.** A provider waterfall can return nothing;
  `statusCounts` will show `failed` for records where no data was found. Enriching
  does not guarantee an email or phone exists.
- **`oneOf` AI variables require a `values` array.** Set
  `provideExplanation: true` so scores are auditable.
- **Chain single-decision layers for multi-dimensional ICPs.** Prefer
  `industry_classification → tech_stack_fit → icp_tier` (each `snake_case`, each
  one decision) over one all-in-one prompt — each layer is testable and
  filterable on its own. Bias detection/matching layers toward recall
  (over-include); let the final tier layer supply precision.
- **Placeholders only from field metadata.** Generic aliases like
  `previousMessage` are rejected by `create_an_ai_variable`.
- **Verify needs existing data.** `VERIFY_LEAD_EMAIL` / `VERIFY_LEAD_PHONE` only
  work on a contact that already has a stored email / phone.
- UI walkthroughs live at https://docs.enginy.ai — link there rather than
  describing screens.

## Examples

**1. Single thin inbound lead.** User pastes a contact ID with only a name and
company. Locate it, find it has no email/phone/LinkedIn. Propose
`LINKEDIN_FROM_NAME_LASTNAME_COMPANY` → then a contact run with
`ENRICH_WITH_EMAIL` (`stopType: VERIFIED_EMAIL`) + `ENRICH_WITH_PHONE`, plus a
company run to scrape firmographics. Show ~X credits, confirm, run, poll. Then
create an A/B/C ICP variable, fill it, report "Tier A — matches your 50-500 SaaS
ICP, VP-level" with the `appUrl`.

**2. Bulk list qualification.** User wants their 400-contact "Webinar signups"
list scored. Skip enrichment if records are already complete (check a sample).
Create the ICP variable once, run `FILL_LEAD_WITH_SMART_FIELDS` against
`contactGroupIds: [listId]`, poll, then offer to build an A-tier sublist and
launch a campaign.

**3. Poor-fit cleanup.** A scored contact comes back Tier C — "solo founder,
no budget signal, outside target size." Show the explanation; on user
confirmation, blocklist the domain with `reason: NOT_TARGET`.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `start_an_actions_run` 400 "no entities to process" | Target selector empty or all records filtered out | Re-check the ID list; confirm the list isn't empty |
| 400 validation error on the run | Contact + company actions in one run | Split into one run per target kind |
| Enrichment completes but field still empty | No data found by any provider | Read `statusCounts.failed`; try `speed: SLOW` or a wider `sortedApis` |
| `create_an_ai_variable` 400 unsupported placeholder | Placeholder not a real workspace field | Re-fetch names via `get_contact_field_metadata` |
| `create_an_ai_variable` 409 | Variable name already exists for that entity | Reuse it or pick a new name |
| Score run shows records `blocklisted` | Target already on the blocklist | Expected — those are intentionally excluded |
| Run stuck at PROCESSING | Worker backlog (stale `lastUpdatedAt`) | Keep polling; do not restart the run |
| Not enough credits | `spendableCredits` < cost | Reduce record count or top up before running |

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
