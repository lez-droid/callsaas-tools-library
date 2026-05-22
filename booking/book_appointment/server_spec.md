# Server Spec — book_appointment

## Endpoint
POST /api/tools/book_appointment

## Requires
- google_calendar_id set on tenant record in PocketBase
- slot_duration_minutes set on tenant record (default 60)
- GOOGLE_CALENDAR_KEY in server .env (service account JSON)
- Google Calendar API enabled on GCP project

## Receives
```json
{
  "tenant_id": "norwood-family-dental",
  "record_id": "abc123def456",
  "date": "2026-05-27",
  "time": "10:00am",
  "caller_name": "Sarah Johnson",
  "service_type": "check-up",
  "notes": "First visit, needs x-rays"
}
```

## What it does
1. Looks up tenant record — gets google_calendar_id, slot_duration_minutes
2. Converts date + time to ISO datetime (start and end times)
3. Creates Google Calendar event:
   - Title: "[caller_name] - [service_type]"
   - Description: Notes + contact details from CRM record
   - Start/end times based on slot_duration_minutes
4. Updates PocketBase booking record:
   - Sets status: "confirmed"
   - Adds google_calendar_event_id
   - Adds confirmed_datetime

## Returns
```json
{
  "ok": true,
  "booking_reference": "NOR-240527-001",
  "calendar_event_id": "abc123def456_google_event_id",
  "appointment_datetime": "2026-05-27T10:00:00+09:30",
  "duration_minutes": 60
}
```

## Error cases
- Slot no longer available → returns ok: false, error: "slot_unavailable"
- Calendar API failure → returns ok: false, error: "calendar_error"
- Invalid record_id → returns ok: false, error: "record_not_found"

## Status
PENDING — requires Google Calendar integration (Step 3 of build plan)