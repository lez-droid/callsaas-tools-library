# Server Spec — check_availability

## Endpoint
POST /api/tools/check_availability

## Google Calendar Architecture
- One GCP service account with Calendar API access
- Service account JSON key stored as GOOGLE_CALENDAR_KEY in server .env
- Each client shares their Google Calendar with service account email
- Calendar ID stored in PocketBase tenant record as google_calendar_id

## PocketBase Schema Requirements
Add to tenants collection:
```json
{
  "google_calendar_id": "abc123@group.calendar.google.com",
  "slot_duration_minutes": 60,
  "trading_hours": {
    "monday": {"start": "09:00", "end": "17:00"},
    "tuesday": {"start": "09:00", "end": "17:00"},
    "wednesday": {"start": "09:00", "end": "17:00"},
    "thursday": {"start": "09:00", "end": "17:00"},
    "friday": {"start": "09:00", "end": "17:00"},
    "saturday": null,
    "sunday": null
  }
}
```

## Receives
```json
{
  "tenant_id": "norwood-family-dental",
  "date": "2026-05-27",
  "service_type": "check-up"
}
```

## Processing Logic
1. Look up tenant record from PocketBase
2. Validate google_calendar_id exists (error if not configured)
3. Determine day of week, check trading_hours for that day
4. Generate all possible slots: trading_hours ÷ slot_duration_minutes
5. Query Google Calendar API for existing events on that date
6. Remove slots that overlap with existing events (including buffer time)
7. Return remaining free slots in human-readable format

## Google Calendar API Call
```javascript
const calendar = google.calendar({version: 'v3', auth: serviceAccount});
const events = await calendar.events.list({
  calendarId: tenant.google_calendar_id,
  timeMin: startOfDay.toISOString(),
  timeMax: endOfDay.toISOString(),
  singleEvents: true,
  orderBy: 'startTime'
});
```

## Returns Success
```json
{
  "ok": true,
  "date": "2026-05-27",
  "day_name": "Tuesday",
  "available_slots": ["9:00am", "10:00am", "2:00pm", "3:00pm"],
  "slot_duration_minutes": 60,
  "total_slots": 8,
  "booked_slots": 4
}
```

## Returns Error
```json
{
  "ok": false,
  "error": "Calendar not configured for this client",
  "code": "NO_CALENDAR_ID"
}
```

## Error Codes
- NO_CALENDAR_ID — tenant.google_calendar_id is empty
- INVALID_DATE — date format not recognised
- CALENDAR_API_ERROR — Google Calendar API returned error
- CLOSED_DAY — no trading hours set for this day of week

## Dependencies
- googleapis npm package
- Google Calendar API enabled on GCP project
- Service account with Calendar API permissions
- PocketBase tenant record with calendar configuration

## Status
READY FOR BUILD — architecture defined, waiting for cc implementation