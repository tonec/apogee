# Spec: Data protection (UK GDPR)

Status: draft · Last updated: 2026-10-01
Depends on: all specs that store or send personal data (`caller-intake`, `job-card`, `observability-and-metrics`, `trader-onboarding`).

This is a working spec for engineering and for briefing a solicitor. It is not legal advice; every item marked TBC needs legal confirmation.

## Purpose
Know exactly what personal data the service handles, who is responsible for it, where it goes, how long it is kept, and what must be in place before real customers are involved.

## Roles (working assumption)
| Party | Role | For what |
|---|---|---|
| Trader | Controller | Callers' data (they decide why it is collected: to respond to enquiries) |
| This service | Processor | Callers' data, on the trader's instructions |
| This service | Controller | Trader account data (contact details, billing, regulatory bundle data) |
| Twilio, Fly.io, Supabase, LLM provider | Sub-processors | Telephony/SMS, hosting, database, extraction |

Rule: if the service uses caller data for its own purposes (for example sharing spam numbers across traders, improving models), it becomes a controller for that processing. In the pilot, do not do this.

## Data inventory
| Data | Source | Stored in | Purpose | Retention (proposed, TBC) |
|---|---|---|---|---|
| Caller number | Caller ID | Postgres, Twilio logs | Text back, merge repeats, job card | Card lifetime; Twilio logs deleted on schedule |
| Caller name | Reply | Postgres | Job card | Card lifetime |
| Postcode/address | Reply | Postgres | Job card, service-area check | Card lifetime |
| Problem description (verbatim and summary) | Reply | Postgres | Job card | Card lifetime |
| Photos | MMS | Twilio media, app storage | Job card | 30–90 days |
| Message bodies | SMS | Postgres, Twilio logs | Intake, audit | 30–90 days |
| Call metadata (time, status, ForwardedFrom) | Voice webhook | Postgres, Twilio logs | Metrics, debugging | 12 months, de-identified after 90 days |
| Outcome, job value | Trader taps | Postgres | Pilot metrics | 12 months |
| Events | System | Postgres | Metrics, safety | 12 months, no raw message text |
| Trader details (name, mobile, backup contact, business info) | Onboarding | Postgres | Service delivery, billing | Contract term plus period TBC |
| Regulatory bundle data (and ID/address documents if Twilio requires them) | Onboarding | Twilio; local copies only if unavoidable | Number provisioning | Delete local copies after approval |

**Sensitive content risk:** free-text replies may include health details ("my mum is ill"), other people's names, or photos showing people and home interiors. Do not ask for health information; store only a "vulnerable occupant" flag where the rules need it; treat all free text and photos as potentially sensitive.

## Data flow
Caller → mobile network divert → Twilio (voice, SMS) → Fly.io app → Supabase Postgres. Reply text (minimised, phone number removed) → LLM API → structured fields back to the app. Job card link and alerts → trader via Twilio SMS and automated calls.

## Sub-processor register (to complete and keep current)
| Supplier | Purpose | Data | Region | DPA available | Transfer mechanism |
|---|---|---|---|---|---|
| Twilio | Voice, SMS, MMS | Numbers, messages, photos, call metadata | TBC | TBC | TBC (US parent) |
| Fly.io | Application hosting | All app data in transit and memory | Prefer London | TBC | TBC |
| Supabase | Postgres | All stored data | Prefer London (eu-west-2) | TBC | TBC |
| LLM provider | Field extraction | Reply text only | TBC | TBC | TBC; check retention and training terms |

## Requirements
- **DP-1 (MUST)** A signed data-processing agreement with each trader before their number goes live (also `trader-onboarding` step 8).
- **DP-2 (MUST)** Written trader authorisation for sub-processors (general authorisation with notice of changes is acceptable if the agreement says so), and written contracts with each sub-processor giving equivalent protection.
- **DP-3 (MUST)** Data minimisation: collect only the fields in `job-card`; send the LLM only the reply text and minimal context, with phone numbers stripped; no personal data in application logs.
- **DP-4 (MUST)** Retention enforced by scheduled deletion jobs, including Twilio-side message logs and media via API, with the periods above as configurable defaults pending legal advice.
- **DP-5 (MUST)** Data subject requests: able to find, export and delete everything held about a caller number across Postgres, storage and Twilio. Requests received by the service are forwarded to the trader (controller) promptly and logged.
- **DP-6 (MUST)** Breach process: detect, record, and notify the affected trader without undue delay (timeframe fixed in the DPA).
- **DP-7 (MUST)** The first text links to a plain-language privacy notice naming the trader and the service.
- **DP-8 (MUST)** Security baseline: TLS everywhere, encryption at rest (provider defaults), secrets in a secrets manager, least-privilege database access, signed expiring card URLs, backups with retention aligned to DP-4.
- **DP-9 (MUST)** No cross-trader use of caller data (blocklists and allowlists are per trader in the pilot).
- **DP-10 (MUST)** No call recording or voicemail capture in the prototype.
- **DP-11 (SHOULD)** Strip EXIF metadata (including location) from stored photos; allow deletion of a photo on request.
- **DP-12 (SHOULD)** Document a DPIA screening decision (is it high-risk processing?) before the pilot.
- **DP-13 (SHOULD)** Check ICO registration or fee obligations for each role.

## Acceptance criteria
- A log scan over a fixture run finds no phone numbers, names or message text.
- A captured LLM request in test contains no phone number.
- Running the deletion job on seeded old data removes the Postgres rows, stored photos, and (via API) the Twilio-side messages and media.
- A search by caller number returns every record across Postgres, storage and Twilio, and a delete removes them.
- The DPA checklist (DP-1, DP-2, DP-7) is complete before any trader goes live.

## Questions for the solicitor
1. Are the roles above right (trader controller, service processor), and when would the service become a controller?
2. Retention periods for messages, photos, cards and events.
3. Breach notification timeframe and liability cap in the DPA.
4. Transfers to US sub-processors: which mechanism, and how the DPA should describe it.
5. Whether the first text and voice message wording is sufficient notice, and whether it is a service message under PECR.
6. Whether a DPIA is required, and any ICO registration or fee consequences.
7. Handling of free-text health information or photos showing people.
8. Terms for traders who also act as controllers for their own marketing.

## Open questions
- Each supplier's DPA, region, retention defaults and transfer mechanism (fill in the register).
- LLM provider terms: retention, training use, zero-retention options.
- Twilio default retention for message logs and media; whether they can all be deleted by API.
- Whether sole traders count as controllers needing ICO fee payment (they may already pay).
