# Tasks: Conversation state machine

Build day: 1 (foundation for everything else). Spec: `spec.md`.

## Scaffold and test harness
- [ ] Scaffold TypeScript/Node project (Hono, Vitest, ESLint, tsconfig strict); add build, lint, test and single-test commands to root `CLAUDE.md`
- [ ] Typed config module: all timings and thresholds from the specs with defaults, per-trader overrides
- [ ] `Clock` interface with system and injected (fake) implementations
- [ ] `Messenger` interface (SMS, voice call) with Twilio and recording-mock implementations
- [ ] Twilio signature validation middleware (bypass only when `NODE_ENV=development`)
- [ ] Fixture replay helper: load Twilio voice/SMS/status payloads and POST them to the app

## Schema
- [ ] Migrations: `traders`, `conversations`, `messages`, `job_cards`, `alert_attempts`, `events` (events append-only: no UPDATE/DELETE grant)
- [ ] Unique constraints for idempotency: `CallSid`, `MessageSid`; partial unique index for one open conversation per (trader, caller number) (CSM-6)
- [ ] pg-boss setup, queue names, retry policy, worker entrypoint separate from web process

## State machine
- [ ] Transition table for all states in the spec; illegal transitions rejected and logged
- [ ] `alerting` modelled as an independent flag/track so it can run alongside `collecting` (CSM-4)
- [ ] Every transition writes an event in the same DB transaction (CSM-3)
- [ ] Idempotency helper: dedupe on SID, repeat delivery is a no-op (CSM-1)
- [ ] Job wrapper: re-read state, exit silently if the action no longer applies (CSM-2)
- [ ] Accept out-of-order status callbacks without error (CSM-5)

## Tests (acceptance)
- [ ] Replaying one webhook twice yields one conversation, one text, one event of each kind
- [ ] Nudge job scheduled before a reply does not send after the reply
- [ ] Injected clock: +7 min sends exactly one nudge; +30 min gives `number_only` and a card
- [ ] Urgency raised in `collecting` starts the ladder immediately
- [ ] Decide and record retention window before forced `closed` (open question)
