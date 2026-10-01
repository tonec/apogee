# Spec: Trader alerting and escalation

Status: draft · Last updated: 2026-10-01
Depends on: `urgency-and-safety`, `job-card`, `conversation-state-machine`.

## Purpose
Make sure urgent jobs reach a human fast. Push alerts, not dashboards: the Kent test (see research) missed two emergencies because nobody checked a dashboard.

## Alert ladder (urgent conversations)
All offsets configurable per trader (defaults below):
| Offset | Action |
|---|---|
| t+0 | Text with card link, plus automated call to the trader |
| t+2 min | Automated call again |
| t+5 min | Text and call to the backup contact |
| t+8 min | Fallback text to the caller (below) |

Fallback text to caller: "We're still trying to reach {trader_first_name}. If anyone is in danger call 999 (gas emergency 0800 111 999). We'll keep trying."

The automated call reads a short summary ("Urgent job near {area}: {problem_summary}. Press 1 to acknowledge.").

## Requirements
- **TA-1 (MUST)** The ladder starts as soon as urgency is set, even if intake is still in progress.
- **TA-2 (MUST)** Acknowledgement by any of: tapping the card link, replying "OK" to the text, or pressing 1 on the call. Any acknowledgement cancels all pending ladder jobs and records `acknowledged`.
- **TA-3 (MUST)** Ladder jobs are queued with delays; each job re-reads state before acting (see `conversation-state-machine`).
- **TA-4 (MUST)** If the ladder is exhausted without acknowledgement, keep retrying the trader at a slower interval until acknowledged or a maximum window; each attempt is logged.
- **TA-5 (MUST)** Urgent alerts ignore the trader's quiet hours. Routine cards respect them (held until the morning).
- **TA-6 (SHOULD)** Backup contact is optional but strongly encouraged in onboarding; without one, the t+5 step repeats the trader call.
- **TA-7 (MUST)** Routine cards: text with link immediately (or at quiet-hours end), plus one reminder if no acknowledgement after `routine_reminder_min` (default 60).
- **TA-8 (MUST)** All attempts (channel, target, time, result) recorded in `alert_attempts`.

## Config
`ladder=[0,2,5,8]` minutes, `urgent_retry_interval_min=5`, `urgent_retry_window_min=60`, `routine_reminder_min=60`.

## Acceptance criteria (drills, before any real customer)
- About 20 scripted scenarios (gas smell, no power, active leak, routine job, spam, repeat caller, silent caller) against three trader states: phone off, busy, ignoring the alert.
- **100%** of urgent cases reach a human or the backup within the configured ladder. Any miss blocks launch.
- Acknowledgement by each method (link, reply, keypress) cancels pending jobs; no further alerts afterwards.
- Injected-clock tests verify exact ladder timings.

## Open questions
- Automated-call voice wording and speech rate.
- Whether carriers treat repeated automated calls as spam (watch in drills).
