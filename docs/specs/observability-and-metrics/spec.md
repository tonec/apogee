# Spec: Observability, metrics and safety nets

Status: draft · Last updated: 2026-10-01
Depends on: all other specs.

## Purpose
Make pilot decisions possible from data, and make sure silent failures are caught. The service becomes a single point of failure once traders divert to it: an outage means callers reach a dead line, which is worse than voicemail.

## Events (append-only `events` table)
Minimum event types: `call_received`, `repeat_call_merged`, `forwarded_from_seen`, `first_text_sent`, `text_delivered`, `text_failed`, `reply_received`, `extraction_ok`, `extraction_failed`, `safety_text_sent`, `urgency_set`, `alert_attempt`, `acknowledged`, `card_sent`, `card_opened`, `outcome_logged`, `nudge_sent`, `abandoned`, `stop_received`.

## Metrics (queries over `events`)
- Time from call to first text (p50, p95).
- Reply rate; intake completion rate; card completeness.
- Time to acknowledgement for urgent cards.
- Callbacks made within the promise (outcome logged vs promise window).
- Outcome distribution (booked / quoted / lost / not a job).
- Calls arriving per trader per week (and trader's own count of missed calls for the first week, to estimate calls lost before the divert).
- Share of calls with `ForwardedFrom` present, by network.
- Spam/duplicate share.

Explicitly **not** a metric: "conversations handled".

## Requirements
- **OM-1 (MUST)** Every state transition and every outbound message/call writes an event.
- **OM-2 (MUST)** Synthetic check: a scheduled test call to a test number must produce a text on a test phone within the first-text deadline. Run on a schedule (suggest hourly); on failure, alert the developer by text and email.
- **OM-3 (MUST)** Twilio status callbacks recorded; delivery failures trigger an alert for urgent conversations.
- **OM-4 (MUST)** Voice fallback URL configured on every number (see `call-capture` CC-10).
- **OM-5 (SHOULD)** Daily summary to the developer: calls, texts, failures, unacknowledged urgent cards.
- **OM-6 (MUST)** Personal data kept out of logs; raw messages auto-deleted after the retention period.
- **OM-7 (SHOULD)** Injected clock available in all timer code.

## Acceptance criteria
- Stopping the app causes the synthetic check to fail and alert within one check interval.
- Metrics queries run against a seeded fixture dataset and match hand-calculated values.
- A delivery failure on an urgent first text raises an alert.

## Open questions
- Retention period for raw messages and events (legal advice needed).
