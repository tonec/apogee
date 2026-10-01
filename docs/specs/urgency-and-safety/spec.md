# Spec: Urgency and safety rules

Status: draft · Last updated: 2026-10-01
Depends on: `caller-intake`. Feeds `trader-alerting`.

## Purpose
Decide urgent vs routine with deterministic rules, and send fixed safety advice for dangerous situations. When unsure, escalate.

## Requirements
- **US-1 (MUST)** Rules run on the raw caller text. The LLM may raise urgency (via `urgency_hint` or hazards), never lower it.
- **US-2 (MUST)** Safety text fires immediately, from keyword rules and before extraction completes, for: gas smell, smoke, burning, sparks, flooding near electrics. Fixed wording (to verify): "If there's a gas smell, leave the property now and call the gas emergency line on 0800 111 999. If anyone is in danger, or there is fire or smoke, call 999." Sent regardless of trader availability.
- **US-3 (MUST)** Urgent if any of: safety keywords; no power to the whole property; active leak; no heating or hot water with a vulnerable occupant (elderly, baby, ill); the caller says "emergency"; repeat call within the window; caller answers Yes to the safety question.
- **US-4 (MUST)** Routine otherwise. If rules and hints disagree, take the more urgent.
- **US-5 (MUST)** The system gives no diagnosis or advice beyond the fixed safety text.
- **US-6 (SHOULD)** The keyword lists are editable config, covered by the labelled fixture set, and reviewed after every pilot week.
- **US-7 (MUST)** The card records the urgency reason ("Urgent: no power to property").

## Starter keyword groups (illustrative; refine with fixtures)
- Safety: gas smell/"smell gas", smoke, burning, sparks, "fuse box" + burning, flood, "water through ceiling" near lights.
- Urgent: "no power", "power cut" (whole property), "burst", "leaking", "no heating"/"no hot water" + (baby|elderly|old|ill|pregnant), "emergency", "urgent".

## Acceptance criteria
- Every fixture scenario labelled urgent is classified urgent (zero false negatives allowed in the drill set); false positives are acceptable but tracked.
- Safety text is sent within seconds of a reply containing a safety keyword, before extraction.
- A model output of "routine" cannot downgrade a rules-based urgent classification.

## Open questions
- Official emergency wording; whether to add electrical-fault guidance (verify with an authoritative source).
