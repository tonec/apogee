# Design: Data protection

Spec: `spec.md` · Tasks: `tasks.md`. Engineering design only; legal items stay with the solicitor.

## Overview
Minimise at the edges (logger, LLM input, events), enforce retention with scheduled jobs, and provide one tool that can find and delete a caller's data across all stores.

## Components
| File | Role |
|---|---|
| `core/log.ts` | logger accepting only IDs/enums; redaction guard for digits, postcodes and message bodies (DP-3) |
| `intake/redact.ts` | strips phone numbers from LLM input (DP-3) |
| `ops/retention/` | scheduled deletion jobs (DP-4) |
| `ops/dsar/` | find/export/delete by caller number (DP-5) |
| `card/photos.ts` | EXIF strip on ingest, delete on request (DP-11) |
| `docs/data-protection/` | DPA, privacy notice, sub-processor register, DPIA screen, breach process (DP-1, DP-2, DP-6, DP-7, DP-12, DP-13) |

## Retention jobs
Daily pg-boss job per class; periods are config with the spec's proposed defaults:
| Data | Default | Action |
|---|---|---|
| message bodies | 60 days | null body, keep metadata |
| photos | 60 days | delete from storage and Twilio media |
| call metadata | de-identify caller number after 90 days; delete at 12 months | hash/null number |
| outcomes, events | 12 months | delete; events contain no raw text |
| Twilio message logs/media | match messages | delete via API (`messages(sid).remove()`) |
Each run writes `deletion_log(id, class, count, ran_at)` (no personal data), and a dry-run mode prints counts.

## DSAR tool
`dsar find <number>` queries conversations, messages, cards, photos, events, alert_attempts and Twilio (list messages/calls by number); `export` writes JSON; `delete` removes everything and records a `deletion_log` row. Requests received go to the trader (controller), logged in `dsar_requests(received_at, forwarded_at, trader_id)` (DP-5).

## Controls
| Req | Implementation |
|---|---|
| DP-1/2/7 | go-live gate checks DPA flag and privacy page URL in first text |
| DP-3 | log wrapper + redaction + LLM request capture test |
| DP-8 | TLS, provider encryption, Fly secrets, separate DB roles (app, worker, read-only reporting, retention) |
| DP-9 | all list/lookup queries scoped by `trader_id`; test asserts cross-trader isolation |
| DP-10 | no recording/voicemail code; CI grep for `record` TwiML attributes |
| DP-4 | backups retained no longer than the longest deletion period plus a grace window; note in DPA |

## LLM handling
Send reply text only, numbers stripped, no names of the trader. Choose a provider tier with no training on API data and configurable retention; record in the register.

## Testing
Log-scan on a full fixture run (regex for UK numbers, sample names, message strings); captured LLM request asserted number-free; retention job on seeded old data checks Postgres, storage and a Twilio mock; DSAR search then delete returns empty on re-search.

## Open
Supplier DPAs, regions and transfer mechanisms; LLM retention terms; Twilio deletability; ICO fees.
