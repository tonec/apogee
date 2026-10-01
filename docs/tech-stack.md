# Tech stack

Status: draft · Last updated: 2026-10-01

## Decisions

| Area | Choice | Rationale | Revisit when |
|---|---|---|---|
| Language/runtime | TypeScript on Node | Developer's strength; good Twilio SDK | n/a |
| HTTP framework | Hono (or Express) | Thin webhooks | n/a |
| Database | Postgres | State machine, events, reporting in one place | Multi-region or high volume |
| Jobs/timers | pg-boss or Graphile Worker | Timer-driven alert ladder without a separate service | Queue becomes a bottleneck |
| Telephony/SMS | Twilio, one UK mobile (+447) number per trader | Voice + SMS on one number; see research | Cost per number or shared-number viability |
| LLM | Any API with structured JSON output | Extraction and summary only | n/a |
| Phone validation | libphonenumber | Normalise caller and trader numbers | n/a |
| Job card UI | Server-rendered mobile page at a signed, expiring URL | No login, no PWA at prototype stage | Traders want accounts or history |
| Hosting | Persistent-worker host (Fly, Railway, Render) | Serverless makes timers awkward | n/a |

## Architecture rules
- **Webhooks respond fast; real work happens in queued jobs.**
- **Idempotent handlers:** key on `CallSid` and `MessageSid`; Twilio retries and delivers out of order.
- **Every queued job re-reads conversation state before acting.**
- **Inject a clock** so ladder timers can be tested without waiting.
- **Validate Twilio signatures** on every webhook (disabled only in local dev).
- **Outbound client behind an interface** so tests record calls instead of sending.
- **Config, not constants:** all timings and thresholds in config (see each spec).

## Initial data model
- `traders`: id, business_name, first_name, mobile, backup_contact, twilio_number, hours/quiet hours, service_area (optional), callback_promises, voicemail_used.
- `conversations`: id, trader_id, caller_number (nullable if withheld), state, urgency, created_at, updated_at.
- `messages`: id, conversation_id, direction, body, twilio_sid, status.
- `job_cards`: id, conversation_id, fields (jsonb), token, expires_at, outcome.
- `alert_attempts`: id, conversation_id, channel, target, sent_at, result.
- `events`: id, conversation_id, type, payload (jsonb), occurred_at. Append-only; source of all metrics.

## Testing approach
- **Replay fixtures** of Twilio voice, SMS and status payloads against the endpoints.
- **Labelled fixture set** of 50–100 sample replies for rules and extraction.
- **Mock outbound** SMS/voice clients; Twilio test credentials where they cover the call (check current docs).
- **Real UK phones** for divert, caller ID and delivery checks once the UK number is approved. A US number is fine for logic only; do not draw conclusions about UK delivery from it.

## Hosting and safety nets
- Voice fallback URL on each number returns a static message and hangs up. **Do not dial back to the trader's mobile** from the fallback: the call may divert straight back to the service and loop (to be verified).
- Scheduled synthetic call/text check; page the developer on failure.
- Sign card URLs and expire them; keep personal data out of logs; auto-delete raw messages after the retention period.
