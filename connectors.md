# Connectors

Every MCP server this prompt uses, with setup and the placeholders to replace.

## Zevari — LinkedIn MCP for Claude (REQUIRED, the core of the prompt)

**What it does:** Live LinkedIn profile lookup (`linkedin_get_profile`), company intelligence (`agents_company_intelligence`), ICP scoring (`agents_icp_score`), research (`agents_research_person`), and lead save (`targets_save`).

**Why it's the core of this workflow:** Every other job-change tool gets you a candidate; Zevari confirms it's real and current. The whole point of the prompt is that LinkedIn is the source of truth for someone's current job, and Zevari is the safe way to query that source on a schedule.

**Setup:** [zevari.ai](https://zevari.ai) → connect LinkedIn → copy MCP server URL into Claude Code config.

**Placeholders:**
- `[ZEVARI_MCP_ID]` — your Zevari MCP server ID

**Endpoints used:**
- `linkedin_get_profile` — live current title + company (the load-bearing call)
- `agents_company_intelligence` — new company size, industry, funding, signals
- `profile_get` — load current ICP rules
- `agents_icp_score` — score new company against ICP
- `agents_research_person` — research the contact at the new role
- `library_context_get` (`craft-outreach`) — load hook formatting rules
- `targets_save` + `targets_create_list` + `targets_get_pipeline` — save lead + dedupe check
- `linkedin_get_company` (optional fallback if `agents_company_intelligence` returns thin data)

## Pipedrive — Champion list + CRM updates (REQUIRED)

**What it does:** Sources the champion watchlist via a filter. Receives title/company/note updates after a change is detected.

**Setup:** Any Pipedrive MCP server. I use `mcp__pipedrive__` from the community list.

**You need to set up before running:** A Pipedrive filter that returns the contacts you want to watch for job changes. Save the filter ID — that's `[PIPEDRIVE_CHAMPION_FILTER_ID]`.

Suggested filter criteria:
- Stage = Closed Won (past customers) OR Tag = Champion / Advocate
- Has LinkedIn URL (custom field, required)
- Last activity > 90 days ago (no point sweeping people you already talk to weekly)

**Placeholders:**
- `[PIPEDRIVE_CHAMPION_FILTER_ID]` — the Pipedrive filter ID

**Endpoints used:**
- `list_persons` / `search_contacts` — pull the filter
- `update_person` — write back new title
- `update_organization` / `create_organization` — record new company
- `add_note` — log the change for audit

## Instantly — Revival sequence enrollment (REQUIRED for the email arm)

**What it does:** Enrolls confirmed job-changers in your "Job Change Revival" cold sequence.

**Setup:** [Instantly MCP docs](https://developer.instantly.ai/mcp). Build a campaign called something like "Job Change Revival Sequence" with 3-4 emails: the trigger note, a soft value-add, a re-engagement nudge, and a breakup. Get the campaign UUID.

**Placeholders:**
- `[INSTANTLY_MCP_ID]` — your Instantly MCP server ID
- `[INSTANTLY_CAMPAIGN_ID]` — the revival campaign UUID

**Endpoints used:**
- `list_leads` — dedupe check
- `add_leads_to_campaign_or_list_bulk` — enrollment

## Apollo — Optional watchlist seed

**What it does:** If you don't have an existing champion list, Apollo's "job change" filter gives you a starting set of candidates. The prompt's Step 1 then runs each one through Zevari's live lookup to confirm.

**Why it's optional:** If your Pipedrive already has a healthy champion list (past buyers, prior contacts, anyone who knows you), you don't need Apollo. The prompt uses Pipedrive as the source of truth.

**Setup:** Any Apollo MCP server.

**Placeholders:**
- `[APOLLO_MCP_ID]` — your Apollo MCP server ID (optional)

## Claude in Chrome — Slack webhook

**What it does:** Posts to `hooks.slack.com` (the bash sandbox blocks this URL — Chrome is the only reliable way).

**Setup:** [Claude in Chrome](https://www.anthropic.com/news/claude-for-chrome).

**Placeholders:**
- `[SLACK_WEBHOOK_URL]` — your incoming webhook
- `[ALERT_CHANNEL]` — the channel name (cosmetic, used in the prompt body for clarity)
- `[YOUR_SLACK_USER_ID]` — your Slack member ID (`U...`)

## WebSearch

Built into Claude Code. Used for finding signals at the new company (press releases, funding announcements, hiring sprees) when Zevari's company intelligence is thin.

No placeholder.
