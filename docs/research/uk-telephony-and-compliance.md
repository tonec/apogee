# UK telephony and compliance notes

Status: draft · Last updated: 2026-10-01
Sources are provider docs and third-party guides; verify against current docs before building.

## Number types and capabilities (Twilio, UK)
| Type | Prefix | Capabilities |
|---|---|---|
| Mobile | +44 7 | Voice + SMS |
| Local | +44 1 / +44 2 | Voice (SMS support varies; check Console filters) |
| National | +44 3, 844, 870, 845 | Voice |
| Toll-free | +44 800 / 808 | Voice |
Use a UK mobile number per trader so the diverted call and the outgoing text share a number. Filter by Voice + SMS in the Console; avoid SMS-only numbers (cannot take the divert).

## Regulatory compliance (KYC)
- All UK long codes need an approved Regulatory Compliance bundle to send messages or make voice calls (new numbers since 27 May 2024; all numbers since 30 Sep 2024).
- Review generally up to 3 business days; some regions take weeks.
- Business bundles: authorised representative's phone must be a real mobile number, not a number from a provider like Twilio.
- **ISV vs Direct:** Direct Customer = business uses the number to communicate internally or with its own customers. ISV/Reseller/Partner = business uses Twilio in a product it sells to its customers. This product is **ISV**. The UK form asks ISVs whether numbers are assigned to end customers (answer: yes). Extra per-trader requirements not confirmed; see open questions.
- Consider a Twilio subaccount per trader (to confirm).

## Call forwarding (diverts) from UK mobiles
- Standard GSM codes: busy `**67*number#`, no answer `**61*number#`, unreachable `**62*number#`; cancel with `##67#`, `##61#`, `##62#`; check with `*#61#` etc. Set all three to the same destination.
- Ring time before no-answer divert can be set on some networks; syntax varies by network. Verify per network and handset.
- Declining a call counts as busy (iPhone). Not picking up is no-answer.
- Conditional diverts override network voicemail. O2: restore with its voicemail divert code; EE: `##004#` clears all conditional diverts.
- `ForwardedFrom` webhook parameter: only set on forwarded calls and depends on the forwarding carrier; not all carriers pass it.
- Caller ID may not survive forwarding; withheld numbers cannot be texted.

## SMS notes
- Alphanumeric sender IDs are one-way only (no replies). Provisioning requirements to check.
- UK SMS to landlines is converted to voice by Twilio; not rejected, but may not be usable by the caller.
- A US number cannot validly test UK delivery: international long codes have been blocked for A2P SMS to the UK since June 2023 (per a HighLevel/Twilio compliance note).

## UK GDPR and PECR (get proper advice)
- Trader is typically data controller; the service is processor: data-processing agreement needed.
- Privacy notice link in the first text; keep retention short; do not record calls in the prototype.
- Whether text-back counts as a service message vs marketing: confirm.
- Emergency wording in the first text (999; gas emergency 0800 111 999): verify current official wording before shipping.

## Sources
- https://www.twilio.com/en-us/guidelines/gb/regulatory
- https://www.twilio.com/docs/phone-numbers/regulatory/reading-regulations-for-the-uk-bundle
- https://www.twilio.com/docs/phone-numbers/regulatory/getting-started/console-create-new-bundle
- https://help.twilio.com/articles/226194807-United-Kingdom-Porting
- https://www.twilio.com/en-us/guidelines/gb/sms
- https://www.twilio.com/docs/voice/twiml and https://www.twilio.com/docs/voice/api/call-resource
- https://www.twilio.com/docs/messaging/compliance/toll-free/api-onboarding (ISV business identity)
- https://connection-technologies.co.uk/help/call-forwarding/call-forwarding-a-complete-guide
- https://connection-technologies.co.uk/help/call-forwarding/o2-call-forwarding
- https://call2sms.co.uk/guides/iphone-call-forwarding-guide.php
- https://safina.ai/en/guides/call-forwarding-ee/
- https://help.gohighlevel.com/support/solutions/articles/48001240411-action-required-international-sms-compliance-updates-uk-turkey
