# Mybizz — API Specification (HTTP Endpoints)

**Scope:** Mybizz-wide. Covers all HTTP endpoints exposed by client instances — webhook handlers and management endpoints.
**Authority:** `webhook-architecture` ADR, `mybizz-management-visibility` ADR, `anvil-platform-constraints` ADR

---

## 1. Endpoint Categories

Client instances expose two categories of HTTP endpoints:

| Category | Purpose | Authentication | Built In |
|---|---|---|---|
| Webhook handlers | Receive events from external services (Stripe, Paystack, Brevo) | Gateway signature verification | Phase 2 |
| Management endpoints | Mybizz_management communication with client instances | Bearer token | Phase 1 |

Both ship with master_template — all client instances have them from provisioning.

---

## 2. Webhook Endpoints

### 2.1 Stripe Webhooks

**URL:** `[base_url]/api/webhooks/stripe`
**Method:** POST
**Authentication:** Stripe signature header (`stripe-signature`)

**Events handled:**
- `payment_intent.succeeded` → Update booking status to CONFIRMED, generate invoice
- `payment_intent.payment_failed` → Mark booking as failed, notify customer
- `customer.subscription.updated` → Update subscription status
- `customer.subscription.deleted` → Cancel subscription

**Pattern:**
1. Validate signature
2. Check idempotency (prevent duplicate processing)
3. Log to `webhook_log` table
4. Dispatch to background task
5. Return HTTP 200 immediately

### 2.2 Paystack Webhooks

**URL:** `[base_url]/api/webhooks/paystack`
**Method:** POST
**Authentication:** Paystack signature header

**Events handled:**
- `charge.success` → Update payment status
- `subscription.disable` → Cancel subscription
- `invoice.payment_failed` → Mark payment failed

**Pattern:** Same as Stripe — validate, log, dispatch, return 200.

### 2.3 Brevo Webhooks

**URL:** `[base_url]/api/webhooks/brevo`
**Method:** POST
**Authentication:** Brevo webhook signature

**Events handled:**
- Email open tracking → Update `email_log` status
- Email click tracking → Log click event
- Unsubscribe events → Update contact subscription status

**Pattern:** Same as payment webhooks — validate, log, dispatch, return 200.

### 2.4 Webhook Handler Requirements

| Requirement | Rule |
|---|---|
| Response time | Must return HTTP 200 within 5 seconds |
| Signature validation | Reject invalid signatures with HTTP 400 |
| Idempotency | Check `webhook_log` before processing to prevent duplicates |
| Logging | All webhooks logged to `webhook_log` table |
| Processing | Deferred to background task — never process in HTTP handler |
| Error handling | Log failures to `webhook_log.error_message` |
| Reconciliation | Hourly task retries orphaned (unprocessed) webhook rows |

---

## 3. Management Endpoints

### 3.1 Authentication

All management endpoints require a Bearer token in the `Authorization` header:

```
Authorization: Bearer <management_token>
```

The token is validated against a value stored in the client instance's management configuration. Requests without a valid token receive HTTP 403.

### 3.2 Endpoints

#### GET /management/health

Returns current health status of the client instance.

**Response:**
```json
{
  "status": "healthy",
  "last_active": "2026-06-15T10:30:00Z",
  "error_count_24h": 0,
  "last_error": null,
  "background_tasks": {
    "health_heartbeat": {"last_run": "...", "status": "ok"},
    "campaign_enrollment": {"last_run": "...", "status": "ok"}
  }
}
```

#### GET /management/status

Returns subscription and feature flag status.

**Response:**
```json
{
  "subscription_status": "active",
  "plan": "launch",
  "features": {
    "bookings_enabled": true,
    "marketing_enabled": true,
    "blog_enabled": true
  },
  "onboarding_status": "completed"
}
```

#### POST /management/suspends

Suspends the client instance.

**Request body:** `{"reason": "subscription_lapsed"}`
**Response:** HTTP 200 on success

#### POST /management/unsuspend

Removes suspension from the client instance.

**Response:** HTTP 200 on success

#### POST /management/push-notification

Pushes a notification to the client instance.

**Request body:** `{"title": "...", "message": "...", "type": "info|warning|error"}`
**Response:** HTTP 200 on success

#### POST /management/push-update-alert

Notifies the client instance of an available update.

**Request body:** `{"version": "v1.2.0", "release_notes": "..."}`
**Response:** HTTP 200 on success

---

## 4. Server Function Pattern

All server functions (not just HTTP endpoints) follow the response envelope pattern:

```python
{'success': True, 'data': result}
{'success': False, 'error': msg}
```

See `anvil-platform-standards.md` §2.

---

## 5. Rate Limiting

HTTP endpoints are subject to the same rate limits as all server functions:

| Scope | Limit |
|---|---|
| Unauthenticated | 10 requests/minute/IP |
| Authenticated | 100 requests/minute/user |

Rate limits enforced via `rate_limits` Data Table. See `spec-security.md`.

---

## 6. Error Responses

All HTTP endpoints return standard error responses:

| HTTP Status | Meaning |
|---|---|
| 200 | Success |
| 400 | Invalid request (bad signature, malformed payload) |
| 403 | Authentication failed (invalid Bearer token) |
| 500 | Server error (logged to `error_log`) |

---

*spec-api-specification.md v1.0 — HTTP endpoint patterns for webhooks and management.*
