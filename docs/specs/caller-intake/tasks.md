# Tasks: Caller intake

Build day: 2-3. Spec: `spec.md`. Prompt in `docs/system-prompt.md`.

## Inbound SMS
- [ ] `POST /twilio/sms`: validate signature, match conversation by (trader number, caller), dedupe on `MessageSid`, store message, enqueue processing job
- [ ] STOP/UNSTOP/HELP handling: STOP blocks all further outbound to that number, emits `stop_received` (CI-10)
- [ ] MMS: store media references, attach to card; intake continues if media fails (CI-4)

## Extraction
- [ ] Extraction JSON schema (name, postcode, problem, summary, urgency_hint, hazards, flags) and validator
- [ ] LLM client behind an interface; mock for tests; strip phone numbers from input (see `data-protection` DP-3)
- [ ] Prompt from `docs/system-prompt.md`; caller text wrapped as untrusted data (CI-8)
- [ ] Validate output, retry once, then store raw text and emit `extraction_failed` (CI-2)
- [ ] Postcode validation and normalisation; phone normalisation with libphonenumber (CI-3)

## Script engine
- [ ] Fixed template registry with slot filling; no other path to outbound caller text (CI-1)
- [ ] Next-prompt selection: only ask for missing fields, order postcode, problem, safety, callback; cap at `max_question_turns`
- [ ] Completion check (caller number, postcode, problem) and closing text with trader's `callback_promise`
- [ ] Fixed reply for price questions (CI-6)
- [ ] Existing-customer/complaint/payment-dispute detection: flag, priority to trader, no further bot engagement (CI-7)
- [ ] Spam handling: no card, logged (CI-9)

## Timers
- [ ] Nudge job at `nudge_after_min`, exactly one nudge (CI-5)
- [ ] Abandon job at `abandon_after_min`: `number_only` and card

## Tests (acceptance)
- [ ] Labelled fixture set of 50-100 replies (include typos, no postcode, multi-message, injection attempts, price questions, spam)
- [ ] Hand-check extraction for fabricated details; record pass rate for Gate 1
- [ ] Injection-style replies never change outbound text (assert outbound is always from template registry)
- [ ] Single message with all fields completes with no further prompts
- [ ] Silent caller: one nudge, `number_only` at 30 min (injected clock)
- [ ] Price question: fixed reply, no price anywhere in output
- [ ] Tune script wording for 1-2 SMS segments
