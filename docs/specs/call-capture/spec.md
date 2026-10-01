# Spec: Call capture (missed-call trigger and first response)

Status: draft · Last updated: 2026-10-01
Depends on: `conversation-state-machine`, `trader-onboarding`.

## Purpose
Treat any call the network diverts to the service as a missed call, answer it with a short fixed message, and send the first text within 60 seconds.

## What counts as a trigger
A call arriving at the trader's service number from the trader's divert (busy, no answer, or unreachable; a declined call counts as busy). Calls the trader answers, and calls where the caller hangs up before the divert fires, never reach the service and cannot be captured.

## Requirements
- **CC-1 (MUST)** `POST /twilio/voice` validates the Twilio signature, identifies the trader by dialled number, creates a conversation (`call_received`), enqueues the first-text job, and returns TwiML without slow work in the handler.
- **CC-2 (MUST)** TwiML plays a fixed message in a UK English voice, then hangs up. Wording: "Sorry we missed your call. This is an automated service. We're sending you a text message now. Please reply to it with your details and [business name] will get back to you."
- **CC-3 (MUST)** First text is sent within 60 seconds (config `first_text_deadline_s`), disclosed as automated, from the same number the caller reached. Template:
  "Hi, this is the automated assistant for {business}. {trader_first_name} can't take your call right now. Reply with your name, postcode and what's wrong and we'll pass it on straight away. If there's a gas smell, smoke or danger to life, leave and call 999 (gas emergency: 0800 111 999). Reply STOP to opt out."
  (Emergency wording and opt-out text to be verified; see `research/uk-telephony-and-compliance.md`.)
- **CC-4 (MUST)** Withheld number: no text possible. Create a card with "call received, number withheld" and alert the trader as routine.
- **CC-5 (SHOULD)** Landline or non-UK number: send the text only if the number type allows it; otherwise create a card flagged "landline: text may not be usable".
- **CC-6 (MUST)** Repeat caller within `repeat_caller_window_min` (default 30): no second text. Merge into the open conversation, record a repeat-call event, and raise urgency to urgent.
- **CC-7 (SHOULD)** Allowlist (trader's own staff and family) and blocklist (known spam) per trader; matching calls create no text.
- **CC-8 (MUST)** Log `ForwardedFrom` when present, to measure how often carriers supply it.
- **CC-9 (MUST)** A caller who hangs up during the message still gets the text.
- **CC-10 (MUST)** Voice fallback URL returns a static message and hangs up. It must not dial the trader's mobile (divert loop risk).

## Config
`first_text_deadline_s=60`, `repeat_caller_window_min=30`. Recommended trader-side ring time before divert: 15–20 seconds (set in onboarding).

## Acceptance criteria
- Fixture replay of a diverted call produces one conversation, TwiML with the fixed message, and a text enqueued; first-text latency under 60s in staging.
- A second call from the same number within the window sends no second text and marks the conversation urgent.
- A call with no caller ID creates a withheld-number card and sends no text.
- Voice fallback returns valid TwiML when the app is down.

## Open questions
- Does caller ID survive diverts on each UK network? (Pilot logging.)
- Is the first text a service message under PECR?
