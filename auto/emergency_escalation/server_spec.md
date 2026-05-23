POST /api/tools/emergency-escalation
Auth: X-Webhook-Secret header

Request body:
  caller_phone  string  required  Caller phone (normalise with formatAusPhone)
  caller_name   string  optional  Caller name
  situation     string  required  Brief situation description
  vehicle       string  optional  Vehicle description
  location      string  optional  Caller's current location
  tenant_id     string  required  Tenant identifier

Tenant config field required:
  emergency_contact_phone  string  Mobile number of workshop owner/manager for urgent alerts

Logic:
  1. Load tenant emergency_contact_phone and name (business_name) from PB
  2. If emergency_contact_phone not set, fall back to tenant phone field
  3. Build SMS body:
     "URGENT CALLSAAS ALERT — {business_name}
      Caller: {caller_name or 'Unknown'} | {caller_phone}
      Situation: {situation}
      Vehicle: {vehicle or 'Not provided'}
      Location: {location or 'Not provided'}
      Time: {timestamp AEST}
      — Reply or call them back ASAP"
  4. Send SMS via Twilio: From TWILIO_FROM, To emergency_contact_phone
  5. Log escalation to PB leads collection (status: 'emergency')
  6. Return success regardless of Twilio delivery status (fire and forget — caller must not wait)

Response (success):
  {
    "sent": true,
    "alerted": "owner"
  }

Response (no emergency contact):
  {
    "sent": false,
    "reason": "no_emergency_contact"
  }

Agent behaviour after successful send:
  "I've flagged this as urgent and someone from the team will call you back as soon as possible.
  If you're in immediate danger, please call triple zero.
  Is there anything else I can help you with right now?"

Agent behaviour if tool fails or returns sent: false:
  "I want to make sure the team knows about this — can I take your name and number? I'll make sure it gets to them urgently."
  Then collect name + phone and log to PB leads.

Notes:
  - Do not retry on failure — one alert is enough; duplicate SMSes create confusion
  - Rate limit: max 3 emergency escalations per tenant per hour to prevent abuse
  - This tool must NOT be in the agent's tools list unless emergency_contact_phone is configured
    (gate at agent-build time in build_demo / provision_client_dashboard)
  - Future enhancement: also send escalation to a Telegram group or email
