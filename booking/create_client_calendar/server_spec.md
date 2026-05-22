# create_client_calendar Server Implementation

## Endpoint
POST /api/tools/create_client_calendar

## Purpose
Creates a new Google Calendar owned by the service account for a client. This solves the Google Workspace org policy issue where external services cannot write to user calendars.

## Architecture
- Service account creates and owns the calendar
- Calendar is shared with specified email addresses (client, Leslie, etc.)
- Returns calendar ID for storage in PocketBase tenant record
- No Workspace policy restrictions since service account owns the resource

## Request Payload
```json
{
  "business_name": "Norwood Family Dental",
  "tenant_id": "norwood-family-dental", 
  "share_with_email": "lez@callsaas.io"
}
```

## Implementation Steps
1. Authenticate with Google Calendar API using service account JWT
2. Create calendar with summary: "{business_name} - CallSaaS Bookings"
3. Set calendar description: "Automated booking calendar for {business_name} via CallSaaS"
4. If share_with_email provided, add as owner/editor via ACL
5. Return calendar ID in response

## Response Format
```json
{
  "success": true,
  "calendar_id": "083970e125f37bab753c7fb3e6e97ba6b67a78ef9c6fd7da17917657130823ba@group.calendar.google.com",
  "calendar_url": "https://calendar.google.com/calendar/embed?src=...",
  "message": "Calendar created and shared with lez@callsaas.io"
}
```

## Error Handling
- Invalid service account credentials → 500 with auth error
- Calendar API failure → 500 with Google API error
- Missing required fields → 400 with validation error

## Dependencies
- googleapis npm package
- GOOGLE_SERVICE_ACCOUNT_JSON environment variable
- Google Calendar API enabled on GCP project

## Testing
Use the same pattern as the working check_availability/book_appointment tools. The service account authentication and calendar creation logic is already proven working.