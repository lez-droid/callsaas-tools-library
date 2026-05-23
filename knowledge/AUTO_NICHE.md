AUTO NICHE KNOWLEDGE FILE
=========================
For use by Ash and the ElevenLabs voice agent when handling auto workshop calls.
Maintained by CC. Last updated: 2026-05-23.


INDUSTRY CONTEXT
----------------
Australian independent auto workshops are a high-volume, phone-dependent business. The typical
SMB workshop has 2-6 bays and handles 15-40 jobs per week. Inbound calls are the primary booking
channel — Google searches and word of mouth drive most inquiries. After-hours calls are common
(breakdowns happen at night, on weekends) and are almost always lost without an AI agent.

Workshop types relevant to CallSaaS:
- General mechanical (most common): services, repairs, roadworthy certificates
- Tyre and brake specialists
- European or Japanese specialists (Bosch Service agents, Toyota-trained)
- 4WD and commercial vehicle specialists
- Transmission specialists

The typical caller wants to:
1. Book a service or repair
2. Ask how much something costs
3. Check if a part is available or when the car will be ready
4. Pay an invoice
5. Report an urgent or safety-critical situation


KEY TERMINOLOGY
---------------
These terms are used by Australian workshop callers and must be understood by the agent:

Logbook service / log book service / scheduled service
  Manufacturer-specified service at set km intervals (e.g. every 10,000 km or 12 months).
  Stamps the service logbook — preserves new car warranty. Typically $180-$400 for passenger cars.

RWC / Roadworthy Certificate / Safety Certificate
  Government-mandated inspection required when selling a registered vehicle or re-registering
  a written-off vehicle. Different names by state: RWC (VIC, QLD), Pink Slip (NSW), Safety
  Certificate (QLD informal). Typically $80-$180.

Rego / registration / rego check / rego renewal
  Vehicle registration — annual fee paid to state government. Workshop does not handle rego
  renewals directly but may be asked. Agent should clarify: "We can do a roadworthy inspection
  if needed, but rego renewals go through the RMS [NSW] / VicRoads [VIC] / TMR [QLD]."

Warrant of Fitness (WoF)
  New Zealand term — not used in Australia. If a caller uses this term they may be a recent
  migrant. Equivalent to a roadworthy certificate.

COF (Certificate of Fitness)
  New Zealand heavy vehicle certificate. Not applicable in Australian workshops.

Pink slip
  NSW term for a safety check (annual vehicle inspection required with rego renewal for vehicles
  over 5 years old in NSW). Agent should recognise this as an inspection/safety check booking.

Green slip
  NSW term for CTP (Compulsory Third Party) insurance. Not provided by workshops. Agent:
  "Green slips are insurance — you'd get that from your insurer or the SIRA website."

Compulsory Third Party / CTP
  Not a workshop service. Redirect.

NEVDIS
  National Exchange of Vehicle and Driver Information System — the Australian national rego
  database used by rego_lookup. Callers won't know this term.

Mechanic Desk / Workshop Plus / Tekmetric / AutoGuru Workshop
  Workshop management systems (WMS) used by Australian workshops for job cards, invoicing,
  customer records. Integration with these systems enables caller_identity_lookup and
  get_invoice_status tools.

Repco / Burson / AutoBarn / Supercheap Auto
  Major auto parts suppliers in Australia. Repco (Bapcor) and Burson are trade-focused.
  Supercheap and AutoBarn are retail. Trade accounts with Repco or Burson are common for
  independent workshops — relevant for parts_availability integration.

Bay / hoist
  A service bay. Each bay can handle one vehicle at a time. Bay count drives workshop_capacity_check.

Job card
  The internal work order for a vehicle service or repair. Each vehicle gets a job card.

ETA / handback time
  When the vehicle will be ready for collection. Callers frequently ask "when will my car be ready?"
  — agent should take their number and note that a technician will call them with an ETA.

Drivability complaint
  Vague performance issue: shuddering, pulling, hesitation, rough idle. Agent should book a
  "diagnostic inspection" rather than trying to diagnose over the phone.

Dashboard light / warning light / engine light / CEL
  Check Engine Light / MIL (Malfunction Indicator Lamp). Book a diagnostic scan — typically
  $80-$150. Agent: "That's a diagnostic check — we plug in the scanner and read the fault codes
  to find out what's going on."


