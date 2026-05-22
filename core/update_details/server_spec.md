# Server Spec — update_details

## Endpoint
POST /api/update-details

## Receives
```json
{
  "tenant_id": "norwood-family-dental",
  "record_id": "abc123",
  "caller_name": "Jane Smith",
  "caller_phone": "0412 345 678",
  "caller_email": "jane@email.com",
  "notes": "First visit, referred by Dr Jones"
}
```

## What it does
1. Validates webhook secret
2. Finds existing record by record_id in PocketBase leads collection
3. Updates: caller_name, caller_phone, caller_email, vehicle_rego, notes
4. Verifies tenant_id matches record (security check)

## Returns
```json
{
  "ok": true,
  "record_id": "abc123"
}
```
