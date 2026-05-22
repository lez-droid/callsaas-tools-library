# Server Spec — book_appointment

## Endpoint
POST /api/tools/book_appointment

## Google Calendar Architecture
- Uses same service account as check_availability
- Creates calendar event on client's shared Google Calendar
- Stores calendar event ID in PocketBase for future reference

## Receives
```json
{
  "tenant_id": "norwood-family-dental",
  "record_id": "abc123def456",
  "date": "2026-05-27",
  "time": "10:00am",
  "caller_name": "Sarah Johnson",
  "service_type": "check-up",
  "notes": "First visit, no dental history"
}
```

## Processing Logic
1. Look up tenant record — get google_calendar_id, slot_duration_minutes
2. Parse date + time into ISO datetime
3. Calculate end time (start + slot_duration_minutes)
4. Create Google Calendar event
5. Update PocketBase record with calendar_event_id and status: "confirmed"
6. Return confirmation details

## Google Calendar Event Creation
```javascript
const event = {
  summary: `${caller_name} - ${service_type}`,
  description: `Service: ${service_type}\nNotes: ${notes}`,
  start: {
    dateTime: startDateTime.toISOString(),
    timeZone: 'Australia/Adelaide'
  },
  end: {
    dateTime: endDateTime.toISOString(),
    timeZone: 'Australia/Adelaide'
  },
  attendees: [
    {email: caller_email} // if provided
  ]
};

const result = await calendar.events.insert({
  calendarId: tenant.google_calendar_id,
  resource: event
});
```

## PocketBase Update
Update the existing record from reserve_tentative:
```json
{
  "status": "confirmed",
  "confirmed_date": "2026-05-27",
  "confirmed_time": "10:00am",
  "calendar_event_id": "abc123_google_event_id",
  "updated": "2026-05-22T14:30:00Z"
}
```

## Returns Success
```json
{
  "ok": true,
  "booking_reference": "NDF-240527-001",
  "calendar_event_id": "abc123_google_event_id",
  "confirmed_datetime": "Tuesday 27 May 2026 at 10:00am",
  "duration_minutes": 60,
  "message": "Appointment confirmed for Sarah Johnson"
}
```

## Returns Error
```json
{
  "ok": false,
  "error": "Time slot no longer available",
  "code": "SLOT_UNAVAILABLE"
}
```

## Error Codes
- NO_CALENDAR_ID — tenant calendar not configured
- RECORD_NOT_FOUND — invalid record_id from reserve_tentative
- SLOT_UNAVAILABLE — time slot was booked between check and confirm
- CALENDAR_API_ERROR — Google Calendar API error
- INVALID_DATETIME — date/time parsing failed

## Booking Reference Format
Generate unique reference: `[TENANT_PREFIX]-[YYMMDD]-[SEQUENCE]`
- TENANT_PREFIX: First 3 chars of business name (uppercase)
- YYMMDD: Year/month/day
- SEQUENCE: Daily counter (001, 002, etc.)

Example: "NDF-240527-001" for Norwood Family Dental

## Dependencies
- Same as check_availability
- PocketBase record must exist from reserve_tentative
- Calendar event ID stored for future cancel/reschedule operations

## Status
READY FOR BUILD — architecture defined, waiting for cc implementation