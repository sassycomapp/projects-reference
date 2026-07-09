# Mybizz — Integration Specification

**Scope:** Mybizz-wide. Covers all external service integrations — Brevo (email + CRM), Stripe (payments), Paystack (payments).
**Authority:** `brevo-replaces-zoho-email` ADR, `payment-security-boundary-vault` ADR, `webhook-architecture` ADR

---

## 1. Integration Architecture

All external service integrations follow the same pattern:

1. **Credentials stored in Vault** — secret keys encrypted at rest, never in config tables
2. **Server-side only** — integration code lives in server modules, never client code
3. **Response envelope** — all integration calls return `{'success': bool, 'data': ...}` or `{'success': bool, 'error': ...}`
4. **Error logging** — all integration failures logged to `error_log` table
5. **Retry logic** — exponential backoff for transient failures

---

## 2. Brevo Integration

Brevo is the single platform for transactional email, marketing campaigns, and CRM.

### 2.1 Credentials

| Vault Key | Purpose | Used By |
|---|---|---|
| `brevo_smtp_key` | SMTP authentication | Transactional email |
| `brevo_api_key` | REST API | CRM sync, campaigns, automation, webhooks |

### 2.2 SMTP (Transactional Email)

**Constants (hardcoded, identical for all clients):**
- Host: `smtp-relay.brevo.com`
- Port: `587` (STARTTLS)

**Client-specific configuration (in `email_config` table):**
- `from_email` — verified sender address
- `from_name` — sender display name

**Emails sent via Brevo SMTP:**
- Booking confirmations
- Appointment reminders (24h and 1h)
- Invoice delivery
- Payment receipts
- Welcome emails (post-onboarding)

**Error handling:**
- SMTP connection failures → retry 3 times with exponential backoff
- Authentication failures → log to `error_log`, alert owner
- Rate limiting (429) → skip and retry on next scheduled run

### 2.3 Campaigns API (Marketing Email)

**Endpoint:** `https://api.brevo.com/v3/emailCampaigns`

**Campaign types:**
- **Sequences** — automated email sequences triggered by events (new client welcome, appointment follow-up, re-engagement, referral request)
- **Broadcasts** — one-time email announcements sent to selected segments

**Integration pattern:**
1. Create campaign via Brevo API
2. Enroll contacts based on triggers
3. Track opens/clicks via Brevo webhooks
4. Handle unsubscribes automatically

**Error handling:**
- API rate limits (429) → skip contact, retry on next hourly run
- API errors → log to `error_log`, continue with next contact
- Maximum 3 retries per contact per campaign run

### 2.4 CRM API

Brevo's built-in CRM stores contacts. Mybizz syncs contacts from the client instance to Brevo.

**Sync pattern:**
- On contact creation → push to Brevo
- On contact update → push to Brevo
- On contact deletion → remove from Brevo (if applicable)

**Fields synced:**
- Email, first name, last name, phone
- Tags, status, lifecycle stage

---

## 3. Stripe Integration

### 3.1 Credentials

| Vault Key | Purpose |
|---|---|
| `stripe_secret_key` | Server-side payment processing |

**Public key** (safe for client-side): stored in `payment_config.stripe_publishable_key`

### 3.2 Payment Processing

**Flow:**
1. Client confirms booking
2. Server creates Stripe PaymentIntent
3. Client completes payment (Stripe Elements or redirect)
4. Stripe sends webhook to `/api/webhooks/stripe`
5. Webhook handler dispatches to background task
6. Background task updates booking status, generates invoice

**Test mode:** `payment_config.test_mode` flag. Test keys vs live keys distinguished by key prefix (`sk_test_` vs `sk_live_`).

### 3.3 Webhook Events

| Event | Action |
|---|---|
| `payment_intent.succeeded` | Mark booking CONFIRMED, generate invoice |
| `payment_intent.payment_failed` | Mark booking failed, notify customer |

**Webhook URL:** `[base_url]/api/webhooks/stripe`
**Signature verification:** Required on every webhook. See `webhook-architecture` ADR.

### 3.4 Error Handling

| Error | Action |
|---|---|
| Card declined | Show error to customer, allow retry |
| API timeout | Retry once, then fail booking gracefully |
| Invalid API key | Log to `error_log`, alert owner via notification |
| Webhook signature failure | Return HTTP 400, log to `webhook_log` |

---

## 4. Paystack Integration

### 4.1 Credentials

| Vault Key | Purpose |
|---|---|
| `paystack_secret_key` | Server-side payment processing |

**Public key** (safe for client-side): stored in `payment_config` table.

### 4.2 Payment Processing

**Flow:** Same as Stripe — create transaction, customer pays, webhook confirms.

**Currency:** Paystack processes in NGN, GHS, ZAR, KES, and other African currencies.

### 4.3 Webhook Events

| Event | Action |
|---|---|
| `charge.success` | Mark payment successful |
| `invoice.payment_failed` | Mark payment failed |

**Webhook URL:** `[base_url]/api/webhooks/paystack`

---

## 5. Gateway-Agnostic Service Layer

Both Stripe and Paystack use the same underlying service interface. The active gateway is selected in Settings (`payment_config.active_gateway`).

**Service interface:**
```python
def create_payment(amount, currency, description, metadata) -> dict
def verify_payment(payment_id) -> dict
def refund_payment(payment_id, amount=None) -> dict
```

Switching gateways later does not require reconfiguring services or bookings. The service layer routes to the correct gateway based on `payment_config.active_gateway`.

---

## 6. Secret Management

All integration secrets follow the two-level model defined in `spec-vault-system.md`:

1. **Anvil Secrets** holds exactly one item: `encryption_key`
2. **Vault** holds all integration credentials (Brevo, Stripe, Paystack)
3. **`payment_config`** holds only publishable keys (safe for client-side)
4. **`email_config`** holds only sender name and email (not credentials)

**Enforcement:** Any server function that saves payment config must assert that secret key columns are not present. See `spec-vault-system.md` §2.

---

## 7. Integration Error Logging

All integration failures are logged to the `error_log` table with:

- `function_name` — the integration function that failed
- `error_type` — exception class
- `error_message` — error details
- `severity` — "error" for transient failures, "critical" for configuration failures

---

*spec-integration.md v1.0 — Brevo, Stripe, and Paystack integration patterns.*
