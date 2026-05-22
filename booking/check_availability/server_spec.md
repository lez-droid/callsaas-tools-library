# Server Spec — check_availability

## Endpoint
POST /api/tools/check_availability

## Requires
- google_calendar_id set on tenant record in PocketBase
- slot_duration_minutes set on tenant record (default 60)
- GOOGLE_CALENDAR_KEY in server .env (service account JSON)
- Google Calendar API enabled on GCP project

## Receives
```json
{
  "tenant_id": "norwood-family-dental",
  "date": "2026-05-27",
  "service_type": "check-up"
}
```

## What it does
1. Looks up tenant record — gets google_calendar_id, slot_duration_minutes, trading hours
2. Generates all possible slots for that day (trading hours ÷ slot_duration)
3. Queries Google Calendar for existing events on that day
4. Removes slots that overlap with existing events
5. Returns remaining free slots

## Returns
```json
{
  "ok": true,
  "date": "2026-05-27",
  "available_slots": ["9:00am", "10:00am", "2:00pm", "3:00pm"],
  "slot_duration_minutes": 60
}
```

## Status
PENDING — requires Google Calendar integration (Step 3 of build plan)
