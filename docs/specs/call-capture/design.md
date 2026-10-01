# Design: Call capture

Spec: `spec.md` · Tasks: `tasks.md`. Layout in `conversation-state-machine/design.md`.

## Overview
`POST /twilio/voice` records the call and returns fixed TwiML immediately. A `first_text` job sends the SMS. `POST /twilio/voice-fallback` is a separate, dependency-free route.

## Components
| File | Role |
|---|---|
| `http/routes/voice.ts` | CC-1: validate signature, find trader by `To`, dedupe on `CallSid`, create/merge conversation, enqueue job, return TwiML |
| `http/routes/fallback.ts` | CC-10: static TwiML string constant; no DB, no config load; mounted first |
| `core/jobs/first-text.ts` | send text via `Messenger`, record latency |
| `core/caller.ts` | libphonenumber classification: mobile, landline, non-UK, withheld |
| `templates/call.ts` | voice message and first text with slot filling |

## Flow
1. Validate signature; unknown `To` -> generic TwiML hang-up and an ops alert.
2. In one transaction: insert conversation (`call_received`), event `call_received`, event `forwarded_from_seen` if `ForwardedFrom` present (CC-8).
3. Decide path: blocklist/allowlist match -> no text (CC-7); withheld -> card, routine alert (CC-4); landline/non-UK -> card flagged or send if the type allows (CC-5); open conversation within `repeat_caller_window_min` -> merge, `repeat_call_merged`, set urgent, no text (CC-6); otherwise enqueue `first_text` with `startAfter 0`.
4. Return TwiML: `<Say voice="Polly.Amy" language="en-GB">` message then `<Hangup/>` (CC-2). Text sends regardless of whether the caller stays on the line (CC-9).
5. `first_text` job sends template (CC-3), writes `messages` row, emits `first_text_sent` with `latency_ms`; if over `first_text_deadline_s`, emits a `slow_first_text` event and alerts the developer.

## First text template
Target under 160 GSM-7 characters, no emoji or curly quotes (unit test checks both). Draft: "Hi, automated assistant for {business}. {first} can't take your call. Reply with name, postcode and the problem and we'll pass it on. Gas/danger: leave, call 999. Info: {shortlink}. STOP to opt out." Final emergency and privacy wording pending verification; if over 160, accept two segments but not three.

## Data
No new tables. Uses `traders.twilio_number`, `traders.settings` (allow/block lists live in `trader_lists(trader_id, number, kind, created_at)`, new, per trader only per DP-9).

## Config
`first_text_deadline_s=60`, `repeat_caller_window_min=30`, `ring_time_recommended_s=15-20` (onboarding only).

## Failure modes
Twilio send failure -> retry job up to 3 times at 5s; after the second failure raise developer alert; `text_failed` event from status callback. Fallback route stays valid with the app down because Fly serves it from a tiny separate process or Twilio's fallback URL points to a static host (decide at deploy; default: static file on a second origin).

## Testing
Replay fixtures: normal divert, repeat, withheld, landline, blocklisted, duplicate `CallSid`. Latency assertion in staging with real UK phones. Fallback test starts the route with DB unreachable.

## Open
Caller ID after divert per network; PECR status of the first text (spec open questions); measured via `forwarded_from_seen`.
