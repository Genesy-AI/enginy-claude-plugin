# Enginy for Claude

Find and reach B2B contacts from Claude. Enginy helps B2B sales and prospecting teams build targeted lead lists, enrich contacts, and run outreach campaigns. Search for companies and people that match your ideal customer profile, verify contact information, organize lists, draft email and LinkedIn sequences, handle replies, and review campaign performance in your existing Enginy workspace.

This plugin bundles 39 Enginy skills and the Enginy remote MCP connector. An Enginy account is required. Your connected account and its permissions determine which records and actions are available.

## Connect and use

Install the plugin in Claude, then open its Connectors tab and connect Enginy with your own account through OAuth. The server is `https://openapi.enginy.ai/mcp`, the same endpoint used by the Enginy connector. If Enginy is already connected, reuse that connection.

In Claude Code, test a checkout with `claude --plugin-dir ./enginy-claude-plugin`. Validate the files with `claude plugin validate ./enginy-claude-plugin`. The plugin contains Markdown skills, JSON configuration, and a logo; no install scripts, hooks, executables, or embedded credentials.

## Example requests

- Find companies that match my ideal customer profile and organize them into a list.
- Enrich my newest contacts with verified emails and LinkedIn profiles.
- Draft a three-email sequence for my target persona.
- Give me a performance summary of my active campaigns.

Skills retain their existing confirmation steps before sending messages, activating campaigns, and other consequential actions. Enrichment and prospect imports can consume Enginy credits; follow the estimates and approvals in the relevant skill.

## Data handling and optional services

Enginy tools can read and create or update personal data in your workspace, including names, business email addresses, phone numbers, LinkedIn profiles, contact and company records, outreach content, replies, and campaign activity. Requested changes persist in Enginy. The plugin adds no separate datastore or telemetry service. Enginy retains service data according to its privacy policy and applicable customer agreements; it does not promise deletion within 30 days.

Only the Enginy connector is bundled. Some skills can use additional services when requested and available in the user's Claude environment:

- HubSpot or Salesforce connectors for deal, revenue, and pipeline analysis.
- Claap, Modjo, Gong, Chorus, Fireflies, or another connected call-recording service for requested transcript analysis.
- Claude's web-search and browsing tools for public company websites, LinkedIn profiles, reviews, case studies, and market research, and for URLs supplied by the user. Search queries and requested URLs are sent to the selected web tools and destinations.

These optional services are not installed, authenticated, or granted access by this plugin. They require the user's own available tools and permissions. Their own privacy policies apply. Users can supply their data directly when a connector is unavailable. Some analysis skills can produce a client-rendered dashboard only when the user requests it and the client supports artifacts; Markdown reports are the default.

Enginy is a business service and is not intended for users under 18. See [Privacy policy](https://enginy.ai/privacy-policy), [Terms of service](https://enginy.ai/terms-of-service), [MCP documentation](https://docs.enginy.ai/mcp/overview), and [Support](https://enginy.ai/contact).

## Troubleshooting

If tools are missing, connect Enginy from the plugin's Connectors tab and check your workspace permissions. Reconnect if OAuth expires. If a particular action is unavailable, report the tool name and the error to Enginy support without sharing credentials. Skills that depend on optional CRM or recording integrations will request the missing data or connection.

## Skills

**Strategy & targeting**
| Skill | What it does |
| --- | --- |
| `icp-definer` | Define, score, and live-validate narrow ICPs in AI Finder |
| `persona-definer` | Pick which person to target at those companies |
| `offer-definer` | Extract value props into tiered, persona-mapped offers |
| `pain-identifier` | Evidence-based pain points for a target company |
| `competitor-finder` | Competitive map, battlecard, displacement targeting |
| `market-research-edp` | Research a SaaS market for Existential Data Points |
| `deep-company-analyser` | Customer-voice intelligence: why customers really buy |
| `gtm-action-thinker` | Pressure-test a GTM idea and name the execution path |
| `quarterly-outbound-review` | Account-level review plus a 90-day plan |

**List building & enrichment**
| Skill | What it does |
| --- | --- |
| `list-builder` | Strategy front-door — decides the targeting approach |
| `build-targeted-lead-list` | Executes the list build end-to-end in AI Finder |
| `signal-prospector` | Find companies showing hiring/funding/tech signals |
| `trigger-finder` | Which buying triggers to activate, and how |
| `niche-data-finder` | Find complementary B2B data sources for a vertical |
| `enrich-and-score-lead` | Enrich contacts and score them against your ICP |
| `pre-call-research-brief` | Prep brief on a contact and company before a call |

**Copywriting**
| Skill | What it does |
| --- | --- |
| `copywriting-first-touch` | One high-converting cold opener |
| `copywriting-follow-up` | Bump emails with a genuinely new angle |
| `copywriting-sequence` | Full 3-email sequence, calibrated to seniority |
| `copywriting-analyzer` | Audit, score, and rewrite existing copy |
| `cta-designer` | Value-based CTAs instead of "book a call" |
| `campaign-angle-finder` | 3 differentiated angles for a persona |
| `voice-profile` | Capture your writing voice so every skill writes in it |
| `linkedin-outbound-angle` | Find the attack angle on a LinkedIn profile |
| `linkedin-sequence` | 2-message post-connection DM sequence |
| `linkedin-post-drafter` | Turn an idea into a founder/seller LinkedIn post |
| `cold-call-script` | 6-part cold call script from a target description |

**Campaigns & execution**
| Skill | What it does |
| --- | --- |
| `outbound-campaign-architect` | Design the sequence structure and cadence |
| `launch-campaign` | Take a campaign from draft to live in Enginy |
| `ai-research-builder` | Per-contact AI research fields and classifications |
| `ai-snippet-builder` | Reusable AI-generated copy fragments |
| `ai-message-builder` | Channel-aware AI messages written per contact |
| `reply-handler` | Classify, tag, and answer real inbox replies |
| `meeting-follow-up` | Post-call recap with honest next steps |

**Analytics & health**
| Skill | What it does |
| --- | --- |
| `campaign-performance-analyzer` | Diagnose live campaign performance vs benchmarks |
| `deliverability-health-check` | Mailbox and domain sending health audit |
| `pipeline-analysis` | Pipeline value, conversion, forecast, velocity |
| `team-performance-review` | Side-by-side rep and sender scorecard |
| `persona-insights-analysis` | Persona intelligence from call transcripts |


## Source and version

The skill files originate from Enginy's skill source at commit `c9e2fa0124c76a160ab9fff24a2095d47f05de6a`. This includes the 38 skill names shown in the existing OpenAI listing plus `outbound-campaign-architect`, which is referenced by other skills and is part of the Enginy Skills Library. The OpenAI release export contains listing metadata but not its imported skill snapshot, so byte-for-byte equivalence to that snapshot has not been asserted.

Three copywriting skills clarify that `search` is a tool argument, avoiding a scanner misinterpretation of the word “pass” as a credential-manager command. Their behavior is unchanged.

The Claude manifest and MCP configuration adapt distribution to Claude; the server endpoint remains unchanged. The package retains the existing Enginy plugin's `UNLICENSED` designation; no new open-source license is granted.
