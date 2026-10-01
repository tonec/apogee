# Tasks: Job card and outcome logging

Build day: 2-3 (card and delivery), day 6 (outcomes). Spec: `spec.md`.

## Card
- [ ] Card builder from conversation: fields table in spec, flags, urgency reason, service-area check, callback promise
- [ ] Merge duplicate callers into one card (JC-6)
- [ ] Signed token generation (HMAC or random 128-bit stored hashed), expiry `card_expiry_days` (JC-1)
- [ ] `GET /c/:token`: server-rendered mobile page, large tap targets, `tel:` and `sms:` links (JC-2)
- [ ] Expired/tampered/unknown token: identical generic error, nothing revealed
- [ ] Opening card emits `card_opened` and acks urgent conversations (JC-3)
- [ ] Photos displayed from storage with signed URLs; EXIF stripped on ingest (see `data-protection` DP-11)

## Delivery
- [ ] Card-sent text to trader with link; one delivery per completed conversation; `card_sent` event
- [ ] `number_only` cards use the same page with reduced fields
- [ ] Routine delivery respects quiet hours (coordinate with `trader-alerting`)

## Outcomes
- [ ] Buttons: booked, quoted, lost, not_a_job; optional reason lost and job value
- [ ] Idempotent POST (double tap = one event); timestamps logged (JC-4)
- [ ] "Not a job" adds caller to that trader's blocklist, with undo (JC-5)
- [ ] Daily summary text of cards without an outcome (JC-8)

## Hygiene and tests
- [ ] No personal data in logs (JC-7); log scan test shared with `data-protection`
- [ ] Completed conversation yields one card and one text
- [ ] Double tap on each outcome records exactly one event
- [ ] Expired and tampered links give clear error, no data
- [ ] Manual check on iOS Safari and Android Chrome
