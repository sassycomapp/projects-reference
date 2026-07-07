# `webhook-architecture` ADR — Webhook Architecture

**Status:** Approved  
**Date:** 2026-06-14  
**Authority:** Derived from platform-overview.md, `real-time-and-background-tasks` ADR, integration-architecture.md

---

## Context

Three external services POST webhooks to client instances:

| Service | Events | Criticality |
|---------|--------|-------------|
| Stripe | `payment_intent.succeeded`, `payment_intent.payment_failed`, subscription updates | High — payment status must be accurate |
| Paystack | `charge.success`, `subscription.disable`, `invoice.payment_failed` | High — payment status must be accurate |
| Brevo | Open tracking, click tracking, unsubscribe events | Medium — campaign analytics |

Each client instance is a separate Anvil app with a separate URL. Webhook URLs are:
- Stripe: `[base_url]/api/webhooks/stripe`
- Paystack: `[base_url]/api/webhooks/paystack`
- Brevo: `[base_url]/api/webhooks/brevo`

**Critical constraint:** Gateway webhook handlers must return HTTP 200 within a few seconds or the gateway times out and retries. Processing must be deferred.

---

## Decision

### Webhook Handler Pattern: Immediate 200 + Background Task

All webhook handlers follow the same pattern:

1. **Receive** the webhook payload
2. **Validate** the signature (prevents spoofing)
3. **Log** the raw payload to `webhook_log` table
4. **Dispatch** to a background task for processing
5. **Return HTTP 200** immediately

```python
@anvil.server.http_handler(path='webhooks/stripe')
def handle_stripe_webhook():
    payload = anvil.server.request.body
    sig_header = anvil.server.request.headers.get('stripe-signature', '')
    
    # 1. Validate signature
    if not verify_stripe_signature(payload, sig_header):
        return anvil.server.HttpResponse(
            status=400,
            headers={'Content-Type': 'application/json'},
            body='{"error": "Invalid signature"}'
        )
    
    # 2. Log raw payload
    event_row = app_tables.webhook_log.add_row(
        gateway='stripe',
        event_type=event.get('type', ''),
        payload=payload,
        signature=sig_header,
        verified=True,
        processed=False,
        received_at=datetime.now(TZ.UTC)
    )
    
    # 3. Dispatch to background task
    anvil.server.launch_background_task('process_stripe_webhook', event_row.get_id())
    
    # 4. Return 200 immediately
    return anvil.server.HttpResponse(
        status=200,
        headers={'Content-Type': 'application/json'},
        body='{"received": true}'
    )
```

### Background Task Processing

```python
@anvil.server.background_task
def process_stripe_webhook(event_row_id):
    event_row = app_tables.webhook_log.get_by_id(event_row_id)
    
    try:
        event = json.loads(event_row.payload)
        event_type = event.get('type', '')
        
        if event_type == 'payment_intent.succeeded':
            handle_payment_success(event)
        elif event_type == 'payment_intent.payment_failed':
            handle_payment_failure(event)
        elif event_type == 'customer.subscription.updated':
            handle_subscription_update(event)
        elif event_type == 'customer.subscription.deleted':
            handle_subscription_cancellation(event)
        
        event_row.processed = True
        event_row.received_at = datetime.now(TZ.UTC)  # Update with processing timestamp
        
    except Exception as e:
        event_row.error_message = str(e)
        # Push error event to Mybizz_management (`mybizz-management-visibility` ADR)
        push_management_event('webhook_processing_error', {
            'source': 'stripe',
            'event_id': event_row_id,
            'error': str(e)
        })
```

### Idempotency

Webhook delivery is not guaranteed to be exactly-once. Gateways may retry on timeout. Idempotency must be enforced:

- **Stripe:** Use `event.id` as idempotency key — check `webhook_log` before processing
- **Paystack:** Use `event.id` or `reference` as idempotency key
- **Brevo:** Use webhook event ID or timestamp-based dedup

```python
def handle_payment_success(event):
    payment_intent_id = event['data']['object']['id']
    
    # Idempotency check
    existing = app_tables.bookings.search(
        payment_intent_id=payment_intent_id
    )
    if existing:
        return  # Already processed
    
    # Process payment...
```

### Webhook Log Table

**Reconciled with authoritative schema (2026-06-16).** The authoritative schema is the source of truth.

