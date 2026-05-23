POST /api/tools/service-estimate
Auth: X-Webhook-Secret header

Request body:
  service_type   string  required  Free-text service description from caller
  vehicle_class  string  optional  passenger | suv | ute | van | truck | motorcycle
  tenant_id      string  required  Tenant identifier

PocketBase collection required: pricing_rules
  Fields: id, tenant_id, service_key, service_aliases (json array), vehicle_class, price_min, price_max, currency, notes, active

Logic:
  1. Load all active pricing_rules for tenant_id
  2. Fuzzy-match service_type against service_key and service_aliases
     - Normalise input: lowercase, strip punctuation
     - Match: 'logbook service', 'log book', 'scheduled service' → service_key: 'logbook_service'
     - Match: 'rwc', 'roadworthy', 'road worthy certificate' → service_key: 'rwc'
  3. If vehicle_class provided, prefer rule matching that class; fall back to class = 'all'
  4. If no match found, return { found: false }

Response (found):
  {
    "found": true,
    "service": "Logbook Service",
    "price_min": 180,
    "price_max": 280,
    "currency": "AUD",
    "vehicle_class": "passenger",
    "notes": "Includes oil, filter and multi-point inspection. Parts extra if required."
  }

Response (not found):
  {
    "found": false,
    "service_type_received": "transmission rebuild"
  }

Agent behaviour on found:
  "A logbook service for a passenger vehicle typically runs between $180 and $280 — that covers oil, filter and a multi-point inspection. Parts are extra if anything needs replacing. Prices can vary a bit depending on the condition of the vehicle. Would you like to lock in a time?"

Agent behaviour on not found:
  "I don't have a set price for that in front of me — that's one for the team to quote properly once they've had a look at the car. I can book you in for an assessment. Would that work?"

Seeding pricing_rules during onboarding:
  Ash should ask the workshop owner for their standard services and prices during onboarding,
  then POST each rule to POST /api/admin/upsert-pricing-rule (auth: X-Provision-Secret).

Common service keys to seed:
  logbook_service, oil_change, brake_pads_front, brake_pads_rear, brake_pads_full,
  rwc, tyre_rotation, wheel_alignment, transmission_flush, coolant_flush,
  battery_replacement, timing_belt, clutch_replacement
