# Design: Caller intake

Spec: `spec.md` · Tasks: `tasks.md`. Prompt: `docs/system-prompt.md`.

## Overview
Inbound SMS -> store -> deterministic safety scan -> LLM extraction -> script engine picks the next fixed template -> send. The LLM never sees or writes outbound text.

## Components
| File | Role |
|---|---|
| `http/routes/sms.ts` | validate, dedupe on `MessageSid`, STOP handling (CI-10), store, enqueue `process_reply` |
| `intake/process-reply.ts` | job orchestrating the steps below |
| `intake/extract.ts` | `Extractor` interface; `LlmExtractor`, `FixtureExtractor` |
| `intake/schema.ts` | zod schema for extraction output |
| `intake/script.ts` | next-prompt selection, completion check |
| `intake/templates.ts` | the only source of outbound caller text (CI-1) |
| `intake/flags.ts` | existing-customer, complaint, price-question, spam detectors |
| `intake/postcode.ts` | UK postcode validate/normalise (CI-3) |

## Interfaces
```ts
interface Extractor { extract(text: string): Promise<Extraction> }
type Extraction = { name?: string; postcode?: string; problem?: string; summary?: string;
  urgency_hint?: 'routine'|'urgent'; hazards?: string[]; flags?: string[]; spam?: boolean }
type Template = (slots: Record<string,string>) => string
```
Outbound path is `send(templateKey, slots)`; there is no `send(rawString)` for caller messages, enforced by type and a lint rule (CI-1, CI-8).

## process_reply flow
1. Strip phone numbers from text (DP-3), wrap as untrusted data in the prompt (CI-8).
2. Run rules first (`rules/`) for safety keywords; send safety text if matched (see `urgency-and-safety`).
3. Extract; validate; on failure retry once then store raw text and emit `extraction_failed` (CI-2). Emit `extraction_ok` otherwise.
4. Merge into `job_cards.fields` (non-empty wins; postcode validated).
5. Detect flags: price question -> fixed reply (CI-6); existing customer/complaint -> flag, priority card, stop engaging (CI-7); spam -> log, no card (CI-9).
6. Completion check (caller number, postcode, problem): complete -> closing template, `complete`; else next missing prompt in order postcode, problem, safety, callback, capped by `max_question_turns` (turn counter on conversation).
7. MMS: save media URLs; fetch lazily by the card job; failure never blocks (CI-4).

## Timers
`nudge:{conv}` at `nudge_after_min`, `abandon:{conv}` at `abandon_after_min`, both scheduled at first text, cancelled implicitly by guards on state (CI-5). Reply cancels nothing explicitly; guard checks `last_inbound_at`.

## Data
`job_cards.fields` jsonb: name, postcode, problem, summary, photos[], flags[], urgency_reason. New: `conversations.turns int`, `opt_outs(trader_id, number, at)` for STOP (CI-10; sending checks this first).

## Config
`max_question_turns=4`, `nudge_after_min=7`, `abandon_after_min=30`, `extraction_timeout_s=10`, `llm_model` (use a small Claude model, structured output).

## Failure modes
LLM timeout or outage -> same path as extraction failure; conversation still completes on raw text if postcode regex matches. Postcode invalid -> re-ask once.

## Testing
50-100 labelled fixtures in `test/fixtures/replies.json` with expected extraction and expected flags; hand-check run records fabrication count. A test asserts every outbound body equals a template output (catches injection). FakeClock for nudge/abandon.
