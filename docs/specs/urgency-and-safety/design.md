# Design: Urgency and safety

Spec: `spec.md` · Tasks: `tasks.md`.

## Overview
A pure, synchronous rules module over normalised raw text. It runs on every inbound reply before extraction and returns hazards, an urgency level and a reason. A resolver combines it with other signals using max().

## Components
| File | Role |
|---|---|
| `rules/keywords.json` | editable groups: `safety`, `urgent`, `vulnerable`, `qualifiers` (US-6) |
| `rules/normalise.ts` | lowercase, strip punctuation, common typo map, simple stemming |
| `rules/evaluate.ts` | `evaluate(text): {safety: string[]; urgent: Reason[]}` pure function |
| `rules/resolve.ts` | `resolveUrgency(signals): {level; reason}` |
| `rules/safety-text.ts` | sends the fixed safety template once per conversation |

## Interfaces
```ts
type Level = 'routine' | 'urgent'
type Signals = { rules: Level; model?: Level; repeatCall: boolean; safetyAnswerYes: boolean }
resolveUrgency(s: Signals): { level: Level; reason: string }   // level = urgent if any signal urgent (US-4)
```
No code path writes `urgency` from the model alone; `setUrgency()` only ever raises and rejects downgrades (US-1).

## Flow
1. SMS handler calls `evaluate(raw)` synchronously in `process_reply` step 2.
2. Safety hits: send safety template immediately (US-2), emit `safety_text_sent`, once per conversation (flag on conversation); ignores quiet hours and trader availability.
3. Resolve urgency; if raised and not already urgent: store `urgency` and `urgent_reason` (US-7), emit `urgency_set`, enqueue alert ladder start (see `trader-alerting`), set `alerting`.
4. Urgent reasons seen later (e.g. safety answer "Yes", repeat call) re-run resolve; can only raise (US-3).
5. Vulnerable-occupant: store boolean `vulnerable` only, never the quoted health text (data minimisation).

## Rule matching
Phrase patterns with word boundaries and simple proximity (e.g. `no (heating|hot water)` within 8 tokens of `(baby|elderly|old|ill|pregnant)`). Negation ("no gas smell") is deliberately not handled: false positives are acceptable, false negatives are not (US-4). Emergency wording is a template, not generated (US-5).

## Config
`rules/keywords.json` path, `safety_text_once=true`. Version string stored on each event to explain historical classifications.

## Failure modes
Rules file invalid at startup -> process refuses to start (fail loud). Evaluate never throws: on error treat as urgent and log.

## Testing
Same labelled fixture file as intake with urgency labels; assertions: zero false negatives, false positives counted. Property test: `resolveUrgency` is monotone (adding a signal never lowers the level). Timing test: safety text enqueued before extractor mock resolves.

## Open
Official emergency wording and electrical-fault guidance (verify before launch).
