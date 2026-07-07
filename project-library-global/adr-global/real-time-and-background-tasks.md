# `real-time-and-background-tasks` ADR — Real-Time Updates and Background Task Architecture

**Status:** Approved  
**Date:** 2026-06-14  
**Authority:** Derived from platform-overview.md, `observability-architecture` ADR, `anvil-platform-constraints` ADR, integration-architecture.md

---

## Context

The platform requires two forms of asynchronous behavior:

1. **Real-time updates** — Users expect to see new bookings, payment confirmations, and notifications without manual page refresh
2. **Background processing** — Campaign enrollment, appointment reminders, health monitoring, and webhook processing must happen without blocking the user

Anvil has no native WebSocket push from server to client. Background tasks have plan-dependent limits. With 100 client instances each running multiple background tasks, the architecture must be designed for Anvil's actual capabilities.

---

## Decision

### Real-Time Updates: Client-Side Timer Polling

The platform uses client-side timer polling for near-real-time updates. A timer on key dashboard and list forms periodically calls a server function to check for new data.

**Pattern:**

```python
# In DashboardForm
import anvil.server
import time

class DashboardForm:
    def __init__(self):
        self.poll_timer = None
        self.last_check = None
    
    def form_show(self):
        self.start_polling()
    
    def form_hide(self):
        self.stop_polling()
    
    def start_polling(self):
        self.poll_timer = anvil.Timer(interval=30, callback=self.check_for_updates)
    
    def stop_polling(self):
        if self.poll_timer:
            self.poll_timer.stop()
    
    def check_for_updates(self):
        result = anvil.server.call('get_dashboard_updates', self.last_check)
        if result['has_updates']:
            self.update_display(result['data'])
            self.last_check = result['timestamp']
```

**Polling intervals by context:**

| Context | Interval | Rationale |
|---------|----------|-----------|
| Dashboard | 30 seconds | Balance between freshness and server load |
| Booking list (active) | 15 seconds | Higher frequency during active use |
| Background forms | 60 seconds | Lower priority; reduce server load |

**Why polling over alternatives:**

| Alternative | Reason not chosen |
|---|---|
| WebSocket push | Not available in Anvil |
| Streaming responses | Limited Anvil support; complex to implement |
| Server-sent events | Not natively supported in Anvil |

### Background Task Architecture

All asynchronous operations use Anvil's `@anvil.server.background_task` decorator. Background tasks have no timeout limit on the Business plan.

**Task inventory per client instance:**

| Task | Schedule | Purpose | Estimated Load |
|------|----------|---------|----------------|
| Health heartbeat | Every 15 minutes | Write health metrics to `health_log` table | Low |
| Campaign enrollment | Hourly | Process active campaign enrollments, send next email | Medium |
| Appointment reminders (24h) | Hourly check | Send reminders for appointments in next 24 hours | Medium |
| Appointment reminders (1h) | Hourly check | Send reminders for appointments in next 1 hour | Medium |
| Lifecycle stage updates | Daily 02:00 | Recalculate contact lifecycle stages | Low |
| Automated task generation | Daily 03:00 | Create follow-up tasks based on upcoming bookings | Low |
| Webhook processing | On event | Defer webhook payload processing from HTTP handler | Burst |

**Total per client instance:** ~6 scheduled tasks + event-driven webhook processing

**At 100 clients:** ~600 scheduled tasks + webhook bursts

**Anvil Business plan verification required:** Confirm that 600+ scheduled tasks across 100 apps on a single account is within limits. This is a blocking question before launch.

### Per-Task Failure and Retry Policy

**Source:** CEO review 2026-06-15 Section 2

Each background task must define its failure behavior. The following table specifies the retry policy and failure action for each task:

| Task | On Failure | Max Retries | Retry Interval | Failure Action |
|------|-----------|-------------|----------------|----------------|
| Health heartbeat | Log failure, continue | 1 | Next scheduled run (15 min) | Mark instance `"degraded"` in `health_log` after 3 consecutive failures |
| Campaign enrollment | Skip failed contact, continue | 3 | 5 minutes (exponential backoff) | Log to `error_log`, skip contact, continue with next |
| Appointment reminders (24h) | Retry, then skip | 3 | 10 minutes | Log to `error_log`, send on next hourly run if missed |
| Appointment reminders (1h) | Retry, then skip | 2 | 5 minutes | Log to `error_log` — shorter window means fewer retries |
| Lifecycle stage updates | Retry full batch | 2 | 30 minutes | Log to `error_log`, run on next daily cycle |
| Automated task generation | Retry full batch | 2 | 30 minutes | Log to `error_log`, run on next daily cycle |
| Webhook processing | Retry individual event | 5 | Exponential backoff (1m, 2m, 4m, 8m, 16m) | Mark webhook `"failed"` in `webhook_log`, alert Mybizz_management |

