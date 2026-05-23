POST /api/tools/caller-identity
Auth: X-Webhook-Secret header

Request body:
  caller_phone  string  required  Caller phone number (any format, normalise to E.164)
  tenant_id     string  required  Tenant identifier

Logic:
  1. Normalise caller_phone using formatAusPhone()
  2. Query PB bookings where tenant_id = tenant_id AND caller_phone = normalised_phone
     ORDER BY created DESC LIMIT 5
  3. If no bookings found, query PB leads collection same way
  4. If WMS integration configured (tenant has wms_endpoint + wms_api_key):
     - Forward to WMS API and merge results

Response (found):
  {
    "found": true,
    "customer_name": "Jane Smith",
    "vehicles": ["2019 Toyota Camry", "2022 Ford Ranger"],
    "last_service": "2025-11-14",
    "last_issue": "Brake pad replacement",
    "visit_count": 4
  }

Response (not found):
  {
    "found": false
  }

Response (error):
  {
    "found": false,
    "error": "lookup_failed"
  }

Agent behaviour on found:
  Greet by name: "Hi Jane, welcome back to [workshop]. Are you calling about the Camry or the Ranger today?"

Agent behaviour on not found:
  Standard greeting: "Thanks for calling [workshop], this is Ash. How can I help you today?"

Notes:
  - Always return HTTP 200 even if not found — the agent should not error on a 404
  - Timeout is 8s — keep query fast; if WMS call exceeds 5s, return PB-only result
  - Do not return any financial data in this endpoint
