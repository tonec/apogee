# Design: Observability, metrics and safety nets

Spec: `spec.md` · Tasks: `tasks.md`.

## Overview
Events are the only source of metrics. Queries are SQL views run from a CLI. Safety nets run outside the main app so they still fire when it is down.

## Components
| File | Role |
|---|---|
| `core/events.ts` | typed catalogue + `emit(type, payload, tx)`; payload types forbid free text (OM-1, OM-6) |
| `http/routes/status.ts` | Twilio status callbacks -> `text_delivered`/`text_failed`, call results (OM-3) |
| `ops/metrics.sql` | views for each metric in the spec |
| `ops/report.ts` | CLI weekly report |
| `ops/synthetic/` | check runner, separate deployable (OM-2) |
| `ops/daily-summary.ts` | developer summary (OM-5) |
| `ops/verify-numbers.ts` | check every number has voice fallback URL set (OM-4) |

## Events
`emit` accepts only the enumerated types and a per-type payload interface (ids, enums, counts, durations). Compile-time check plus a unit test that serialises sample payloads and scans for digits patterns and names.

## Metrics views
`v_first_text_latency` (call_received -> first_text_sent, p50/p95), `v_reply_rate`, `v_completion`, `v_card_completeness`, `v_urgent_ack_time`, `v_callback_in_promise`, `v_outcomes`, `v_calls_per_trader_week`, `v_forwarded_from_share`, `v_spam_share`. "Conversations handled" is intentionally absent.

## Synthetic check
Runs as a separate Fly machine (or external cron such as GitHub Actions) so an app outage is detected. Places a call from a Twilio test number to the service number of a dedicated test trader, waits for the text on a test number (Twilio webhook to the checker or polling the message log), asserts under `first_text_deadline_s`. Failure -> SMS and email to developer via a path that does not use the app. Interval config: start every 4 hours (about $10-20/month); hourly (about $45-80/month) only around launch and drills.

## Alerting rules
| Condition | Action |
|---|---|
| synthetic check fails | developer SMS + email |
| urgent first text `text_failed` | developer alert and start ladder anyway (OM-3) |
| `slow_first_text` | developer alert |
| urgent unacknowledged after retry window | developer alert |

## Daily summary
Cron job at 08:00: calls, texts, failures, unacknowledged urgent (OM-5).

## Timers and clock
All timer code takes `Clock` from context (OM-7); lint rule bans direct `Date.now()`/`setTimeout` outside `core/clock.ts`.

## Testing
Seeded fixture dataset with hand-calculated answers per view; kill-the-app test showing the check fails and alerts within one interval; simulate delivery failure on an urgent first text; log-scan test (shared with data-protection).

## Open
Retention for raw messages and events (legal advice).
