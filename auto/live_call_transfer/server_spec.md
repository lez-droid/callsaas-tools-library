LIVE CALL TRANSFER — SYSTEM TOOL
==================================
ElevenLabs Tool Type: System (built-in telephony)
No server endpoint required.

HOW IT WORKS
ElevenLabs provides native call transfer functionality when operating as a telephony agent
(inbound/outbound calls via SIP trunk or Twilio Voice). The agent calls the system tool,
which instructs the telephony platform to transfer the call trunk to the configured number.

The system tool type in ElevenLabs is "transfer_to_number" (exact name may vary by EL version).
Configure it via the ElevenLabs dashboard:
  1. Agent settings > Tools > Add system tool > Transfer call
  2. Set destination: tenant's on_call_phone number
  3. Set trigger description (used by LLM to know when to call it)

WHEN TO USE
- Safety emergency: caller's brakes have failed, vehicle smoking, stranded
- Caller explicitly asks to speak to a person
- Complex technical question outside the agent's scope
- After emergency_escalation SMS, if caller wants to stay on the line

WHEN NOT TO USE
- Routine bookings (agent handles this)
- Price inquiries (use service_estimate or Knowledge Base)
- After-hours calls where no human is available (use callback flow instead)

FALLBACK BEHAVIOUR
If telephony is not configured (demo widget, web embed):
  1. Use emergency_escalation tool to SMS the owner
  2. Tell caller: "I've flagged this as urgent and someone will call you straight back."
  3. Collect name and phone number if not already captured

COMPANION TOOL PAIRING
live_call_transfer and emergency_escalation serve different scenarios:

  Scenario: caller calls at 9am, brakes failed, workshop is open
    1. Call emergency_escalation (SMS to owner — "URGENT: caller on line, brakes failed")
    2. Call live_call_transfer (hand the live call to the owner immediately)

  Scenario: caller calls at 10pm, brakes failed, after hours
    1. Call emergency_escalation (SMS to owner's mobile — they can call back)
    2. Tell caller team will call back urgently, advise 000 if in danger
    3. Do NOT transfer — no one to take the call after hours

  Scenario: caller asks a complex engine rebuild question
    1. Call live_call_transfer (transfer to tech who can handle it)
    2. No SMS needed

TENANT CONFIG FIELDS
  on_call_phone       string  Primary transfer destination (e.g. workshop owner's mobile)
  after_hours_phone   string  Optional separate number for after-hours transfers (if staffed)

NOTE ON EL WIDGET VS TELEPHONY
The ConvAI widget (used in demo pages) is browser-based audio — it has no SIP trunk.
Call transfer is only available when ElevenLabs is the telephony layer (inbound call via EL phone number).
For S.J. Garage and future clients, this requires provisioning an EL phone number or SIP integration.
