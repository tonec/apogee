# Mission

Status: draft · Last updated: 2026-10-01

## One-liner
Turn a missed call into a structured job card on a UK tradesperson's phone within a minute, and make sure urgent ones reach a human fast.

## Problem
UK sole traders and small trade firms lose work when they can't answer the phone (on a job, on another call, out of hours). Customers who can't reach someone move on, and many report being ghosted. Existing options are either voicemail (rarely used), human answering services (billing and quality complaints), or full AI receptionists (callers hang up on synthetic voices, emergencies go unnoticed). See `research/`.

## Users
- **Primary:** sole traders and 2–10 person firms in UK electrical, plumbing and heating work.
- **Secondary:** members of the public ringing a tradesperson, who need to know their call was not lost.

## Product principle: capture, triage, hand off
The service only does three things:
1. **Capture** the caller's details after a missed call (by text, within 60 seconds).
2. **Triage** into urgent or routine using plain rules.
3. **Hand off** a structured job card to the trader, escalating urgent ones until a human acknowledges.

## Design principles
- **No commitments on the trader's behalf.** The system never quotes, books, promises availability, diagnoses, or negotiates. Test for any feature: does it commit the trader to something? If yes, it is out of scope.
- **Honest automation.** Always disclosed as automated. Never pretend to be a person.
- **Urgent first.** Emergencies must reach a human or backup contact; a missed urgent alert blocks launch.
- **AI reads, rules decide.** An LLM only extracts fields from caller replies. Urgency, safety messages and escalation are deterministic rules. The caller only receives scripted messages.
- **Measure outcomes, not conversations.** Track jobs booked and time to acknowledgement, not "conversations handled".
- **Boring billing.** Month-to-month, no per-minute surprises, easy cancellation.

## Non-goals (for now)
- Quoting, pricing, booking into diaries, diagnosing problems.
- Open-ended AI voice conversations.
- Handling complaints, payment disputes or existing-customer issues (routed to the trader).
- Lead generation or a marketplace (parked; see `roadmap.md` and `research/ai-receptionist-landscape.md`).
- Call recording and voicemail capture (deferred; see `roadmap.md`).

## Success measures (pilot)
Thresholds to be set before the pilot starts; track:
- Time from missed call to first text (target: under 60 seconds).
- Caller reply rate and job-card completeness.
- Time to trader acknowledgement for urgent cards (drills: 100% reach a human or backup).
- Callbacks made within the promised window.
- Outcomes logged (booked / quoted / lost / not a job).
- Whether traders say they would pay for it.
