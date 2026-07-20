```markdown
# Oakfield AI Receptionist

An AI phone receptionist for a residential care home, built to answer inbound calls, extract structured caller details, triage urgency, and automatically follow up by WhatsApp — with every call logged for the front-of-house team.

---

## The problem

Care homes get a steady stream of inbound calls — families checking on a resident, prospective families enquiring about a place, staffing agencies, suppliers, and occasionally a genuinely urgent call about a fall or medical concern. Reception is often unstaffed for periods of the day, and there's no consistent way to capture who called, why, and whether it needs an immediate callback.

## What this does

1. **Answers the call** with a natural-sounding AI voice agent (Vapi + ElevenLabs)
2. **Holds a real conversation** — collects the caller's name, relationship to the home, callback number, and reason for calling, while staying strictly within reception-desk boundaries (no clinical information, no fee quotes, no bed availability confirmed)
3. **Detects urgency** in real time — falls, injuries, distress — and flags the call automatically
4. **Extracts structured data** from the conversation using Vapi's structured output feature
5. **Routes automatically** via Make.com:
   - Urgent calls → immediate WhatsApp alert with an emergency line
   - Standard calls → WhatsApp confirmation with a summary
   - Incomplete calls (caller hung up before giving a number) → flagged separately for manual follow-up
6. **Logs every call** to Google Sheets with caller details, urgency flag, duration, transcript, and call recording link

## Why WhatsApp instead of SMS

This build uses WhatsApp rather than SMS for the caller follow-up. Families are more likely to see and trust a WhatsApp message than an SMS from an unfamiliar number, it costs the home nothing per message, and the underlying automation is functionally identical — same trigger, same data, same routing logic. Swapping to SMS is a one-field change in the Twilio module.

(Note: a production WhatsApp deployment requires Meta-approved message templates for business-initiated messages outside a 24-hour reply window — this demo uses the Twilio WhatsApp sandbox to simulate that flow.)

## Architecture

![Architecture diagram](./architecture-diagram/Screenshot%202026-07-20%20at%2015.13.09.png)

```
Caller dials in
      ↓
Twilio (number) → Vapi (voice agent: GPT-4o-mini + 11Labs + Deepgram)
      ↓
Vapi extracts structured data (end-of-call-report webhook)
      ↓
Make.com scenario:
   1. Log call → Google Sheets
   2. Router — branch by urgency / completeness
        → Urgent    → WhatsApp (immediate alert)
        → Standard  → WhatsApp (confirmation)
        → Incomplete → Google Sheets (flagged for manual callback)
```

## Stack

- **Twilio** — inbound number, voice + WhatsApp messaging
- **Vapi** — voice agent orchestration, transcription (Deepgram), TTS (ElevenLabs), structured data extraction
- **Make.com** — webhook handling, routing logic, Google Sheets logging, Twilio/WhatsApp dispatch
- **Google Sheets** — call log and incomplete-call queue

## Guardrails built in

- **Never discusses resident health, condition, or whereabouts** over the phone — deflects to a callback from the care team
- **Never quotes fees or confirms availability** — takes the enquiry, doesn't commit the home to anything
- **Emergency detection** — falls, injuries, distress trigger the urgent branch and a same-message reminder to call 999 if needed
- **Incomplete-call capture** — if a caller drops off before giving a number, the call isn't silently lost; it's logged with the transcript and recording for a human to review
- **Phone number normalisation** — handles inconsistent formats from natural speech (leading zero, missing country code) before sending to Twilio
- **Failed-send handling** — Make's incomplete execution storage catches and queues any failed WhatsApp send for retry

## Repo contents

```
/architecture-diagram   — Make.com scenario screenshot
/prompt                 — full system prompt used by the Vapi assistant
/schema                 — structured output schema (JSON) for call data extraction
/sample-call-log        — anonymised example rows from the live call log
```

## Notes

This is a demo/portfolio build using a fictional client. Twilio numbers, WhatsApp sandbox, and API keys used in testing have been excluded from this repository.

---

Built by [Damilola Odunlami](https://github.com/YoDammy) — [Venly Labs](https://venlylabs.com)
```
