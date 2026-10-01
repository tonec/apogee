# Tasks: Observability, metrics and safety nets

Build day: 6, drills day 7. Spec: `spec.md`. Event writes are added incrementally by every other feature (OM-1).

## Events
- [ ] Typed event catalogue (all types in the spec) and single `emitEvent` helper; payloads carry IDs and enums, never message text or numbers
- [ ] Lint/test rule that every state transition and outbound message/call emits an event (OM-1)
- [ ] Twilio status callback endpoint recording delivery status; `text_delivered` / `text_failed` (OM-3)
- [ ] Delivery failure on an urgent conversation raises developer alert and triggers fallback (OM-3)

## Metrics
- [ ] SQL views/queries: first-text latency p50/p95, reply rate, completion rate, card completeness
- [ ] Time to ack for urgent; callbacks within promise; outcome distribution
- [ ] Calls per trader per week; `ForwardedFrom` share by network; spam/duplicate share
- [ ] Seeded fixture dataset with hand-calculated expected values; tests compare
- [ ] Simple CLI or SQL script to print the weekly pilot report (no dashboard)

## Safety nets
- [ ] Synthetic check: scheduled call to test number, assert text on a test phone within deadline (OM-2)
- [ ] Make interval config (hourly ~$45-80/mo, 4-hourly ~$10-20/mo); start 4-hourly outside drills
- [ ] On failure alert developer by SMS and email (separate path from the app being checked, e.g. external cron or second Fly machine)
- [ ] Confirm voice fallback URL set on every number, with startup/CI check (OM-4)
- [ ] Daily developer summary: calls, texts, failures, unacknowledged urgent (OM-5)
- [ ] Injected clock available in all timer code; test coverage review (OM-7)
- [ ] Raw-message retention job hook (implemented in `data-protection`) (OM-6)

## Tests (acceptance)
- [ ] Stop the app: synthetic check fails and alerts within one interval
- [ ] Urgent first-text delivery failure raises alert
- [ ] Log scan finds no phone numbers, names or message text
- [ ] Decide retention for raw messages and events (legal advice pending)
