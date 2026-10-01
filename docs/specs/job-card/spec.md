# Spec: Job card and outcome logging

Status: draft · Last updated: 2026-10-01
Depends on: `caller-intake`, `trader-alerting`.

## Purpose
Give the trader one clean summary of a missed call on their phone, with one-tap actions and outcome logging that powers the pilot metrics.

## Delivery
Card is sent to the trader by text with a signed, expiring link to a server-rendered mobile page. No login at prototype stage.

## Fields
| Field | Notes |
|---|---|
| Caller name | Optional |
| Caller number | From caller ID; "withheld" if absent |
| Postcode/area | Flag if outside the trader's service area |
| Problem | Caller's own words plus the one-line summary |
| Photos | Attached if sent |
| Urgency and reason | e.g. "Urgent: no power to property" |
| Call time and trigger | Missed, busy, unreachable, or out of hours |
| Callback promise | The trader's own setting shown to the caller |
| Flags | Repeat caller, existing customer, possible spam, landline, extraction failed |
| Actions | Call, Text, and outcome buttons |

## Outcomes (one tap)
`booked`, `quoted`, `lost`, `not_a_job`. Optional follow-up: reason lost, job value.

## Requirements
- **JC-1 (MUST)** Card URL is signed, unguessable, and expires after `card_expiry_days` (default 14).
- **JC-2 (MUST)** Page works on a mobile browser with large tap targets; Call and Text open the phone's dialler/messages.
- **JC-3 (MUST)** Opening the card records `acknowledged` for urgent conversations.
- **JC-4 (MUST)** Outcome taps are idempotent and logged with timestamps.
- **JC-5 (SHOULD)** "Not a job" feeds the spam/allow-block lists.
- **JC-6 (MUST)** Duplicate callers are merged into one card.
- **JC-7 (MUST)** Personal data is kept out of logs; photos stored with the same retention as messages.
- **JC-8 (SHOULD)** Daily summary text to the trader listing cards with no outcome.

## Config
`card_expiry_days=14`.

## Acceptance criteria
- A completed conversation produces one card and one delivery text.
- Tapping each outcome records exactly one event even if tapped twice.
- An expired or tampered link returns a clear error and reveals nothing.

## Out of scope
Trader accounts, history views, invoicing (revisit if the pilot succeeds).
