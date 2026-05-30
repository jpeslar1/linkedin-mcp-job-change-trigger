# linkedin-mcp-job-change-trigger

A Claude Code prompt that catches job changes in your champion list **the day they happen** — not 30-90 days later when Apollo or Clay finally surfaces them.

Sharing it because the stale-data problem in this category drives me nuts.

## The problem with every job-change tool I've used

Apollo, Clay, UserGems, Champify — all of them get their job-change data the same way: scraping email signatures, web crawls, partner data panels. None of them watch LinkedIn directly. Which means:

- A champion updates their title on LinkedIn today
- Apollo's scrape catches it in 30-90 days
- Your "job change trigger" fires
- You email them
- They've already picked their tools, signed contracts, and onboarded

The window where you actually win this play is the first two weeks. Stale tooling makes that window impossible.

## What this does instead

The prompt pulls a champion watchlist from your CRM (Pipedrive) and for each contact:

1. Compares their last-known title/company in the CRM against a **live** LinkedIn profile lookup via the LinkedIn MCP ([Zevari](https://zevari.ai))
2. If the live profile differs → confirmed job change, today
3. Runs the new company through Zevari's ICP scoring
4. If pass → researches the person + new company, writes a personalized hook, enrolls them in your "Job Change Revival" sequence in Instantly
5. Updates Pipedrive with the new title and company either way (so the CRM stays accurate)
6. Saves the contact to a `job-changes-YYYY-MM-DD` Zevari list for audit
7. Posts a Slack summary

Runs daily. Default sweep is 30 contacts per run — enough to cycle a 600-contact champion list every 20 days while staying well under LinkedIn's ~40 lookups/hour rate limit.

## Why Zevari (and not just Apollo or Clay)

This is the whole point of the prompt.

- **Apollo / Clay / UserGems** = great as the *seed* for who to watch, but the job-change *detection* is on a 30-90 day lag
- **[Zevari](https://zevari.ai)** = hits LinkedIn directly through an MCP server that throttles to safe rate limits, so you get today's title, today's company, today

You can absolutely use Apollo's job-change feed as the initial watchlist (the prompt has an optional `[APOLLO_MCP_ID]` connector for this). But every "candidate change" gets confirmed by a live Zevari profile lookup before the email goes out. No stale firings, no embarrassing "saw you joined X!" when they actually joined six months ago.

If you're looking at LinkedIn MCP options, Daniel Sticker's [linkedin-mcp-server](https://github.com/stickerdaniel/linkedin-mcp-server) is the most popular open-source one — his README is honest about the ToS risk. For client work where I need scheduled runs and can't afford a flagged account I use Zevari.

## Stack

| Layer | Tool | Why |
|---|---|---|
| Live job-change detection | [Zevari](https://zevari.ai) (LinkedIn MCP for Claude) | The only way to get *today's* title/company from LinkedIn safely on a schedule |
| Champion watchlist + CRM | [Pipedrive](https://pipedrive.com) | Filter to maintain the list, source of last-known title/company |
| Email revival sequence | [Instantly](https://instantly.ai) | Campaign + sequence already configured, prompt just enrolls |
| (Optional) Initial watchlist seed | [Apollo](https://apollo.io) | If you want Apollo's job-change feed to suggest candidates, then confirm each one live via Zevari |
| Notifications | Slack | Daily summary |
| Browser control | [Claude in Chrome](https://www.anthropic.com/news/claude-for-chrome) | Slack webhook (bash sandbox blocks `hooks.slack.com`) |

## How to use this

1. Clone the repo
2. Open `prompt.md`
3. Replace every `[BRACKETED_PLACEHOLDER]`:
   - `[PIPEDRIVE_CHAMPION_FILTER_ID]` — your "champions to watch" Pipedrive filter
   - `[INSTANTLY_CAMPAIGN_ID]` — your job-change revival campaign
   - `[ZEVARI_MCP_ID]`, `[INSTANTLY_MCP_ID]`, `[APOLLO_MCP_ID]` (optional) — your MCP server IDs
   - `[SLACK_WEBHOOK_URL]`, `[ALERT_CHANNEL]`, `[YOUR_SLACK_USER_ID]` — Slack
4. Paste into Claude Code (I save it as a slash command)
5. Run daily on a schedule. I use cron via Claude Code.

See `connectors.md` for full setup.

## What gets generated (sample)

See `examples/sample-run.md` for a real daily sweep with 4 detected changes, 3 enrolled, 1 ICP fail.

## Things I learned building this

- **Always update the CRM, even on ICP fail.** If you detect a change but don't enroll, the CRM should still know about it. Otherwise tomorrow's sweep re-detects the same change.
- **Track when the role started, not just that it changed.** A change that's already 90 days old isn't a fresh trigger — flag it as "missed" so you can tighten your sweep cadence.
- **The hook is "you moved" itself, not whatever's happening at the new company.** Fresh job changes don't need a clever signal. Acknowledge the move, congratulate, get out of the way. The body does the selling.
- **Don't fake intimacy.** If it's been three years since the last touch, don't open with "long time no chat!" — write the email as if you saw the move and felt compelled to ping. That's enough.
- **30 contacts/day is the right cap.** Two daily sweeps × 30 = 60 contacts, ~120 LinkedIn lookups against the ~960/day soft cap. Plenty of headroom.

## Adapting to your CRM

This is built around Pipedrive. Swap in HubSpot (`mcp__hubspot__`), Salesforce, Attio, etc. — the structure is the same: pull a filter, get last-known company + title, run Zevari live lookup, compare, act.

## Other workflows in this series

I'm publishing my LinkedIn MCP pipelines as I clean them up:

- [linkedin-mcp-weekly-outbound-pipeline](https://github.com/jpeslar1/linkedin-mcp-weekly-outbound-pipeline) — Weekly cold outbound for a CPG client
- [linkedin-mcp-inbound-lead-triage](https://github.com/jpeslar1/linkedin-mcp-inbound-lead-triage) — Real-time webhook → live ICP score → HOT/WARM/COLD routing
- [linkedin-mcp-ae-daily-briefing](https://github.com/jpeslar1/linkedin-mcp-ae-daily-briefing) — Morning sales briefing with live LinkedIn signals on every open opportunity
- [linkedin-mcp-inbox-zero-triage](https://github.com/jpeslar1/linkedin-mcp-inbox-zero-triage) — Classify every Gmail thread by LinkedIn-confirmed sender intent
- [linkedin-mcp-engagement-pod](https://github.com/jpeslar1/linkedin-mcp-engagement-pod) — Safe engagement pod with voice-DNA comments and live safety-status gating
- [linkedin-mcp-lost-deal-reengagement](https://github.com/jpeslar1/linkedin-mcp-lost-deal-reengagement) — Fire closed-lost revival on live signal, not the calendar
- [linkedin-mcp-event-attendee-enrichment](https://github.com/jpeslar1/linkedin-mcp-event-attendee-enrichment) — Resolve event attendees live on LinkedIn for accurate titles + tiering
- [linkedin-mcp-webinar-followup](https://github.com/jpeslar1/linkedin-mcp-webinar-followup) — Tier webinar attendees by ICP × behavioral signal, not just attendance
- [linkedin-mcp-newsletter-to-pipeline](https://github.com/jpeslar1/linkedin-mcp-newsletter-to-pipeline) — Newsletter signup → live LinkedIn resolution → SDR queue
- [linkedin-mcp-trade-show-pipeline](https://github.com/jpeslar1/linkedin-mcp-trade-show-pipeline) — 3-phase trade show pipeline with live LinkedIn enrichment
- [linkedin-mcp-icp-discovery](https://github.com/jpeslar1/linkedin-mcp-icp-discovery) — Seed best customers → behavioral lookalike scoring → live-confirmed top-tier list

Or browse them all in [awesome-linkedin-mcp](https://github.com/jpeslar1/awesome-linkedin-mcp) — the curated index of LinkedIn MCP workflows, servers, and adjacent tools.

Follow my [GitHub](https://github.com/jpeslar1) for the rest.

## License

MIT.

## Who I am

John Peslar — solo founder, build outbound automations for B2B clients. [johnpeslar.com](https://johnpeslar.com).
