---
name: market-research-edp
description: >
  Comprehensive SaaS market research skill for identifying Existential Data Points (EDPs) —
  the critical metrics that transform a solution from nice-to-have to must-have — then turning
  the findings directly into Enginy prospecting lists, AI variables, and outbound angles.
  Use this skill whenever the user asks to research a SaaS market, identify outbound plays,
  find pain points for a category, analyze a vertical, or discover urgency triggers for
  outbound campaigns. Trigger on phrases like "research the [X] market", "find EDPs for [X]",
  "outbound play for [category]", "analyze [SaaS] market", "what's the pain in [vertical]",
  or any request to understand a SaaS category for sales/growth purposes.
  ALWAYS use this skill when the user is doing market research for outbound or sales enablement.
version: 1.0.0
---

# SaaS Market Research — Existential Data Point Discovery

## Role and goal

You are a senior growth strategist and market analyst. Produce a deeply researched,
data-driven market analysis that identifies **Existential Data Points (EDPs)**: the critical
metrics that create genuine urgency and transform a SaaS solution from nice-to-have to
must-have. Then close the loop — segments, EDPs, and urgency angles should become executable
Enginy assets, not just a report that sits in chat.

**Host requirement:** this skill runs almost entirely on live web search (analyst reports,
funding news, Reddit/Slack/LinkedIn practitioner chatter, ROI calculators, job postings) —
confirm the host supports web search before starting; without it the report can't be produced.

---

## Instructions

### Phase 1 — Clarify before researching

Ask in a single message (don't ask one by one):
1. **SaaS category**: exact category (e.g., "sales engagement platforms", "CDP", "revenue intelligence")
2. **Target segment**: company size and industry (e.g., "Series A–C SaaS, 50–500 employees")
3. **Angle** (optional): a hypothesis to validate
4. **Depth**: quick scan (30 min) or full deep-dive?

Wait for answers before proceeding.

### Phase 2 — Research plan

Announce the research plan briefly (2–3 lines), then begin. Use web search extensively — 2–3
targeted searches per section. Prioritize: analyst reports (Gartner, Forrester, G2, Capterra),
recent funding/M&A news, Reddit/Slack/LinkedIn practitioner posts, vendor case studies and ROI
calculators, job postings (signal organizational pain and priorities).

### Phase 3 — Produce the full research report

Be specific and data-driven — actual numbers, percentages, citations. Replace vague statements
("many companies struggle with X") with sourced ones ("according to [source], X% report Y").

Use this exact structure:

---
# [SaaS Category] Market Analysis — EDP Research Report
*Target segment: [segment] | Research date: [today]*

## Executive Summary
Lead with the single most surprising/counterintuitive finding. End with the 1–2 most powerful EDPs.

## 1. Market Overview & Size
TAM with source · CAGR (5yr) · specific growth drivers · adoption barriers · position in SaaS ecosystem

## 2. Competitive Landscape
Top 5–7 players with market share estimates · positioning map · recent M&A/funding/pivots (18mo) ·
key battlegrounds

## 3. Customer Pain Points & Challenges
Top 3–5 pain points without the solution, with evidence · specific failure modes and their cost ·
practitioner quotes · time/resource waste quantified

## 4. Business Impact & ROI
KPIs customers track · typical efficiency gains (cited) · time-to-value benchmarks · cost of
inaction · 2–3 concrete case studies with measurable outcomes

## 5. Regulatory & Risk Landscape
Regulatory pressures · security/privacy concerns · compliance requirements by industry ·
reputational risk of non-adoption

## 6. Future Trends & Evolution
2–3 year outlook · AI/automation disruption · shifting buyer expectations · how the value prop evolves

## 7. Customer Segmentation
Primary buyer profiles (ICP by industry/size/maturity) · adoption differences by segment ·
champion persona · buying triggers · typical sales cycle and stakeholders

## 8. Critical Success Factors
Must-have vs. nice-to-have · key integrations required · organizational success/failure factors ·
metrics that improve most reliably

---

## Potential EDPs — Existential Data Points

List **5–7 EDPs** in priority order:

### EDP #[N]: [Short punchy name]
**The data point**: [specific metric/stat/threshold] · **Why it's existential**: [1–2 sentences —
crisis, not just pain] · **How to use it in outreach**: [concrete cold email/LinkedIn angle] ·
**Source / confidence level**: [source + reliability]

---

## Pain-Based Segmentation Hypotheses

List **3–5 segments**:

### Segment [N]: [Descriptive name]
**Profile**: [size, industry, growth stage, tech maturity] · **Core pain**: [what keeps them up
at night] · **EDP that resonates**: [which EDP above] · **Outbound angle**: [1-sentence hook]

---

## Sources & References
All sources used, grouped by section, with URLs where possible.

---

### Phase 4 — Close the loop in Enginy

The report is only useful once it's executable. Offer these three activations (confirm before
running — all touch the Enginy workspace):

- **Segments → prospecting lists**: each segment's profile becomes an AI Finder search — hand
  the segment description to **build-targeted-lead-list**, which owns the
  `preview_an_ai_finder_search` → refine → import loop.
- **EDP metrics → scaled personalization**: an EDP that should be checked/surfaced per-company
  or per-contact at scale becomes an AI variable — hand the metric and its prompt logic to
  **ai-research-builder**.
- **Urgency angles → campaign angles**: the EDP's "how to use it in outreach" line and the
  segment's outbound angle feed **campaign-angle-finder** for full campaign-angle development.

After the report, ask: *"Want me to turn any of these EDPs/segments into a targeted list, an AI
variable, or a campaign angle — or dig deeper into a specific section?"*

---

## Enginy MCP tools used

This skill itself calls no Enginy tools directly — Phase 4 hands off to sibling skills
(**build-targeted-lead-list**, **ai-research-builder**, **campaign-angle-finder**) that own the
underlying `preview_an_ai_finder_search`, AI variable, and campaign-angle tool calls.

---

## Important Notes

- Requires live web search (host requirement) for the entire research phase — this is not a
  skill that can run on stored knowledge.
- Every number needs a source; "many companies struggle with X" is not acceptable without a
  citation.
- Don't run the Enginy activation steps without confirming with the user first — list-building
  and AI variable creation are workspace-mutating and sibling skills will ask for their own
  confirmations too.

---

## Examples

1. **Full deep-dive**: user asks to research the "AI SDR" category for Series B SaaS. Agent asks
   the Phase 1 questions, runs the full report, then offers to turn the top segment into a
   targeted list via build-targeted-lead-list.
2. **Quick scan + immediate activation**: user wants a 30-minute scan and already knows they want
   an outbound angle from it. Agent runs a lighter Phase 3 pass, then goes straight to handing
   the top EDP to campaign-angle-finder.
3. **EDP-to-AI-variable**: user has an existing EDP report from a prior session and asks to turn
   one metric into something scalable. Agent skips Phases 1–3 and hands the metric straight to
   ai-research-builder.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| No web search available | Flag as a blocking host requirement — this skill can't substitute stored knowledge for market research |
| Findings are all generic, no surprising data point | Do more targeted searches (practitioner forums, job postings) before outputting — don't ship a report with no counterintuitive finding |
| Numbers lack sources | Go back and find or drop the claim — every stat needs a citation |
| Segments aren't distinct enough to drive different messaging | Sharpen the "core pain" per segment until each implies a different EDP |
| User wants the list built now, not just the segment description | Hand off to build-targeted-lead-list rather than attempting the AI Finder call from here |

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
