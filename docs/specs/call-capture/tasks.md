# Tasks: Call capture

Build day: 1. Spec: `spec.md`. Needs: `conversation-state-machine` scaffold, trader row (can be seeded by hand until onboarding tooling exists).

## Voice webhook
- [ ] `POST /twilio/voice`: validate signature, look up trader by dialled number, create conversation, enqueue first-text job, return TwiML (CC-1)
- [ ] TwiML: fixed message in UK English voice (`Polly.Amy` or similar; confirm available) then `<Hangup/>` (CC-2)
- [ ] Message wording interpolates business name from a fixed template only
- [ ] Idempotent on `CallSid`; caller hanging up mid-message still triggers text (CC-9)
- [ ] Log `ForwardedFrom` and emit `forwarded_from_seen` (CC-8)

## First text
- [ ] First-text job: send from the dialled number, with template shortened to 1-2 segments, GSM-7 only (no emoji or curly quotes) (CC-3)
- [ ] Unit test asserting template length and character set for worst-case business/trader names
- [ ] Include privacy notice link (see `data-protection` DP-7) and STOP text
- [ ] Record `first_text_sent` with latency from `call_received`; alert if over `first_text_deadline_s`

## Edge cases
- [ ] Withheld number: card "call received, number withheld", routine alert, no text (CC-4)
- [ ] Landline/non-UK: libphonenumber type check, send only if mobile, else flagged card (CC-5)
- [ ] Repeat caller within window: no second text, merge, `repeat_call_merged`, set urgent (CC-6)
- [ ] Per-trader allowlist/blocklist: matching calls create no text (CC-7)
- [ ] Voice fallback endpoint: static TwiML, no DB or dialling, deployed independently of the main handler path (CC-10)

## Tests (acceptance)
- [ ] Fixture replay of diverted call: one conversation, expected TwiML, text enqueued
- [ ] Second call in window: no second text, urgent
- [ ] No caller ID: withheld card, no text
- [ ] Fallback returns valid TwiML with app/DB down
- [ ] Staging check on a UK number with real UK phones: first-text latency under 60s
- [ ] Resolve open questions from pilot logging: caller ID after divert, PECR status of first text
