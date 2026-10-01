# Spec: Caller intake (scripted questions and extraction)

Status: draft · Last updated: 2026-10-01
Depends on: `conversation-state-machine`, `call-capture`. Prompt in `../../system-prompt.md`.

## Purpose
Collect the minimum job details by text using a fixed script. An LLM extracts fields from replies; it never writes outbound messages.

## Script
The first text (see `call-capture`) asks for name, postcode and the problem in one message. Further prompts only ask for what is missing, in this order, at most `max_question_turns` (default 4) in total:
1. Postcode: "What's the postcode where the work is needed?"
2. Problem: "Can you tell us briefly what's wrong? A photo helps."
3. Safety: "Is anything unsafe, or are you without power, heating or water right now? (Yes/No)" (asked once if not already answered).
4. Callback (routine only, optional): "When's best to call you back?"
Closing text on completion: "Thanks, we've passed this to {trader_first_name}. They'll call you back {callback_promise}." where `callback_promise` is the trader's own setting (see `trader-onboarding`).

## Requirements
- **CI-1 (MUST)** All outbound text comes from fixed templates with slot filling. No generated prose reaches the caller.
- **CI-2 (MUST)** Each inbound reply is passed to the extraction prompt; output is validated against the schema (retry once, then fall back to raw text with `extraction_failed`).
- **CI-3 (MUST)** Postcodes validated with a UK postcode check; phone numbers normalised.
- **CI-4 (MUST)** Photos (MMS) are attached to the card; absence of photo support must not block intake.
- **CI-5 (MUST)** No reply after `nudge_after_min` (default 7): send one nudge ("Still need a hand? Reply with your postcode and what's wrong and we'll pass it on."). No reply after `abandon_after_min` (default 30): state becomes `number_only` and the card is sent.
- **CI-6 (MUST)** Price questions get a fixed reply: "We'll give you a price once we know more. Passing this on now." No other commitments.
- **CI-7 (MUST)** Messages claiming existing-customer status or containing complaint/payment-dispute language are flagged and sent to the trader as priority. The bot does not engage further.
- **CI-8 (MUST)** Caller text is treated as untrusted data (prompt-injection defence): instructions in it are ignored; the model output cannot change outbound text.
- **CI-9 (SHOULD)** Duplicate/spam handling: messages flagged spam by the model or blocklist create no card, but are logged.
- **CI-10 (MUST)** STOP replies stop all further messages to that number.

## Config
`max_question_turns=4`, `nudge_after_min=7`, `abandon_after_min=30`.

## Acceptance criteria
- Fixture set of 50–100 labelled replies: extraction hand-check shows no fabricated details; invalid or injection-style replies never alter outbound text.
- A reply containing all fields completes the conversation with no further prompts.
- A silent caller gets exactly one nudge and a `number_only` card after 30 minutes (injected clock).
- A price question gets the fixed reply, and no price appears anywhere.

## Out of scope
Quoting, availability, booking, diagnosis, advice (except the fixed safety text).
