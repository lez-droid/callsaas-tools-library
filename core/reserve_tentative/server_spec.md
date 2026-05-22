# Server Spec — reserve_tentative

## Endpoint
POST /api/reserve-tentative

## Auth
X-Webhook-Secret header must match WEBHOOK_SECRET env var

## Receives
```json
{
  "tenant_id": "norwood-family-dental",
  "preferred_date": "next Tuesday",
  "preferred_time": "morning",
  "service_type": "check-up",
  "caller_name": ""
}
```

## What it does
1. Validates webhook secret
2. Creates a new record in PocketBase `leads` collection with status "tentative"
3. Sets tenant_id, preferred_date, preferred_time, service_type, caller_name
4. Sets created timestamp

## Returns
```json
{
  "ok": true,
  "record_id": "abc123",
  "status": "tentative"
}
```

## Error cases
- Invalid secret → 401
- Missing required fields → 400
- PocketBase unavailable → 503