COMMON CALL TYPES AND AGENT RESPONSE GUIDE
-------------------------------------------

1. Booking a service
   Caller: "I want to book a service for my car."
   Agent: Confirm service type (logbook, oil change, or general service), vehicle, preferred day/time.
   Book via standard 3-step CRM flow.

2. Price inquiry
   Caller: "How much is a logbook service?"
   Agent: Call service_estimate if configured. Give range + disclaimer: "Prices can vary a bit
   depending on what the manufacturer specifies for your vehicle — that's a rough guide."
   Then: "Would you like to lock in a time?"

3. Urgency / same day
   Caller: "My brakes feel weird, can I come in today?"
   Agent: Call workshop_capacity_check for today's availability. If open, offer available time.
   If full: "We're fully booked today but I can fit you in first thing tomorrow."
   For safety-critical brake issues: check if caller says brakes have FAILED (not just squeaking) —
   if failed, call emergency_escalation.

4. Rego lookup
   Caller provides plate: "My rego is ABC123, it's a Toyota."
   Agent: Call rego_lookup with plate. Confirm vehicle details. "So that's a 2019 silver Camry —
   is that right?" This pre-fills the vehicle field in the booking.

5. Returning customer
   At call start, agent calls caller_identity_lookup with caller_id.
   If found: "Hi James, welcome back. Are you calling about the Camry today?"
   Then proceed to booking, skipping redundant identity questions.

6. Invoice / payment
   Caller: "I got a bill — can I pay over the phone?"
   Agent: Retrieve invoice via get_invoice_status. Confirm amount verbally. Call send_payment_link
   to SMS a Stripe link. "I've sent a payment link to your mobile — it should come through in a moment."

7. Car not ready / ETA query
   Caller: "Is my car ready yet?"
   Agent: Agent cannot check job status without WMS integration. Response: "I don't have the
   workshop floor visible from here — let me take your name and number and have the team call you
   back with an update. What's the best number?"
   Log as callback in PB callbacks collection.

8. Safety emergency
   Caller: "My brakes have completely failed / there's smoke coming from under the bonnet / I can't
   control the car."
   Agent: Call emergency_escalation IMMEDIATELY. Then: "I've flagged this as urgent — someone will
   call you back as soon as possible. If you're in immediate danger please call triple zero."
   Do NOT offer a booking slot.

9. Out-of-scope: towing
   Caller: "Can you organise a tow truck?"
   Agent: "We don't arrange towing directly — for a tow truck you can call 13 TOWS (1813 8697) or
   your roadside assistance provider like NRMA, RACV, or RAA."

10. Out-of-scope: parts retail
    Caller: "Do you sell [part] over the counter?"
    Agent: "We're a service workshop — we don't sell parts retail. If you need parts you could try
    Repco or Supercheap Auto. For fitting work, we're happy to book you in."


DEMO PLAYBOOK
-------------
Purpose: Demonstrate CallSaaS to a workshop owner. The demo shows a realistic inbound call scenario.

Recommended demo scenario:
  "A customer calls to book their 2021 Hyundai i30 in for a logbook service, asks about price,
  and wants to book for next Thursday morning."

What the demo should show:
  - Natural Australian voice (female default: ys3XeJJA4ArWMhRpcX1D)
  - Agent greets professionally in the business's name
  - Handles the logbook service inquiry
  - Gives a price range if service_estimate is configured (or says "I'd need to check with the team")
  - Books the appointment via the CRM flow
  - Sends an SMS confirmation with address and Google Maps link

