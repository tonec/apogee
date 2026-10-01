# Design: Conversation state machine

Spec: `spec.md` · Tasks: `tasks.md`. This file also defines the shared project layout used by every other design.

## Overview
A conversation row plus an append-only event log is the single source of truth. Webhooks do minimal work (validate, dedupe, write, enqueue). All timed or slow work runs in pg-boss jobs that re-read state first.

## Shared layout
```
src/
  config/        typed config + per-trader overrides
  core/          clock.ts, messenger.ts, events.ts, state.ts, jobs.ts, idempotency.ts
  db/            migrations/, queries/
  http/          app.ts (Hono), twilio-signature.ts, routes/{voice,sms,status,card,fallback}.ts
  intake/        script, extraction, templates      (caller-intake)
  rules/         urgency + safety                   (urgency-and-safety)
  alerting/      ladder, ack, routine               (trader-alerting)
  card/          builder, page, outcomes            (job-card)
  ops/           metrics sql, synthetic check, retention, dsar
  worker.ts      pg-boss worker entry (separate process from web)
test/            fixtures/{voice,sms,status}, drills/, helpers/
```

## Interfaces
```ts
interface Clock { now(): Date }                       // SystemClock, FakeClock.advance(ms)
interface Messenger {
  sendSms(a: {from: string; to: string; body: string; mediaUrl?: string[]}): Promise<{sid: string}>
  placeCall(a: {from: string; to: string; twimlUrl: string}): Promise<{sid: string}>
}                                                      // TwilioMessenger, RecordingMessenger
type JobHandler<T> = (payload: T, ctx: {clock: Clock; db: Db; messenger: Messenger}) => Promise<void>
```
`transition(conversationId, event, tx)` is the only code that writes `conversations.state`; it checks the table below, writes the row and the event in one transaction, and returns the new state or `null` if the move is a no-op.

## Data model (additions to `tech-stack.md`)
| Table | Notes |
|---|---|
| `conversations` | add `alerting boolean`, `urgent_reason`, `last_inbound_at`, `closed_at` |
| `messages` | unique `twilio_sid` (CSM-1) |
| `conversations` | unique `call_sid`; partial unique index on `(trader_id, caller_number)` where state not in (`closed`) (CSM-6) |
| `events` | insert-only role grant; index `(conversation_id, occurred_at)`, `(type, occurred_at)` |

## State model
`alerting` is a boolean track beside `state`, so `collecting` and alerting coexist (CSM-4). Primary path: `call_received` -> `awaiting_reply` -> `collecting` -> `complete` -> `outcome_logged` -> `closed`. Side exits: `awaiting_reply` -> `number_only` (abandon); any open state -> `acknowledged` flag set by the alert track; `closed` by outcome or retention sweep.

## Key flows
- **Idempotency (CSM-1):** insert with unique SID inside the transaction; on conflict return 200 and do nothing. Job IDs are deterministic (`nudge:{conv}`, `ladder:{conv}:{step}`) so pg-boss singleton keys prevent double scheduling.
- **Job guard (CSM-2):** every handler starts with `guard(conv, predicate)`; failing predicate returns silently and emits nothing.
- **Events (CSM-3):** `transition()` writes the event; handlers never write events for transitions directly.
- **Out of order (CSM-5):** status callbacks upsert by `MessageSid`; if the message row does not exist yet, store the status in a pending table keyed by SID and merge when the send record lands.

## Config
All keys in each spec, loaded via one typed object; trader overrides in `traders.settings jsonb`. Tests construct config explicitly.

## Failure modes
Handler throws -> pg-boss retries with backoff; handlers are safe to retry because of the guards. DB down -> webhook returns 500 so Twilio retries (except the voice fallback, which has no dependency).

## Testing
Fixtures replayed through `app.request()`; FakeClock drives pg-boss via a test-only scheduler that runs due jobs. Maps one-to-one to the spec's four acceptance criteria.

## Open
Retention window before forced `closed` (spec open question); default proposed 90 days after last activity.
