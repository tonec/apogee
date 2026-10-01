# Tasks: Trader alerting

Build day: 4-5, drills day 7. Spec: `spec.md`. Needs: `urgency-and-safety`, `job-card` (link), state machine.

## Ladder
- [ ] Per-trader ladder config (default `[0,2,5,8]`), retry interval and window
- [ ] On urgency set: enqueue one job per ladder step with delays; each re-reads state (TA-1, TA-3)
- [ ] t+0: SMS with card link plus automated call to trader
- [ ] t+2: repeat call
- [ ] t+5: SMS and call to backup; if no backup, repeat trader call (TA-6)
- [ ] t+8: fixed fallback text to caller
- [ ] After ladder exhausted: slower retries until acknowledged or window ends (TA-4)
- [ ] Urgent alerts ignore quiet hours (TA-5)
- [ ] Record every attempt in `alert_attempts` and emit `alert_attempt` (TA-8)

## Automated call
- [ ] Voice webhook for outbound alert call: short summary read from template, `<Gather>` press 1 to acknowledge, repeat once, fixed UK voice
- [ ] Call status callbacks recorded (answered, busy, no-answer, voicemail detection if needed so an answering machine isn't treated as human)
- [ ] Sanitise problem summary for speech (length, digits, postcodes)

## Acknowledgement
- [ ] Ack via keypress, SMS reply "OK" (match trader number to open urgent conversation), card open (JC-3) (TA-2)
- [ ] Any ack cancels pending ladder jobs, sets `acknowledged`, emits event
- [ ] Ambiguous "OK" when trader has multiple urgent conversations: ack all, or reply with fixed disambiguation (decide)

## Routine cards
- [ ] Immediate text with link, or held until quiet hours end (TA-7)
- [ ] One reminder after `routine_reminder_min` if no ack

## Drills (launch gate, run before any real customer)
- [ ] Drill harness: ~20 scripted scenarios (gas smell, no power, active leak, routine, spam, repeat caller, silent caller)
- [ ] Three trader states each: phone off, busy, ignoring alert
- [ ] Injected-clock tests assert exact ladder timings
- [ ] Each ack method cancels pending jobs, no alerts afterwards
- [ ] Live drills with real UK phones; 100% of urgent cases reach a human or backup, any miss blocks launch
- [ ] Watch whether carriers flag repeated automated calls as spam; settle voice wording and speech rate
