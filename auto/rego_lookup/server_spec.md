POST /api/tools/rego-lookup
Auth: X-Webhook-Secret header

Request body:
  plate      string  required  Registration plate (normalise: uppercase, strip spaces/dashes)
  state      string  optional  State abbreviation — improves accuracy for ambiguous plates
  tenant_id  string  required  Tenant identifier (used to select API key/provider)

Logic:
  1. Load tenant rego_api_provider and rego_api_key from tenants PB record
  2. If no API configured, return { found: false, reason: 'not_configured' }
  3. Forward request to configured provider:
     - RedBook Check: GET https://api.redbookcheck.com.au/lookup?plate={plate}&state={state}&key={key}
     - VehicleCheck: POST https://api.vehiclecheck.com.au/v2/plate with JSON body
  4. Parse and normalise response

Response (found):
  {
    "found": true,
    "plate": "ABC123",
    "state": "VIC",
    "year": 2019,
    "make": "Toyota",
    "model": "Camry",
    "series": "Atara SL",
    "body": "Sedan",
    "colour": "Silver",
    "fuel": "Petrol",
    "transmission": "Automatic",
    "rego_expiry": "2026-03-31"
  }

Response (not found):
  {
    "found": false,
    "reason": "not_found"
  }

Response (not configured):
  {
    "found": false,
    "reason": "not_configured"
  }

Agent behaviour on found:
  Confirm verbally: "So that's a 2019 silver Toyota Camry Atara SL — is that right?"
  Use vehicle details to pre-fill booking fields (vehicle = "2019 Toyota Camry")

Agent behaviour on not found / not configured:
  "I wasn't able to pull up that rego automatically. Could you let me know the make, model and year of your vehicle?"

Notes:
  - Normalise plate before querying: uppercase, strip non-alphanumeric
  - Log each lookup for billing reconciliation (tenant_id, timestamp, plate hash — not plain plate)
  - Cache results for 24h per plate to reduce API costs (rego details rarely change day to day)
  - Rego expiry: if expiry is within 30 days, agent may optionally mention it as a courtesy