Demo data to prepare:
  - business_name: "Northside Auto Service" (or owner's actual business name)
  - business_type: "auto workshop"
  - suburb: e.g. "Fitzroy North"
  - address: actual street address (required — always look up before building)
  - voice: female (default)

System prompt elements for auto demo agents:
  - Business name, address, phone
  - Services offered (logbook, tyres, brakes, RWC etc.)
  - Operating hours and days
  - Special notes (e.g. "we specialise in Japanese vehicles", "free courtesy car available")
  - Instruction to speak address out loud when asked, then offer SMS with Maps link


LIVE VS ROADMAP TOOLS
----------------------

LIVE (available for client deployment now):
  - Standard booking flow: reserve-tentative → update-details → finalize-status
  - SMS confirmation with address and Google Maps link (send_booking_confirmation)
  - Client alert SMS on confirmed booking

DRAFT (buildable on current PB infrastructure, no external integrations needed):
  - workshop_capacity_check — needs bay_count and hours_open on tenant config
  - emergency_escalation — needs emergency_contact_phone on tenant config

CONCEPT (requires external integrations — roadmap items):
  - caller_identity_lookup — needs WMS integration or PB booking history query
  - rego_lookup — needs NEVDIS reseller API (Redbook Check, VehicleCheck)
  - service_estimate — needs pricing_rules PB collection + dashboard UI to manage it
  - parts_availability — needs Repco PRONTO or Burson API + trade account
  - send_payment_link — needs Stripe + tenant Stripe account connection
  - get_invoice_status — needs Xero/MYOB OAuth2 integration + token refresh scheduler


INTEGRATION LANDSCAPE — AUSTRALIAN AUTO WORKSHOPS
---------------------------------------------------

Workshop Management Systems (WMS):
  Mechanic Desk — cloud-based, popular mid-market, has REST API
  Workshop Plus — common in QLD, has API
  Tekmetric — US origin, growing in AU, has API
  MaxxTraxx — common in SA/WA
  Autosoft / Drive — older systems, limited API support
  Many small workshops still use paper job cards or basic spreadsheets

Accounting:
  Xero — most common for AU SMBs, full REST API, OAuth2
  MYOB AccountRight / MYOB Business — second most common, has API
  QuickBooks — less common in AU

Parts Suppliers with Trade APIs:
  Repco (Bapcor) — PRONTO B2B platform, available to trade account holders
  Burson Auto Parts (Bapcor) — API available to trade accounts
  BNT (NZ, some AU) — API available
  AutoBarn — limited API

Rego / Vehicle Data:
  Redbook Check — vehicle history + rego data, AU coverage
  VehicleCheck.com.au — NEVDIS reseller
  CarHistory (PPSR) — encumbrance + write-off history

Roadside Assistance:
  NRMA (NSW/ACT), RACV (VIC), RACQ (QLD), RAA (SA), RAC (WA), RACT (TAS)
  Callers with roadside emergencies should be directed to their own provider.

Towing:
  1300 Towing, 13 TOWS, Jim's Towing (franchise) — agent should not book towing directly


PRICING BENCHMARKS — ROUGH GUIDES (2025-2026 AUD)
---------------------------------------------------
These are indicative ranges for agent responses when service_estimate is not configured.
Do NOT use these as firm quotes. Always add "prices may vary depending on the vehicle."

Logbook service (passenger car):        $180 - $380
Logbook service (SUV):                  $220 - $450
Oil and filter change only:             $80 - $150
RWC / roadworthy certificate:           $80 - $180
Pink slip (NSW safety check):           $40 - $90
Brake pads (front set, supply + fit):   $180 - $350
Brake pads (rear set, supply + fit):    $160 - $300
Brake discs (front, supply + fit):      $300 - $600
Tyres (passenger, supply + fit each):   $120 - $280
Wheel alignment (4-wheel):              $90 - $150
Tyre rotation:                          $40 - $80
Wheel balance (per tyre):               $20 - $35
Battery replacement (supply + fit):     $180 - $380
Diagnostic scan:                        $80 - $150
Timing belt replacement:                $600 - $1,200
Clutch replacement:                     $900 - $1,800
Transmission service (auto):            $200 - $400
Coolant flush:                          $120 - $200
Air conditioning regas:                 $120 - $250
Power steering flush:                   $120 - $180

These figures are for agent guidance only. Workshop owners set their own pricing.


CULTURAL NOTES FOR AGENT TONE
------------------------------
- Australian callers prefer a casual, direct, no-nonsense tone. Not overly formal.
- Use "no worries" and "happy to help" naturally — don't sound scripted.
- "Give us a call back" (plural "us") is standard Australian workshop speech.
- Don't say "absolutely" or "certainly" repeatedly — sounds American and robotic.
- Mild humour is fine: "Sounds like your car's trying to tell you something!" but don't overdo it.
- When something goes wrong: be upfront, don't over-apologise, just fix it.
- Price sensitivity: Australian SMB customers are price-conscious. Don't make pricing seem vague
  or evasive — give a range and be confident about it.
- Trust signals matter: mention experience, warranty on work, quality parts where relevant.
