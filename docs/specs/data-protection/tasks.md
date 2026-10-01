# Tasks: Data protection

Build items run alongside features; paperwork must finish before any trader goes live. Spec: `spec.md`. Not legal advice; TBC items need a solicitor.

## Paperwork (blocks launch: DP-1, DP-2, DP-7)
- [ ] Obtain fixed-fee solicitor quotes; send "Questions for the solicitor"
- [ ] DPA template (trader as controller, service as processor); sub-processor authorisation and breach timeframe
- [ ] Complete sub-processor register: Twilio, Fly.io, Supabase, LLM provider (region, DPA, transfer mechanism, retention, training terms)
- [ ] Choose LLM provider terms (zero retention or no-training) and record
- [ ] Plain-language privacy notice page naming trader and service; link from first text (DP-7)
- [ ] DPIA screening decision (DP-12); ICO registration/fee check for each role (DP-13)
- [ ] Breach process document (DP-6)

## Engineering
- [ ] Configure Fly.io and Supabase in London; verify Supabase Pro (Free pauses when idle)
- [ ] Secrets in provider secrets manager; least-privilege DB roles; TLS enforced (DP-8)
- [ ] Logger wrapper that rejects/redacts phone numbers, names, message text; CI log-scan on fixture run (DP-3)
- [ ] LLM input sanitiser strips phone numbers; test captures request and asserts none (DP-3)
- [ ] EXIF stripping on photo ingest; photo deletion on request (DP-11)
- [ ] Retention config with defaults (messages and photos 30-90 days; events 12 months, de-identified calls after 90 days) (DP-4)
- [ ] Scheduled deletion jobs: Postgres rows, stored photos, Twilio messages and media via API
- [ ] Subject-access tool: search by caller number across Postgres, storage, Twilio; export and delete (DP-5)
- [ ] Request-forwarding log for requests received by the service
- [ ] Per-trader isolation checks: no cross-trader lists or lookups (DP-9)
- [ ] Confirm no recording or voicemail capture paths exist (DP-10)

## Tests (acceptance)
- [ ] Log scan over fixture run: no numbers, names or text
- [ ] Deletion job on seeded old data removes DB rows, photos and Twilio-side data
- [ ] Search then delete by caller number returns and removes everything
- [ ] DPA checklist (DP-1, DP-2, DP-7) complete before go-live
- [ ] Check Twilio default retention and whether all of it is deletable by API (open question)
