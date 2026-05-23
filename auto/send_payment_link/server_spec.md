POST /api/tools/send-payment-link
Auth: X-Webhook-Secret header

Request body:
  customer_phone  string  required  Customer mobile (normalise with formatAusPhone)
  customer_name   string  optional  Customer name
  amount_aud      number  required  Amount in AUD (must be > 0, max 5000 for voice-collected amounts)
  purpose         string  required  Payment description
  booking_ref     string  optional  Booking reference for reconciliation
  tenant_id       string  required  Tenant identifier

Logic:
  1. Load tenant stripe_secret_key from PB tenants (encrypted field)
  2. If not configured, return { sent: false, reason: 'not_configured' }
  3. Validate amount: must be > 0 and <= tenant.max_payment_amount (default: 500 AUD for deposits)
     - Reject amounts above max to prevent social engineering
  4. Create Stripe Payment Link:
     POST https://api.stripe.com/v1/payment_links
     - line_items: [{ price_data: { currency: 'aud', unit_amount: amount_aud*100, product_data: { name: purpose } }, quantity: 1 }]
     - metadata: { tenant_id, booking_ref, customer_phone, customer_name }
     - after_completion: { type: 'hosted_confirmation', hosted_confirmation: { custom_message: 'Thank you for your payment. See you at the workshop!' } }
  5. Send SMS via Twilio:
     To: normalised customer_phone
     From: tenant TWILIO_FROM
     Body: "Hi {name}! Here's your payment link for {purpose} at {business_name}: {stripe_url} — expires in 24 hours."
  6. Log payment link creation to PB (new collection: payment_links or append to bookings)

Response (success):
  {
    "sent": true,
    "payment_url": "https://buy.stripe.com/...",
    "amount_aud": 50,
    "sms_to": "+61412345678"
  }

Response (not configured):
  {
    "sent": false,
    "reason": "not_configured"
  }

Response (amount exceeded):
  {
    "sent": false,
    "reason": "amount_exceeded",
    "max_allowed": 500
  }

Agent behaviour on success:
  "Done — I've just sent a payment link to your mobile. It covers $50 for your booking deposit and expires in 24 hours. Is there anything else I can help you with?"

Agent behaviour on not configured:
  Do not mention payment links. Advise the caller the team will send through payment details separately.

Security notes:
  - amount_aud must be server-validated; never trust the agent to enforce limits
  - Log all payment link creations with IP, timestamp, tenant_id
  - Stripe webhook for payment completion: POST /api/webhooks/stripe (separate endpoint, verifies Stripe signature)
  - Do not expose Stripe secret key in any response or log
