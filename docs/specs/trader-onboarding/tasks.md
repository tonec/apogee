# Tasks: Trader onboarding and telephony setup

Mostly operational, start Day 1 (bundle review takes days). Spec: `spec.md`.

## Phase 0 research items (do first)
- [ ] Draft a UK mobile regulatory bundle in the Twilio Console as Individual and as Business; record required fields
- [ ] Settle subaccount structure per trader; test whether bundles can be cloned or reused
- [ ] Update `research/uk-telephony-and-compliance.md` and `open-questions.md` with findings
- [ ] Submit bundle for first trader (or yourself as stand-in); allow at least a week before go-live (TO-8)
- [ ] Confirm +447 number has Voice + SMS capability before purchase (TO-3)

## Tooling
- [ ] Script/CLI `onboard-trader`: creates trader row, buys number (only with approved bundle, TO-6), sets voice/SMS/status/fallback URLs
- [ ] Trader settings model and CLI editing: hours, quiet hours, backup contact, promises, ladder timings, allow/block lists (TO-4)
- [ ] Refuse to mark a trader live unless: bundle approved, DPA signed, urgent drill passed, backup tested (TO-1)
- [ ] Offboarding script: release number, delete data per retention policy

## Documents
- [ ] One-page trader guide: what diverts do, voicemail replacement disclosure in writing, undo codes (`##61#`, `##67#`, `##62#`, EE `##004#`, O2 restore) (TO-2)
- [ ] Divert setup instructions per network (EE, O2, Vodafone, Three) with ring time 15-20s where supported
- [ ] Onboarding questionnaire (fields in spec, branching on entity type); do not collect ID or address docs unless Twilio requires (TO-7)
- [ ] Verify divert target always equals the trader's assigned service number (TO-5), checked via `*#61#`, `*#67#`, `*#62#`

## Test-call matrix (per trader, own network and handset)
- [ ] Conditions: not answered, busy, declined, off/airplane
- [ ] Callers: mobile, landline, withheld
- [ ] Do Not Disturb / Focus interaction
- [ ] Record caller ID and `ForwardedFrom` results; document failures with workaround

## Go-live
- [ ] Checklist signed off; trader acknowledges an urgent test alert
- [ ] Check divert-to-mobile charges on trader's tariff
