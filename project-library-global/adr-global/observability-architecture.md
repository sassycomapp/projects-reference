# `observability-architecture` ADR — Observability Architecture

**Status:** Confirmed  
**Date:** 2026-06-13  
**Authority:** Derived from platform-overview.md, `mybizz-management-visibility` ADR, Anvil monitoring feature analysis

---

## Context

The platform requires visibility into the health, activity, and status of up to 100 client instances from a single operator view. Anvil provides built-in monitoring tools, but these are per-app and IDE-bound. External monitoring tools (Prometheus, Grafana) were considered.

### Anvil's Built-In Monitoring (as of 2026)

Anvil has recently introduced the following observability features:

- **App Logs** — session logs, background task logs, print() output, searchable with regex
- **CPU Usage and Memory Usage tabs** — introduced February 2026; show compute and memory consumption over time per app
- **Profiling and Tracing tools** — introduced January 2026; detailed visibility into performance bottlenecks

These tools are valuable for diagnosing individual app issues. They are not suitable as the primary monitoring solution for this platform because:

- They are **per-app** — viewing 100 client instances requires 100 separate IDE logins
- They provide **no cross-client consolidated view**
- They have **no alerting system** — no notification when a client app errors or goes silent
- They are **IDE-bound** — not accessible from a management application

---

## Decision

Client instance observability is achieved through **application-level metrics** written to each client's own Data Tables and queried by Mybizz_management via HTTP endpoints.

Mybizz_management is the consolidated observability layer — the platform's equivalent of a monitoring dashboard.

### What Client Instances Write

Master_template includes server-side logic that writes key events and metrics to designated tables within the client instance:

| Event type | Trigger | Written to |
|---|---|---|
| Health heartbeat | Scheduled background task | `health_log` table |
| Background task completion/failure | On task end | `audit_log` table |
| Payment event | On payment success/failure | `audit_log` table |
| Vault access | On every Vault open | `audit_log` table (existing, `payment-security-boundary-vault` ADR) |
| Error threshold exceeded | On unhandled exception | `error_log` table |
| Session activity | On user login/logout | `audit_log` table |

### What Mybizz_management Displays

Mybizz_management queries client instances via HTTP endpoints (see `mybizz-management-visibility` ADR) and presents:

- Last active timestamp per client
- Background task health (last run, success/failure)
- Error count and last error per client
- Subscription and billing status
- CPU and memory trend (from Anvil's per-app data, accessed via IDE or future API)

---

## Alternatives Rejected

| Alternative | Reason rejected |
|---|---|
| Prometheus + Grafana | External infrastructure; contradicts hosted simplicity; overkill for 100-client scale |
| Anvil App Logs (per-app) | No consolidated view; IDE-bound; no alerting |
| Manual per-app IDE monitoring | Not scalable at 100 clients; reactive not proactive |
| Third-party SaaS monitoring (Datadog, New Relic) | External dependency; cost; no native Anvil integration |

---

## Consequences

- ✅ Consolidated monitoring across all 100 client instances in one dashboard
- ✅ No external infrastructure or third-party monitoring subscriptions required
- ✅ Monitoring logic ships with master_template — all client instances monitored automatically
- ✅ Anvil's built-in tools remain available for deep per-app diagnostics when needed
- ⚠️ Mybizz_management must be built before consolidated monitoring is available — manual IDE monitoring applies until then
- ⚠️ `health_log` and `error_log` tables must be added to the 36-table schema and to `blank_client_template`
- ⚠️ Heartbeat background task adds to per-client scheduled task load — must be accounted for in Anvil plan limits

---

## Relationship to Anvil Built-In Tools

Anvil's built-in observability tools are not replaced — they are complementary:

| Tool | Use case |
|---|---|
| Mybizz_management dashboard | Day-to-day operational monitoring across all clients |
| Anvil App Logs | Deep diagnostic investigation of a specific client issue |
| Anvil CPU/Memory tabs | Performance investigation of a specific client instance |
| Anvil Profiling and Tracing | Code-level bottleneck analysis during development |

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/payment-security-boundary-vault.md`` | Audit log pattern (existing) extended by this ADR |
| ``adr/adr-global/mybizz-management-visibility.md`` | HTTP endpoint mechanism by which management queries client metrics |
| `docs/platform-overview.md` | Section 9: The Vault; audit log referenced |

---

*End of `observability-architecture` ADR*
