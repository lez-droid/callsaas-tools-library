POST /api/tools/capacity-check
Auth: X-Webhook-Secret header

Request body:
  tenant_id   string  required  Tenant identifier
  days_ahead  number  optional  Days to look ahead (default: 5, max: 14)

Tenant config fields required (add to tenants PB collection):
  bay_count         int     Number of service bays (default: 2)
  slot_duration_mins int    Minutes per booking slot (default: 60)
  hours_open        string  Operating hours e.g. "08:00-17:00"
  days_open         string  JSON array e.g. ["Mon","Tue","Wed","Thu","Fri"]

Logic:
  1. Load tenant config: bay_count, slot_duration_mins, hours_open, days_open
  2. Determine date range: today through today + days_ahead (skip closed days)
  3. For each open day in range:
     a. Calculate total slots = (hours_open duration in mins) / slot_duration_mins * bay_count
     b. Query PB: SELECT count(*) FROM bookings WHERE tenant_id = ? AND slot LIKE '{date}%' AND status NOT IN ('abandoned','cancelled')
     c. available_slots = total_slots - confirmed_count
     d. Classify: 'open' (>50% available), 'limited' (1-50% available), 'full' (0 available)
  4. Return per-day summary

Response:
  {
    "days": [
      { "date": "2026-05-26", "day": "Mon", "status": "full",    "available": 0,  "total": 8 },
      { "date": "2026-05-27", "day": "Tue", "status": "limited", "available": 2,  "total": 8 },
      { "date": "2026-05-28", "day": "Wed", "status": "open",    "available": 6,  "total": 8 },
      { "date": "2026-05-29", "day": "Thu", "status": "open",    "available": 7,  "total": 8 },
      { "date": "2026-05-30", "day": "Fri", "status": "open",    "available": 5,  "total": 8 }
    ],
    "next_open_day": "2026-05-28"
  }

Agent behaviour examples:
  Full week: "We're pretty well booked out this week — next available is Monday the 2nd. Does that work for you?"
  Mixed: "Monday's fully booked but we've got good availability from Wednesday onwards — does Wednesday or Thursday suit?"
  Wide open: "We have plenty of openings this week — what day works best for you?"

Onboarding: Ash seeds bay_count, slot_duration_mins, hours_open, days_open from onboarding conversation.
  Example prompt to workshop owner: "How many service bays do you have? What are your opening hours and days?"

Notes:
  - This tool gives a macro view (busy vs available per day), not individual time slots
  - Individual slot selection still uses the standard /api/reserve-tentative flow
  - Closed dates (public holidays, owner closures) should be handled by a future closure_dates table
  - If tenant config is missing defaults apply: 2 bays, 60-min slots, Mon-Fri 08:00-17:00
