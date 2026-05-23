POST /api/tools/parts-availability
Auth: X-Webhook-Secret header

Request body:
  part_description  string  required  Free-text part name from caller
  vehicle_make      string  optional  Make (improves part matching)
  vehicle_model     string  optional  Model (improves part matching)
  vehicle_year      string  optional  Year (improves part matching)
  tenant_id         string  required  Tenant identifier

Logic:
  1. Load tenant parts_supplier, parts_api_key, parts_account_id from PB tenants
  2. If not configured, return { available: false, reason: 'not_configured' }
  3. Build vehicle context string: "{year} {make} {model}"
  4. Query supplier API:

     Repco PRONTO:
       POST https://b2b.repco.com.au/api/v1/parts/search
       Headers: Authorization: Bearer {parts_api_key}, X-Account: {parts_account_id}
       Body: { "query": "{part_description}", "vehicle": "{vehicle_context}", "branch": "{tenant_branch_id}" }

     Burson Auto Parts:
       GET https://api.burson.com.au/v2/catalogue/search?q={part_description}&vehicle={vehicle_context}
       Headers: Api-Key: {parts_api_key}

  5. Parse response: check stock at local branch first, then nearest distribution centre

Response (in stock locally):
  {
    "available": true,
    "stock_location": "branch",
    "estimated_availability": "today",
    "part_name": "Bendix General CT Front Brake Pad Set",
    "part_number": "DB1234"
  }

Response (available from DC):
  {
    "available": true,
    "stock_location": "distribution_centre",
    "estimated_availability": "next_business_day",
    "part_name": "Bendix General CT Front Brake Pad Set",
    "part_number": "DB1234"
  }

Response (not available):
  {
    "available": false,
    "reason": "out_of_stock",
    "estimated_restock": "3-5 business days"
  }

Response (not configured):
  {
    "available": false,
    "reason": "not_configured"
  }

Agent behaviour:
  In stock locally: "We've got that part in stock — we can get you sorted today if you can come in."
  DC available: "That part can be in by tomorrow morning. I can book you in for tomorrow afternoon — does that work?"
  Not available: "That part's not currently in stock and would take a few days to arrive. I can book you in for [next available slot after estimated_restock] if you'd like."
  Not configured: Fall through silently — do not mention the API. Ask caller about urgency and book for next available slot.

Notes:
  - Do not read out part numbers to the caller
  - Log each lookup for cost tracking (parts API calls may have a per-query cost)
  - Response timeout 12s — supplier APIs can be slow; if they time out, treat as not_configured
