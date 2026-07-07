# Mybizz CS — Observability Requirements

**Authority:** `observability-architecture` ADR (Observability Architecture), `mybizz-management-visibility` ADR (Mybizz_management Visibility and Control)

---

## Overview

This document defines the observability requirements for the Mybizz CS platform. Observability is delivered through application-level metrics written to each client instance's Data Tables, queried by Mybizz_management via HTTP endpoints.

**Key architectural decision:** External monitoring tools (Prometheus, Grafana, Datadog) were evaluated and rejected as unnecessary at this scale and inconsistent with the hosted-only infrastructure model. See `observability-architecture` ADR.

Anvil's built-in observability tools (App Logs, CPU/Memory tabs, Profiling and Tracing) remain in use for per-app diagnostic investigation. They are complementary to, not a replacement for, the consolidated Mybizz_management monitoring layer.

## Logging Requirements

### Structured Logging

All server functions must use structured logging with the following fields:

**Entry/Exit Logging:**
```python
logger = get_logger()
logger.info("function_entry", function="get_vault_secret", secret_name=name)
# ... function logic ...
logger.info("function_exit", function="get_vault_secret", result_type=type(result).__name__)
```

**Error Logging:**
```python
logger.error("function_error", function="get_vault_secret", 
             error_type=type(e).__name__, error_message=str(e),
             secret_name=name, user_id=user_id)
```

### Log Levels

- **DEBUG:** Detailed debugging information (development only)
- **INFO:** General operation events (function entry/exit, successful operations)
- **WARNING:** Unexpected but handled conditions
- **ERROR:** Errors that don't prevent operation
- **CRITICAL:** Errors that prevent operation

### Log Format

All logs must be structured (JSON format) for easy parsing and aggregation:

```json
{
  "timestamp": "2026-05-26T10:30:00Z",
  "level": "ERROR",
  "function": "get_vault_secret",
  "secret_name": "stripe_secret_key",
  "error_type": "VaultSecretNotFoundError",
  "error_message": "Vault secret not configured",
  "user_id": "user_123",
  "request_id": "req_abc123"
}
```

## Metrics

### Application-Level Metrics (`observability-architecture` ADR)

Metrics are written to designated Data Tables within each client instance by master_template server modules. Mybizz_management queries these tables via management HTTP endpoints (`mybizz-management-visibility` ADR).

**Health metrics (written to `health_log` table):**
- Health heartbeat — scheduled background task, every 15 minutes
- Last active timestamp — updated on user login/logout
- Background task status — last run, success/failure per task

**Audit metrics (written to `audit_log` table):**
- Background task completion/failure events
- Payment success/failure events
- Vault access events (every occurrence)
- User session events (login/logout)

**Error metrics (written to `error_log` table):**
- Unhandled exceptions
- Error threshold exceeded events
- Webhook processing failures

### Business Metrics (Pre-Computed)

Complex aggregations are computed by background tasks and stored in summary tables (`data-access-patterns` ADR):

**Daily summary (`daily_summary` table):**
- Total revenue
- Revenue by service
- Booking count
- New contacts

**Service performance (`service_performance` table):**
- Revenue per service
- Booking count per service
- Completion rate per service

**Campaign metrics (`campaign_metrics` table):**
- Emails sent, delivered, opened, clicked
- Conversion rates
- Unsubscribe rates

## Alerts

### Mybizz_management Dashboard (`mybizz-management-visibility` ADR)

Mybizz_management presents a consolidated dashboard across all client instances:

- Last active per client
- Background task health (last run, success/failure)
- Error count and last error per client
- Subscription and billing status

### Alert Triggers

**Critical (operator action required):**
- Client instance fails to respond to health check within threshold
- Error rate exceeds threshold for a client instance
- Background task failure rate exceeds threshold
- Payment processing failure rate exceeds 10%

**Warning (monitor closely):**
- Client instance response time degraded
- Background task running longer than expected
- Error rate elevated but below critical threshold

### Anvil Built-In Tools (Complementary)

Anvil's per-app observability tools remain in use for deep diagnostic investigation:

| Tool | Use case |
|---|---|
| App Logs | Deep diagnostic investigation of a specific client issue |
| CPU/Memory tabs | Performance investigation of a specific client instance |
| Profiling and Tracing | Code-level bottleneck analysis during development |

## Log Retention

**Production:**
- Detailed logs: 90 days (via Anvil App Logs)
- Application metrics: retained in Data Tables (health_log, audit_log, error_log)
- Audit logs: 2 years (compliance requirement)

**Staging:**
- Detailed logs: 30 days

## Implementation Notes

### Structured Logging

Use Python's built-in `logging` module for server-side structured logging:

```python
import logging

logger = logging.getLogger(__name__)

def get_vault_secret(secret_name):
    logger.info("get_vault_secret called", extra={"secret_name": secret_name})
    # ... function logic ...
    logger.info("get_vault_secret completed", extra={"secret_name": secret_name, "result_type": type(result).__name__})
```

### Metrics Collection

Metrics are written directly to Data Tables via server functions:

```python
def write_health_heartbeat():
    """Write health heartbeat to health_log table."""
    app_tables.health_log.add_row(
        timestamp=datetime.now(TZ.UTC),
        status='healthy',
        last_active=datetime.now(TZ.UTC),
        background_tasks_running=get_running_task_count()
    )

def log_payment_event(event_type, amount, status):
    """Write payment event to audit_log table."""
    app_tables.audit_log.add_row(
        event_type=f'payment_{event_type}',
        timestamp=datetime.now(TZ.UTC),
        details=json.dumps({
            'amount': amount,
            'status': status
        })
    )
```

### Error Threshold Monitoring

```python
def check_error_threshold(client_id):
    """Check if error count exceeds threshold."""
    recent_errors = app_tables.error_log.search(
        timestamp=datetime.now(TZ.UTC) - timedelta(hours=1)
    )
    if len(recent_errors) > ERROR_THRESHOLD:
        push_management_event('error_threshold_exceeded', {
            'client_id': client_id,
            'error_count': len(recent_errors)
        })
```

---

*End of file — Observability Requirements*
