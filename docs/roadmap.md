# Roadmap

Status: draft · Last updated: 2026-10-01

Thresholds in gates are placeholders. Set them before each phase starts.

## Phase 0: Research and decisions (in progress)
- Consolidate findings in `research/`; resolve items in `research/open-questions.md`.
- Draft a Twilio UK mobile regulatory bundle in the Console as an Individual (sole trader) and as a Business (Ltd company) to see what each requires; assume one bundle per trader.
- Decide the pilot trade and region (electricians suggested: steadier demand than heating in autumn).

## Phase 1: Vertical slice (about 1 week)
Build one end-to-end path using the specs in `specs/`:
1. Day 1: submit the regulatory bundle for your first trader (or yourself as a stand-in) and buy a UK mobile (+447) number once it is approved; build the voice webhook and first-text job locally (US number allowed for logic only). Remember each additional pilot trader needs their own bundle and review time.
2. Days 2–3: inbound SMS, state machine, scripted questions, extraction, job card page and delivery.
3. Days 4–5: urgency rules, safety text, alert ladder with timers and acknowledgement.
4. Day 6: events table, outcome links, metrics queries, synthetic health check, voice fallback.
5. Day 7: drills (see `specs/trader-alerting` and `specs/observability-and-metrics`).

**Gate 1:** all urgent-ladder drills reach a human or backup within limit; first text under 60 seconds; extraction hand-check passes on about 50 sample replies.

## Phase 2: Friendly-trader trial (a few days)
One trader, daily review of every card. Fix scripts, timings and rules.

## Phase 3: Pilot (about 2 weeks)
Two to four traders, one trade, one region. Before launch, collect baseline data from their current lead platforms. Weekly check-in; log everything to `events`.

**Gate 2 (go / rethink):**
- Go: traders act on cards and say they would pay.
- Rethink: low reply rates, consistently incomplete cards, ignored alerts, or traders unwilling to pay.

## Later (decide deliberately; do not drift into these)
- Quote-chasing and follow-up after site visits (outbound, trader-approved messages).
- Voicemail capture (adds call recording and UK GDPR obligations).
- Rules-based booking of routine jobs (this is the point where it becomes a receptionist).
- Shared numbers (depends on carrier `ForwardedFrom` support; see research).
- Thin custom backend and mobile job-card page (only if Phase 3 succeeds).

## Parked parallel track: lead generation
A cheaper, exclusive-lead alternative to marketplaces (managed Google Local Services Ads priced per booked job, with this product converting the calls). Needs its own pilot; see `research/ai-receptionist-landscape.md` for the sketch and caveats.
