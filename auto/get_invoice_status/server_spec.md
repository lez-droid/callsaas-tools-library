POST /api/tools/invoice-status
Auth: X-Webhook-Secret header

Request body:
  caller_phone   string  optional  Caller phone (normalise with formatAusPhone)
  rego_plate     string  optional  Vehicle rego plate (fallback lookup key)
  customer_name  string  optional  Customer name (last-resort lookup)
  tenant_id      string  required  Tenant identifier

At least one of caller_phone, rego_plate, or customer_name must be provided.

Logic:
  1. Load tenant accounting_provider, accounting_access_token, accounting_tenant_id from PB
  2. If not configured, return { found: false, reason: 'not_configured' }
  3. Query accounting system for contacts matching the lookup key:

     Xero:
       GET https://api.xero.com/api.xro/2.0/Contacts?where=Phone.PhoneNumber="{normalised_phone}"
       Headers: Authorization: Bearer {access_token}, Xero-tenant-id: {accounting_tenant_id}
       If not found by phone, try GET .../Contacts?searchTerm={customer_name}
     
     MYOB AccountRight:
       GET {accounting_endpoint}/api/v2/Contact/Customer?$filter=contains(PhoneNumber,'{normalised_phone}')
       Headers: Authorization: Bearer {access_token}

  4. From matched contact(s), get outstanding invoices (status = AUTHORISED or SUBMITTED, AmountDue > 0)
  5. Return up to 3 most recent outstanding invoices

Response (found — invoices outstanding):
  {
    "found": true,
    "customer_name": "James Wilson",
    "invoices": [
      {
        "invoice_number": "INV-0234",
        "date": "2025-11-14",
        "description": "Logbook service — 2019 Toyota Camry",
        "total_aud": 340.00,
        "amount_due_aud": 340.00,
        "status": "outstanding",
        "due_date": "2025-11-28"
      }
    ]
  }

Response (found — no outstanding):
  {
    "found": true,
    "customer_name": "James Wilson",
    "invoices": []
  }

Response (not found):
  {
    "found": false,
    "reason": "no_match"
  }

Agent behaviour (outstanding invoices):
  Summarise: "James, I can see an outstanding invoice for $340 — that's for the logbook service on your Toyota Camry on the 14th. Would you like me to send a payment link to your mobile?"
  If customer confirms: call send_payment_link with amount from invoice.

Agent behaviour (no outstanding invoices):
  "I can see your account is all clear — no outstanding amounts. Is there anything else I can help with?"

Agent behaviour (not found / not configured):
  "I'm not able to pull up invoice details at the moment — the team will be able to look that up for you. Can I take your name and number and have someone call you back?"

Token management:
  - Xero tokens expire every 30 minutes. Implement token refresh logic in the CRM server.
  - Store encrypted refresh token per tenant in PB (accounting_refresh_token field).
  - Refresh before each API call if access_token is expired.
  - Add token-refresh scheduler to scripts/token-scheduler.js alongside PocketBase token refresh.

Security:
  - Never return invoice line-item details that could reveal supplier costs
  - Return only customer-facing description and total amounts
  - Log all lookups: tenant_id, timestamp, lookup_key_type (phone/rego/name) — not the value
