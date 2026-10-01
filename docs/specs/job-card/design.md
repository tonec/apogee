# Design: Job card and outcome logging

Spec: `spec.md` · Tasks: `tasks.md`.

## Overview
A server-rendered mobile page at `/c/:token`, one per conversation, plus one-tap outcome endpoints. No accounts.

## Components
| File | Role |
|---|---|
| `card/build.ts` | assemble card fields, flags, urgency reason, area check |
| `card/token.ts` | create and verify tokens (JC-1) |
| `card/deliver.ts` | send card text; once per conversation |
| `http/routes/card.ts` | GET page, POST `/c/:token/outcome` |
| `card/view.ts` | HTML template (plain server-rendered, minimal CSS, no JS needed) |
| `card/outcomes.ts` | idempotent outcome writes |
| `card/summary.ts` | daily summary job (JC-8) |

## Token
128-bit random, stored as SHA-256 hash in `job_cards.token_hash`; URL carries the raw token. `expires_at = created_at + card_expiry_days`. Lookup by hash; unknown, expired and tampered all return the same 404 page with no data (JC-1). Short link domain configured separately for SMS length.

## Flow
1. `complete` or `number_only` -> `card/build` -> `deliver` (one send guarded by `card_sent_at`), `card_sent` event.
2. Repeat callers merge into the existing conversation, so one card (JC-6).
3. GET: record `card_opened`; if conversation is urgent call `acknowledge(conv,'card')` (JC-3). Page: large Call/Text buttons (`tel:`/`sms:`), problem in caller's words plus summary, photos via short-lived signed storage URLs, flags, outcome buttons (JC-2).
4. POST outcome: upsert on `(job_card_id, outcome)` with unique key; second tap returns same result with no new event (JC-4). Optional reason lost and value in a follow-up form.
5. `not_a_job` offers add-to-blocklist for that trader only, with undo (JC-5, per DP-9).
6. Daily `card_summary` job lists cards with no outcome (JC-8).

## Data
`job_cards`: add `token_hash`, `expires_at`, `card_sent_at`, `outcome`, `outcome_at`, `reason_lost`, `job_value_pence`. Photos stored in Supabase Storage (private bucket), EXIF stripped on ingest; same retention as messages (JC-7).

## Config
`card_expiry_days=14`, `daily_summary_time=17:30`.

## Logging
Handlers log IDs only; never token, name, number or text (JC-7). Access logs scrub path tokens.

## Failure modes
Card text fails -> status callback marks failed; urgent path already has its own ladder; routine retried once then developer-alerted.

## Testing
Snapshot test of rendered page for each flag combination; token tests (expiry, tamper, wrong hash); double-tap test per outcome; manual check on iOS Safari and Android Chrome.