**General rules:**
- All retries use exponential backoff unless specified otherwise
- Maximum retry window: 1 hour for hourly tasks, 24 hours for daily tasks
- After max retries exhausted: log to `error_log`, mark task as `"failed"` in the relevant table
- Critical task failures (webhook processing) trigger an alert to Mybizz_management
- Non-critical failures (campaign enrollment) are logged but do not trigger alerts

### Task Priority for Graceful Degradation

**Source:** Engineering review 2026-06-16

If Anvil enforces per-account background task limits, the platform must degrade gracefully rather than fail silently. Tasks are assigned priority levels:

| Priority | Tasks | Degradation Strategy |
|----------|-------|---------------------|
| **P1 — Must run** | Health heartbeat, webhook processing | Never skip. If capacity is constrained, reduce heartbeat frequency from every 15 min to every 30 min. |
| **P2 — Should run** | Appointment reminders (24h, 1h), campaign enrollment | Stagger across clients (not all at the same minute). If still constrained, consolidate into a single task that runs sequentially. |
| **P3 — Nice to have** | Lifecycle stage updates, automated task generation | Consolidate into a single daily task that runs all three sequentially. Skip if capacity is full — run on next available window. |

**Consolidation fallback:** If per-account limits are confirmed, Priority 3 tasks merge into one `daily_maintenance` background task:

```python
@anvil.server.background_task
def daily_maintenance():
    update_lifecycle_stages()
    generate_automated_tasks()
    # Future: summary table computation
```

**Staggering:** Health heartbeat and campaign enrollment tasks should be scheduled at different minute offsets per client to avoid thundering herd. Example: client A heartbeat at :00, client B at :01, client C at :02, etc.

### Webhook Processing Pattern

Webhook handlers (Stripe, Paystack, Brevo) must return HTTP 200 immediately or the gateway times out. Processing is deferred to a background task.

**Pattern:**

```python
# In webhook handler
@anvil.server.http_handler(path='webhooks/stripe')
def handle_stripe_webhook():
    payload = anvil.server.request.body
    signature = anvil.server.request.headers.get('stripe-signature')
    
    # Validate signature
    if not verify_stripe_signature(payload, signature):
        return anvil.server.HttpResponse(status=400)
    
    # Store raw payload and dispatch to background task
    event_id = app_tables.webhook_log.add_row(
        source='stripe',
        payload=payload,
        status='pending',
        received_at=datetime.now(TZ.UTC)
    )
    
    anvil.server.launch_background_task('process_webhook', event_id)
    
    # Return 200 immediately
    return anvil.server.HttpResponse(status=200)

# In background task
@anvil.server.background_task
def process_webhook(event_id):
    event = app_tables.webhook_log.get_by_id(event_id)
    # Process the webhook payload
    # Update booking status, payment status, etc.
    event.status = 'processed'
```

---

## Consequences

- ✅ Near-real-time updates without WebSocket infrastructure
- ✅ All async operations use Anvil's native background task system
- ✅ Webhook handlers return 200 immediately — no gateway timeouts
- ⚠️ Polling adds server load — must be balanced against freshness needs
- ⚠️ Background task limits per Anvil plan are a hard ceiling — must be verified
- ⚠️ Client-side timers stop when the form is hidden — updates pause in background tabs
- ⚠️ No push notifications for critical events in V1 — polling is the only mechanism until Mybizz_management is built

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/observability-architecture.md`` | Health heartbeat task feeds observability metrics |
| ``adr/adr-global/anvil-platform-constraints.md`` | Background task limits and server call latency constraints |
| ``adr/adr-global/webhook-architecture.md`` | Detailed webhook handling pattern |
| `integration-architecture.md` | Section 4: Background Tasks (existing task inventory) |
| `docs/platform-overview.md` | Section 5.7: Email & Campaigns (campaign enrollment task) |
| `docs/deployment-procedures.md` | Operational procedures for task failure recovery |

---

*End of `real-time-and-background-tasks` ADR*
