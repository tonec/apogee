# Spec: Conversation state machine

Status: draft · Last updated: 2026-10-01
Depends on: nothing. Used by all other specs.

## Purpose
Single source of truth for where each missed-call conversation is, so timers, webhooks and alerts never act on stale state.

## States
| State | Entered by | Timers started |
|---|---|---|
| `call_received` | Voice webhook for a diverted call | First text deadline (60s) |
| `awaiting_reply` | First text sent | Nudge (default 7 min), abandon (default 30 min) |
| `collecting` | Inbound reply | Re-ask timer per scripted prompt |
| `complete` | Required fields captured | Card delivery |
| `number_only` | Abandon timer fired with no usable reply | Card delivery (number and call time only) |
| `alerting` | Urgency = urgent | Alert ladder (see `trader-alerting`) |
| `acknowledged` | Trader acknowledges card or alert | Cancels pending alert jobs |
| `outcome_logged` | Trader taps an outcome | None |
| `closed` | Outcome logged, or no action after retention rules | None |

Required fields for `complete`: caller number, postcode, problem (any non-empty). Name is optional.

## Requirements
- **CSM-1 (MUST)** Every webhook handler is idempotent. Keys: `CallSid` for voice, `MessageSid` for SMS and status callbacks. A repeat delivery produces no new side effects.
- **CSM-2 (MUST)** Every queued job reads current conversation state before acting and exits silently if the state no longer warrants the action (for example a nudge after the caller replied).
- **CSM-3 (MUST)** Transitions are recorded as events (see `observability-and-metrics`).
- **CSM-4 (MUST)** A conversation can be `alerting` and `collecting` concurrently: urgency raised mid-collection starts the ladder without waiting for intake to finish.
- **CSM-5 (SHOULD)** Out-of-order events (for example status callback before send confirmation) are accepted without error.
- **CSM-6 (MUST)** One open conversation per (trader, caller number). A new call from the same caller within the repeat window merges into it (see `call-capture`).

## Acceptance criteria
- Replaying the same webhook payload twice creates one conversation, one text, one event of each kind.
- A nudge job scheduled before a reply does not send after the reply arrives.
- With an injected clock, advancing time by 7 minutes with no reply sends exactly one nudge; advancing to 30 minutes yields `number_only` and a card.
- Urgency raised in `collecting` starts the alert ladder immediately.

## Open questions
- Retention window before `closed` is forced (see `research/open-questions.md`).
