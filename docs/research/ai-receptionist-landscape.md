# AI receptionist landscape and adjacent opportunities (UK)

Status: draft · Last updated: 2026-10-01

**Provenance:** early web research. Most material is vendor marketing. URLs for first-pass findings were not retained; re-capture before relying on figures.

## Vendor claims (treat as hypotheses)
- Missed-call rates of roughly 27–47% for SMEs; "£24K/year lost" figures rest on vendor-chosen assumptions (e.g. 3 missed calls/day, 30% conversion, £180 job).
- Pricing varies: some UK offerings flat-rate with no per-minute charge; others start around £349–£499/month plus setup. Check current pricing.

## Independent-ish evidence: a Kent heating firm 30-day test (single case)
Author is an AI consultant; unverifiable, but detailed. July 2026, three-engineer business.
- 214 calls handled; 89 captured correctly; 41 callers hung up within 10 seconds of realising it was synthetic; 34 duplicates; 50 spam/wrong number/supplier.
- About 21 captured leads became paid jobs; cost about £172 for the month.
- Failures: hang-ups on synthetic voice; poor handling of urgent/emotional calls; pretending to be human backfired; two emergency callouts unnoticed over a weekend because transcripts were reviewed weekly; a spoken option to reach a person reduced frustrated hang-ups.

## Complaint themes (reviews of AI and human answering services)
- Billing and cancellation (charges after cancelling; payment problems).
- Message quality drift (wrong names, thin details, solicitors put through).
- Support disappearing, unclear whether talking to AI or a human.
- Preference for humans: 8x8 UK survey, three-quarters of consumers prefer human support (commercial survey); TechRadar piece reports 42% admit being ruder to AI chatbots.
- Real user feedback on UK AI-receptionist products was thin (few reviews); interview or pilot traders instead.

## Implications (reflected in `mission.md` and specs)
Text-first, disclosed automation; instant human handoff; real-time urgent alerts (not dashboards); duplicate/spam handling; simple month-to-month billing.

## Adjacent: lead marketplace alternative (parked)
- Marketplaces are expensive largely because they buy homeowner traffic from Google; exclusivity raises per-lead cost. "Cheaper and exclusive" needs cheaper demand or a different offer.
- **Google Local Services Ads already exist:** per-lead pricing; one UK agency reports £20–£55/lead for electricians and about £99/booked job from 12 campaigns (V). Agency management from about £500/month plus £500–£1,500 ad spend.
- **Check first:** whether LSAs serve trades in the chosen region. Google's UK help page includes a note that ads for one category won't trigger outside Greater London; unclear if it applies to trades. Verification reportedly takes 3–4 weeks and includes background checks and licence checks (e.g. Gas Safe, NICEIC/NAPIT).
- **Shapes considered:** managed Google demand priced per booked job; territory exclusivity; pay-on-completion marketplace (leakage risk); trader-to-trader overflow exchange; past-customer reactivation.
- **Pilot sketch:** 4 electricians in one region, ~12 weeks, trader-owned Google accounts, management fee waived, stop-loss guarantee, success if cost per booked job ≤ ~£100–£150 and ≥3 of 4 would pay. Pairs naturally with this product's text-back for conversion.
