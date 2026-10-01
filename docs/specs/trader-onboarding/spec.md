# Spec: Trader onboarding and telephony setup

Status: draft · Last updated: 2026-10-01
Depends on: `research/uk-telephony-and-compliance.md`.

## Purpose
Set up each pilot trader safely: a UK number, compliance paperwork, call diverts on their existing mobile, and clear expectations (including the loss of their own voicemail for diverted calls).

## Model
The trader keeps their own mobile number, which customers already dial. The service number is a hidden back-end number that receives calls only when the network diverts them. One UK mobile (+447) number per trader. The service is an **ISV** for Twilio compliance purposes.

## Information to collect
Business name; trader first name; authorised representative's real mobile number; backup contact; mobile network and handset (iOS/Android); working hours and quiet hours; service area (optional postcodes); callback promise wording for urgent and routine (e.g. "within 15 minutes", "by end of day"); whether they use voicemail; consent to pilot terms and data-processing agreement; business address and any details Twilio requires for the end customer.

## Steps
1. Draft a UK regulatory bundle in the Twilio Console (ISV: numbers assigned to end customers); confirm per-trader requirements. Allow up to 3 business days, possibly weeks.
2. Buy a +447 number with Voice + SMS capability; assign the approved bundle.
3. Configure the voice webhook, SMS webhook, status callback and voice fallback URL.
4. Set diverts on the trader's mobile for busy, no answer (ring time 15–20 seconds where the network supports it) and unreachable, all to the service number. Verify with `*#61#`, `*#67#`, `*#62#`.
5. Tell the trader plainly that their network voicemail no longer takes diverted calls, how to undo each divert (`##61#`, `##67#`, `##62#`; EE `##004#`; O2 voicemail restore code), and that callers hear an automated message then get a text.
6. Run the test-call matrix on the trader's own network and handset: not answered, busy, declined, phone off/airplane mode, from a mobile, a landline and a withheld number.
7. Check how Do Not Disturb / Focus modes interact with diverts on their handset.
8. Sign the data-processing agreement; send the privacy notice link.
9. Go-live checklist signed off: drills passed, backup contact tested, trader acknowledges an urgent test alert.
10. Offboarding: trader cancels diverts; number released; data deleted per retention policy.

## Requirements
- **TO-1 (MUST)** No trader goes live before the urgent-alert drill passes with their phone.
- **TO-2 (MUST)** Voicemail replacement is disclosed in writing before diverts are set.
- **TO-3 (MUST)** Each trader has their own number; a number's capabilities are verified (Voice + SMS) before purchase.
- **TO-4 (SHOULD)** Settings are editable per trader: hours, quiet hours, backup contact, promises, ladder timings, allow/block lists.
- **TO-5 (MUST)** Diverts are never set to a number other than the trader's assigned service number.

## Acceptance criteria
- A fresh trader can go from signed-up to a successful end-to-end test call within a day once the regulatory bundle is approved.
- The test-call matrix passes for all four conditions and the three caller types, or failures are documented with a workaround.

## Open questions
- Per-trader vs reusable regulatory bundle; subaccount structure.
- Divert-to-mobile charges on the trader's tariff.
