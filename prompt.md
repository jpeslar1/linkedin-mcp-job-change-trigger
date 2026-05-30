# Job Change Trigger — Daily Sweep

A Claude Code prompt that detects job changes in a "champion list" (past buyers, prior contacts, anyone who already knows your product), confirms the change live via the LinkedIn MCP (Zevari), researches the new company, scores it against your ICP, and enrolls the contact in a revival sequence.

The reason this prompt exists: Apollo's and Clay's job-change feeds are 30-90 days stale because they're scraped from email signatures and web crawls. Zevari hits LinkedIn directly, so the trigger fires the day the person changes their title — which means your email lands while they're still in onboarding at the new role, not after they've already picked their tools.

---

## Overview

You are running the daily job-change sweep. For each champion in the watchlist:

1. Pull their **last-known company + title** from the CRM (Pipedrive).
2. Get **today's company + title** live from the LinkedIn MCP (Zevari `linkedin_get_profile`).
3. If they match → no change, move on.
4. If they differ → confirmed job change. Run the new company through ICP scoring (Zevari), research the person at the new role (Zevari), write a personalized hook, and enroll them in the revival sequence in Instantly.
5. Update Pipedrive with the new title + company.
6. Save the lead to a Zevari list called `job-changes-YYYY-MM-DD`.
7. Slack summary at the end.

Cap: 30 champions per daily sweep to stay under LinkedIn's ~40 lookups/hour rate limit. Anything over rolls to tomorrow's queue.

---

## ERROR & APPROVAL NOTIFICATIONS

If anything stops the task or needs approval, send to `[ALERT_CHANNEL]` via Chrome (bash sandbox blocks `hooks.slack.com`):

1. Call `mcp__Claude_in_Chrome__tabs_context_mcp` with `createIfEmpty: true`
2. If the tab is on `chrome://`, navigate to `https://google.com` first
3. Run via `mcp__Claude_in_Chrome__javascript_tool`:

```js
fetch('[SLACK_WEBHOOK_URL]', {
  method: 'POST',
  mode: 'no-cors',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    text: "<@[YOUR_SLACK_USER_ID]> ⚠️ *Job Change Sweep — Action Needed*\n\n<error or approval ask>"
  })
}).then(r => ({status: r.status, type: r.type})).catch(e => ({error: e.message}))
```

`{status: 0, type: "opaque"}` = success.

---

## CONNECTORS

- **Zevari (LinkedIn MCP):** `mcp__[ZEVARI_MCP_ID]` — live profile lookup, company intelligence, ICP scoring, research, target save
- **Pipedrive:** `mcp__pipedrive__` — champion list + CRM updates
- **Instantly:** `mcp__[INSTANTLY_MCP_ID]` — revival sequence enrollment
- **(Optional) Apollo:** `mcp__[APOLLO_MCP_ID]` — if you want to seed the watchlist from Apollo's job-change feed first, then confirm each one via Zevari
- **Chrome:** `mcp__Claude_in_Chrome__` — Slack webhook
- **WebSearch:** new company news + signal hunting

See `connectors.md`.

---

## STEP 1 — LOAD THE CHAMPION WATCHLIST

The watchlist is a Pipedrive filter called `Champions - Watch for Job Changes` (or whatever your team named it). It should contain:

- Past customers (closed-won contacts)
- Past evaluators who liked the product but didn't buy (champions at the wrong time)
- Anyone tagged "champion" or "advocate" in your CRM

Pull the filter via Pipedrive `search_contacts` or `list_persons` with the filter ID `[PIPEDRIVE_CHAMPION_FILTER_ID]`.

For each contact, you need:
- `linkedin_url` (custom field — REQUIRED, skip the contact if missing and log to Slack)
- `last_known_company` (from `org_id` → org name)
- `last_known_title` (from `job_title` field)
- `last_updated_at` (so you can sort)

