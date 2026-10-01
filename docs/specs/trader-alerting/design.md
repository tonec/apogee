# Design: Trader alerting and escalation

Spec: `spec.md` · Tasks: `tasks.md`.

## Overview
Setting urgency enqueues a set of delayed ladder jobs. Each job re-reads state and exits if the conversation is acknowledged. Acknowledgement can arrive from three entry points and all call one function.

## Components
| File | Role |
|---|---|
| `alerting/start.ts` | `startLadder(conv)`: schedule steps (TA-1) |
| `alerting/steps.ts` | job handlers per ladder step (TA-3) |
| `alerting/ack.ts` | `acknowledge(conv, via)` (TA-2) |
| `alerting/retry.ts` | slow retries after the ladder (TA-4) |
| `alerting/routine.ts` | routine delivery and reminder, quiet hours (TA-5, TA-7) |
| `http/routes/alert-call.ts` | TwiML for outbound alert call and `<Gather>` keypress |
| `http/routes/sms.ts` | recognises trader replies ("OK") by sender number |

## Flow
1. `startLadder` schedules `alert:{conv}:{step}` jobs from `traders.settings.ladder` (default `[0,2,5,8]`), using pg-boss `startAfter`. Step actions:
   - 0: SMS with card link + call trader
   - 2: call trader
   - 5: SMS + call backup; with no backup, call trader again (TA-6)
   - 8: fixed fallback text to caller
2. Each handler: guard (`alerting && !acknowledged`), act through `Messenger`, insert `alert_attempts` row (TA-8) and `alert_attempt` event, update from status callback later.
3. After the last step, `alert:{conv}:retry:{n}` repeats the trader SMS+call every `urgent_retry_interval_min` until `urgent_retry_window_min` (TA-4); final exhaustion raises a developer alert.
4. `acknowledge(conv, via)` in one transaction: set `acknowledged_at`, `acknowledged_via` (`card`|`reply`|`keypress`), emit `acknowledged`, cancel pending jobs by key. Guards make cancellation best-effort safe.
5. Quiet hours: urgent ignores them (TA-5). Routine: if now is within quiet hours, schedule `routine_send` at quiet-hours end, else send immediately; `routine_reminder:{conv}` at `routine_reminder_min` if not acknowledged (TA-7).

## Automated call
`placeCall` points Twilio at `/twilio/alert-call?conv=..&sig=..` (signed). TwiML: `<Gather numDigits=1 timeout=6>` reading "Urgent job near {area}: {summary}. Press 1 to acknowledge." repeated once. Digit 1 calls `acknowledge(..., 'keypress')`. Answer-machine detection: use `machineDetection` and treat machine answers as `no_human`, so the ladder continues.

## Ambiguous "OK"
A reply "OK" acknowledges all open urgent conversations for that trader (simple, errs toward stopping alerts). Risk: a second urgent job gets silenced. Mitigation: card link ack is per conversation and the reminder logic still sends the card; revisit after drills.

## Data
`alert_attempts(id, conversation_id, step, channel, target, sent_at, result, provider_sid)`; `conversations.acknowledged_at`, `acknowledged_via`.

## Config
`ladder=[0,2,5,8]`, `urgent_retry_interval_min=5`, `urgent_retry_window_min=60`, `routine_reminder_min=60`, per-trader override.

## Failure modes
Twilio send/call error: record attempt as `failed`, continue ladder (do not stall). Both trader and backup unreachable: ladder proceeds to caller fallback text and developer alert. Process restart: pg-boss persists jobs.

## Testing
Drill harness in `test/drills/` runs ~20 scenarios x 3 trader behaviours (off, busy, ignoring) with FakeClock and RecordingMessenger, asserting attempts at exact offsets and that a human or backup is reached. Separate tests per ack method asserting zero attempts afterwards. Live drills on real UK phones before any customer; any miss blocks launch.

## Open
Call wording and speech rate; carrier spam handling of repeated automated calls.