| Authoritative Schema Column | Type | Purpose | `webhook-architecture` ADR Alias |
|-----|------|---------|---------------|
| `gateway` | string | `stripe`, `paystack`, or `brevo` | `source` |
| `event_type` | string | e.g., `payment_intent.succeeded` | — |
| `payload` | simpleObject | Raw webhook payload | `payload` |
| `signature` | string | Gateway signature header | — |
| `verified` | bool | Signature verified successfully | — |
| `processed` | bool | Processing completed successfully | `status` (mapped: `pending` = verified:false/processed:false, `processed` = processed:true, `failed` = verified:true/processed:false + error_message) |
| `error_message` | string | Error details if processing failed | `error_message` |
| `received_at` | datetime | UTC — when webhook was received | `received_at` |

**Idempotency:** Enforced via `event_type` + `gateway` combination (check for existing row before inserting). No separate `idempotency_key` column needed — the event type from each gateway is unique per event.

**Row ID:** Anvil auto-generates row IDs. No explicit `id` or `UUID` column needed.

---

## Implementation Hardening Notes

**Source:** Engineering review 2026-06-16

Three additions to the webhook pattern that prevent silent data loss.

### 1. Dispatch Failure Handling

The HTTP handler returns 200 immediately after dispatching the background task. If `launch_background_task()` fails, the `webhook_log` row is stuck at `"pending"` and the gateway won't retry (it already got 200).

**Fix:** Wrap dispatch in try/except:

```python
@anvil.server.http_handler(path='webhooks/stripe')
def handle_stripe_webhook():
    payload = anvil.server.request.body
    sig_header = anvil.server.request.headers.get('stripe-signature', '')
    
    if not verify_stripe_signature(payload, sig_header):
        return anvil.server.HttpResponse(status=400)
    
    # Idempotency check in HTTP handler (see below)
    idempotency_key = extract_stripe_event_id(payload)
    if idempotency_key:
        existing = app_tables.webhook_log.search(
            gateway='stripe',
            event_type=idempotency_key
        )
        if existing:
            return anvil.server.HttpResponse(status=200)
    
    event_row = app_tables.webhook_log.add_row(
        gateway='stripe',
        event_type=idempotency_key,
        payload=payload,
        verified=True,
        processed=False,
        received_at=datetime.now(TZ.UTC)
    )
    
    try:
        anvil.server.launch_background_task('process_stripe_webhook', event_row.get_id())
    except Exception:
        event_row.status = 'dispatch_failed'
        # Reconciliation task will retry these
    
    return anvil.server.HttpResponse(status=200)
```

### 2. Idempotency in HTTP Handler (Not Just in Business Logic)

The original pattern checks idempotency inside `handle_payment_success()` — after the background task starts. This prevents duplicate business logic but not duplicate task dispatch.

**Fix:** Check `webhook_log` for an existing row with the same `idempotency_key` in the HTTP handler, before adding a new row. This prevents duplicate `webhook_log` entries and duplicate background task dispatches.

The `idempotency_key` extraction depends on the gateway:
- **Stripe:** `event['id']` from the parsed payload
- **Paystack:** `event['id']` or `reference`
- **Brevo:** webhook event ID

### 3. Reconciliation Background Task

A low-priority background task runs hourly to find orphaned webhook rows:

```python
@anvil.server.background_task
def reconcile_webhooks():
    orphaned = app_tables.webhook_log.search(
        processed=False,
        received_at=tables.less_than(datetime.now(TZ.UTC) - timedelta(minutes=5))
    )
    for row in orphaned:
        try:
            anvil.server.launch_background_task('process_webhook', row.get_id())
        except Exception:
            pass  # Will be caught on next reconciliation run
```

This ensures webhooks are never silently lost due to dispatch failures.

---

## Consequences

- ✅ Webhook handlers return 200 immediately — no gateway timeouts
- ✅ Processing failures are captured in `webhook_log` with full payload for retry
- ✅ Idempotency prevents duplicate records from webhook retries
- ✅ Background task processing is non-blocking — client UI is not affected
- ⚠️ Webhook processing is asynchronous — payment status update is not instant (seconds delay)
- ⚠️ `webhook_log` table grows over time — retention policy needed (90 days recommended)
- ⚠️ Background task failures must be monitored — failed webhooks require manual retry or investigation

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/real-time-and-background-tasks.md`` | Background task architecture and limits |
| ``adr/adr-global/mybizz-management-visibility.md`` | Error events pushed to Mybizz_management |
| `integration-architecture.md` | Section 1: Payment Gateway Architecture; Section 3: Brevo CRM Integration |
| `docs/client-activation-runbook.md` | §4.9: Webhook URL registration during provisioning |

---

*End of `webhook-architecture` ADR*