Sort by `last_updated_at` ascending (oldest first — they're most likely to have moved). Take the first 30.

---

## STEP 2 — LIVE LOOKUP VIA ZEVARI

For each champion, call `linkedin_get_profile` with their LinkedIn URL.

- Process no more than 3 concurrent calls.
- This task is capped at 30 contacts to stay under LinkedIn's ~40/hour cap.
- If you hit the cap anyway: stop the batch, save remaining contacts to a "tomorrow" file, and flag in the Slack summary.

From the response, extract:
- `current_company` (the most recent experience entry where `end_date` is null or "Present")
- `current_title`
- `current_company_url` (LinkedIn company URL)
- `started_at_current_role` (start date of current role)

If the live profile shows the **same** company AND title as Pipedrive → no change. Skip.

If the live profile shows a **different** company OR title:
- If the new role started > 90 days ago → log as "stale change, missed" but proceed (better late than never)
- Otherwise → confirmed fresh job change, proceed to Step 3

---

## STEP 3 — DEDUPE CHECK (don't re-enroll)

Before enrolling, check that you haven't already pinged this person about this exact change:

- Search Instantly campaign `[INSTANTLY_CAMPAIGN_ID]` via `list_leads` for the contact's email. If they're in the revival campaign in any status → skip and log "already enrolled."
- Search the Zevari job-change lists (`targets_get_pipeline`) for this contact in the last 60 days → skip if found.

---

## STEP 4 — NEW COMPANY INTELLIGENCE + ICP SCORE

For each confirmed job change:

1. Call Zevari `agents_company_intelligence` on the new `current_company_url`. This pulls company size, industry, recent news, funding, hiring signals.
2. Call Zevari `profile_get` to load current ICP rules + hard disqualifiers.
3. Call Zevari `agents_icp_score` with the company intelligence + your ICP profile.

If the new company **fails ICP** → do not enroll. Update Pipedrive with the new title + company (Step 7) but skip the email. Log "ICP fail at new company" in Slack.

If the new company **passes ICP** → proceed to research and enrollment.

---

## STEP 5 — RESEARCH FOR THE HOOK

Call Zevari `agents_research_person` on the LinkedIn URL.

Also WebSearch:
- `"[contact name]" "[new company]"` for press releases or hiring announcements
- The new company's news in the last 30 days

You're looking for a hook angle. In priority order:

1. **The job change itself** — if it's fresh (< 30 days) the move IS the hook. "Saw you just joined [new company] - congrats on the move!"
2. **A company signal at the new role** — fundraise, product launch, retail expansion, hiring spree
3. **Personal-brand signal** — a podcast appearance, a recent post that got traction
4. **Warm opener fallback** — "I hope the new role is going well." (Use this ONLY if nothing above qualifies.)

---

## STEP 6 — WRITE EMAIL VARIABLES

Call Zevari `library_context_get` with `skill_slug='craft-outreach'` and `include_examples=true`. Do not write any variables until you've read this — all hook formatting rules live there.

For each contact, determine: `hook`, `companyRef`, `industryRef`, `oldCompanyRef` (for relationship reminder).

### Hook rules (job-change edition)

1. **Lowercase everything after the first letter** — same rule as cold outbound. Reads like a casual text.
   - ✓ `"Saw you just joined olipop - congrats on the move!"`
   - ✗ `"Saw you just joined Olipop — congrats on the move!"`

2. **No LinkedIn references.** Don't say "saw your post" or "noticed on linkedin." Write as if you heard it through the grapevine.

3. **Don't fake intimacy.** If it's been three years since the last touch, don't open with "long time no chat!" Acknowledge the gap honestly OR don't acknowledge at all.

4. **Reference the relationship if it adds warmth, skip if it's forced.** Good: "we chatted when you were at [old company]." Bad: "remember me?"

5. **Reactions: brief.** `congrats on the move!` `congrats on the new role!` `excited for you!` — pick one, stop there.

6. **Don't pitch in the hook.** The hook earns the open. The body pitches.

7. **Warm opener fallback:** `I hope the new role is going well.` Only if no qualifying signal.

### companyRef
- New company short name, title case.

### industryRef
- Bucket the new company into your standard industry buckets ("food brands", "bev brands", "saas teams", "agency leaders", etc. — match what your campaign expects).

### oldCompanyRef
- The company they're leaving. Used in the email body if you want to reference the prior relationship.

---

## STEP 7 — UPDATE PIPEDRIVE

For every confirmed job change (ICP pass OR fail):

1. Update the person's `job_title` to `current_title`
2. Update or create an organization for `current_company` and link it
3. Append a note: `"Job change detected [YYYY-MM-DD]: [old_title] @ [old_company] → [new_title] @ [new_company]. Live-confirmed via Zevari."`
4. Add a tag: `job-change-[YYYY-MM]`

The CRM should always reflect reality. Even if you didn't enroll them, log the change.

---

## STEP 8 — ENROLL IN INSTANTLY

For ICP-pass contacts only.

Call `add_leads_to_campaign_or_list_bulk` to enroll in campaign `[INSTANTLY_CAMPAIGN_ID]` (`"Job Change Revival Sequence"`).

Required fields: `email`, `first_name`, `last_name`, `company_name` (= new company), `custom_variables: {hook, companyRef, industryRef, oldCompanyRef}`
Set `skip_if_in_campaign: true`.

---

## STEP 9 — SAVE TO ZEVARI

For each enrolled contact:

1. Call `targets_save` to save as a Zevari target with the new company data
2. Create or find a list named `job-changes-YYYY-MM-DD` via `targets_create_list`
3. Add each contact to that list

This builds a historical record of every detected job change, which you can audit later.

---

## STEP 10 — SLACK SUMMARY

Format:

```
<@[YOUR_SLACK_USER_ID]> 🔄 *Job Change Sweep — Daily Run*

📅 Run date: [date]
👥 Champions checked: [N]
🆕 Fresh job changes detected: [N]
✅ Enrolled (ICP pass): [N]
🚫 ICP fail at new company: [N] — [names + new company + reason]
⏭️ Already enrolled / in recent list: [N]
🔗 LinkedIn cap hit: yes/no
❗ Missing LinkedIn URL: [N] — [names]

Confirmed changes:
• [Name] — [old_company] → [new_company] ([new_title]) — [hook OR "ICP fail"]
[list all detected changes]
```

---

## STEP 11 — REFLECTION

- Any contacts where Zevari profile lookup failed (LinkedIn structure changed, contact deleted profile, etc.) → flag for manual review.
- Any "stale change" misses (> 90 days old) → consider increasing watchlist sweep frequency.
- Any hook rule violations → note for next run.

---

## Schedule

Daily at 8:00 AM local time. Sweeps 30 contacts/day, rotates through the full champion list every (list_size / 30) days. For a 600-contact champion list, every champion gets a fresh live-check every 20 days.

If you want faster cycles, run twice a day (8 AM + 8 PM) and double the throughput.
