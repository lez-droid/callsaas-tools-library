# Server Spec — finalize_status

## Endpoint
POST /api/finalize-status

## Receives
```json
{
  "tenant_id": "norwood-family-dental",
  "record_id": "abc123",
  "status": "confirmed",
  "confirmed_date": "Tuesday 27 May at 10am",
  "callback_reason": ""
}
```

## What it does
1. Validates webhook secret
2. Finds record by record_id
3. Updates status field
4. If confirmed: sets confirmed_date
5. If callback_required: sets callback_reason, adds to callbacks collection
6. If cancelled: marks as cancelled

## Returns
```json
{
  "ok": true,
  "record_id": "abc123",
  "status": "confirmed"
}
```
