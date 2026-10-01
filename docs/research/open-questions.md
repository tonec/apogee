# Open questions

Status: living · Last updated: 2026-10-01

## Telephony
- Confirm one regulatory bundle per trader (current assumption). Draft a UK mobile bundle as Individual and as Business in the Console and record exactly which fields and documents each asks for.
- Does an Individual mobile bundle need an address with documents, or only a mobile number? Does Business need a website URL?
- Are sole traders ever rejected as Individual, and what does Twilio reject on? (A forum thread suggests friction; unverified.)
- Subaccount per trader workable? Can bundles be cloned or reused across subaccounts?
- How often do UK carriers populate `ForwardedFrom`? (Log it in the pilot.)
- Does caller ID survive diverts from EE, O2, Vodafone and Three?
- How do Do Not Disturb/Focus modes interact with conditional diverts?
- Ring-time syntax per network (EE, O2, Vodafone, Three).
- Do networks charge traders extra for diverting to mobile numbers?
- Delivery and filtering of texts sent from a +447 Twilio number to UK mobiles.
- Do alphanumeric sender IDs need registration for the UK?

## Product
- Is the first text classed as a service message under PECR?
- Exact retention periods for raw messages and cards.
- Pilot trade and region; whether LSAs serve trades there (parked track).
- Will traders actually use their own voicemail? (Ask at onboarding.)
- Whether a link-instead-of-reply flow outperforms reply-by-text.

## Legal
- Data-processing agreement template; controller/processor roles confirmed by an adviser.
- Whether the voice message and text disclosure wording is sufficient.
