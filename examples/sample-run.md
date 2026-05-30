# Sample run

A real daily sweep (sanitized — names changed) from a recent Tuesday.

## Setup

- Champion watchlist had 612 contacts in Pipedrive
- Sweep cap: 30 contacts (oldest-checked first)
- All connectors healthy

## Step 2 — live Zevari lookups (30 contacts)

| Contact | Last-known (Pipedrive) | Live (Zevari) | Status |
|---|---|---|---|
| A. Chen | Head of Growth @ atomo | Head of Growth @ atomo | no change |
| B. Rivera | Director of Sales @ thrilling | **VP Sales @ olipop** | 🆕 changed (12 days ago) |
| C. Park | CMO @ magic mind | CMO @ magic mind | no change |
| D. Lopez | VP Brand @ olipop | VP Brand @ olipop | no change |
| E. Khan | Founder @ recess | **Operating Partner @ verlinvest** | 🆕 changed (8 days ago) |
| F. Singh | Head of Retail @ ghia | Head of Retail @ ghia | no change |
| G. Yamada | Founder @ graza | Founder @ graza | no change |
| H. Brooks | Brand Manager @ fishwife | **Brand Director @ fishwife** | 🆕 title change only |
| I. Adeyemi | Director @ goat fuel | **VP Marketing @ liquid death** | 🆕 changed (4 days ago) |
| J. Tanaka | Head of DTC @ perfy | Head of DTC @ perfy | no change |
| (...20 more, all no change) |  |  |  |

**Net:** 4 confirmed job changes out of 30 contacts (~13%, which is high — typical run is 2-3%).

## Step 4 — ICP scoring on new companies

| Contact | New company | ICP score | Decision |
|---|---|---|---|
| B. Rivera | olipop | 9/10 | enroll |
| E. Khan | verlinvest (PE/VC firm) | 3/10 (industry mismatch) | **ICP fail — CRM update only** |
| H. Brooks | fishwife (same company, promotion) | 7/10 | enroll |
| I. Adeyemi | liquid death | 9/10 | enroll |

## Step 6 — hooks written (3 enrolled)

```
B. Rivera, olipop
hook: Saw you just joined olipop - congrats on the move!
companyRef: OLIPOP
industryRef: bev brands
oldCompanyRef: Thrilling

H. Brooks, fishwife
hook: Saw the promotion to brand director at fishwife - congrats!
companyRef: Fishwife
industryRef: food brands
oldCompanyRef: Fishwife

I. Adeyemi, liquid death
hook: Saw you just joined liquid death - congrats on the move!
companyRef: Liquid Death
industryRef: bev brands
oldCompanyRef: Goat Fuel
```

## Step 7 — Pipedrive updates (all 4 detected changes)

All 4 contacts updated with new title, new company linked (created `liquid death` org since it didn't exist), and an audit note logged:

```
Job change detected 2026-05-26: Director @ Goat Fuel → VP Marketing @ Liquid Death.
Live-confirmed via Zevari.
```

E. Khan's row got the same audit note even though he wasn't enrolled — the CRM should always reflect reality.

## Step 8-9 — Instantly enrollment + Zevari list save

3 enrolled in `Job Change Revival Sequence`. All 3 saved to a Zevari list `job-changes-2026-05-26`.

## Step 10 — Slack summary posted

```
🔄 Job Change Sweep — Daily Run

📅 Run date: 2026-05-26
👥 Champions checked: 30
🆕 Fresh job changes detected: 4
✅ Enrolled (ICP pass): 3
🚫 ICP fail at new company: 1 — E. Khan → verlinvest (PE firm, industry mismatch)
⏭️ Already enrolled / in recent list: 0
🔗 LinkedIn cap hit: no
❗ Missing LinkedIn URL: 0

Confirmed changes:
• B. Rivera — Thrilling → OLIPOP (VP Sales) — Saw you just joined olipop - congrats on the move!
• H. Brooks — Fishwife → Fishwife (Brand Director, promotion) — Saw the promotion to brand director at fishwife - congrats!
• I. Adeyemi — Goat Fuel → Liquid Death (VP Marketing) — Saw you just joined liquid death - congrats on the move!
• E. Khan — Recess → Verlinvest (Operating Partner) — ICP fail (PE firm)
```

## Step 11 — reflection notes

- All 4 changes were < 30 days old. Sweep cadence is healthy.
- E. Khan moving to a PE firm is interesting context even though it's an ICP fail — flagged for the AE in case he becomes a "warm intro to portfolio cos" angle later.
- I. Adeyemi joining Liquid Death is the highest-value signal of the week. Hand-flagged for AE follow-up in addition to the automated sequence.
- Total run time: ~9 min.
