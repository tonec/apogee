# Design: Trader onboarding and telephony setup

Spec: `spec.md` · Tasks: `tasks.md`.

## Overview
Onboarding is mostly an operational runbook with a thin CLI around Twilio and the `traders` table. No self-service UI in the prototype.

## Components
| Item | Role |
|---|---|
| `ops/onboard/cli.ts` | commands: `create`, `bundle-status`, `buy-number`, `configure`, `settings`, `go-live`, `offboard` |
| `ops/onboard/twilio.ts` | wrapper over Twilio regulatory bundle, number purchase and webhook config APIs |
| `ops/onboard/checks.ts` | go-live gate checks |
| `docs/runbooks/onboarding.md` (to write) | step list, divert codes per network, test-call matrix |
| `docs/runbooks/trader-guide.md` (to write) | plain-language one-pager given to the trader (TO-2) |

## Trader lifecycle
`signed_up` -> `bundle_submitted` -> `bundle_approved` -> `number_assigned` -> `diverts_verified` -> `drills_passed` -> `live` -> `offboarded`. Stored as `traders.status`; CLI refuses skipping steps.

## Gates enforced in code
| Command | Refuses unless |
|---|---|
| `buy-number` | bundle approved in Twilio, Voice+SMS capability confirmed (TO-3, TO-6) |
| `configure` | number owned; sets voice, SMS, status, fallback URLs (OM-4) |
| `go-live` | urgent drill result recorded for this trader's phone, backup contact tested, DPA signed, written voicemail disclosure recorded (TO-1, TO-2, DP-1) |
| any divert record | divert target equals `traders.twilio_number` (TO-5) |

## Data
`traders`: add `status`, `entity_type` (`sole_trader`|`ltd`), `network`, `handset`, `bundle_sid`, `subaccount_sid`, `dpa_signed_at`, `voicemail_disclosure_at`, `settings` jsonb (hours, quiet hours, promises, ladder, lists pointer). Regulatory documents are not stored locally unless Twilio requires them (TO-7); if so, a separate encrypted bucket with short retention.

## Settings
Edited via CLI with schema validation (hours, quiet hours, backup contact, promises, ladder timings, allow/block lists) (TO-4).

## Bundle branching
Individual for sole traders, Business for Ltd (fields per spec). Confirm real field lists by drafting one of each in the Console before coding the wrapper; keep wrapper thin and manual-fallback friendly. Allow one week between sign-up and go-live (TO-8).

## Test-call matrix
Checklist table in the runbook (not code): conditions x callers x DND; results recorded as events on a dedicated test conversation and in the runbook.

## Offboarding
Release number, delete data via retention tooling (see `data-protection`), record in `deletion_log`.

## Open
Actual bundle fields; subaccount per trader and bundle reuse; divert charges on tariffs.
