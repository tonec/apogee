# Tasks: Urgency and safety

Build day: 4-5. Spec: `spec.md`. Needs: `caller-intake` inbound pipeline.

## Rules engine
- [ ] Keyword/rule config file (safety, urgent groups) loaded at startup, editable without code change (US-6)
- [ ] Text normaliser (case, punctuation, common misspellings, "gas smell"/"smell gas"/"smells of gas")
- [ ] Rules run on raw caller text before and independent of extraction (US-1)
- [ ] Urgency resolver: max(rules, model hint, repeat call, safety answer = Yes); never downgrade (US-4)
- [ ] Vulnerable-occupant rule (no heating/hot water + elderly/baby/ill/pregnant) storing only a flag, not health text (US-3)
- [ ] Urgency reason string stored and shown on card (US-7)

## Safety text
- [ ] Safety message template (final wording pending verification) sent immediately on safety keyword, before extraction completes (US-2)
- [ ] Send once per conversation, regardless of trader availability or quiet hours; emit `safety_text_sent`
- [ ] Emit `urgency_set` and hand off to alert ladder
- [ ] Confirm no diagnosis or advice anywhere beyond fixed text (US-5)

## Tests (acceptance)
- [ ] Labelled urgent/routine fixtures; zero false negatives on urgent set; track false positives
- [ ] Safety text sent within seconds of keyword reply, before extraction mock resolves
- [ ] Model "routine" cannot downgrade rules-urgent
- [ ] Review keyword lists after each pilot week (recurring)
- [ ] Verify official emergency wording and electrical-fault guidance against an authoritative source (open question)
