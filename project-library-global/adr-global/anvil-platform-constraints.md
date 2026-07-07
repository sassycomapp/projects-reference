# `anvil-platform-constraints` ADR — Anvil Platform Constraints and Design Boundaries

**Status:** Approved  
**Date:** 2026-06-14  
**Authority:** Derived from platform-overview.md, Issues to be resolved, Anvil platform documentation

---

## Context

The Mybizz CS platform is built on Anvil.works, a hosted full-stack Python platform. Anvil provides significant velocity advantages — built-in hosting, HTTPS, Data Tables, user authentication, and a visual designer — but imposes specific constraints that differ from traditional self-hosted Python deployments.

Every architectural decision in this platform must be made with awareness of these constraints. Designing against Anvil's parameters produces a faster, more reliable product than designing around them. The meta-decision is deliberate: velocity over migration path.

---

## Decision

The platform is designed to work within Anvil's specific parameters. All team members must understand the following constraints before building features. No feature should be built assuming capabilities that Anvil does not provide.

---

## Constraint Catalogue

### 1. Skulpt Runtime Limitations

Anvil's client-side Python runs on Skulpt (a Python-to-JavaScript compiler), not CPython. Not all Python 3 standard library modules work as expected.

**Known limitations:**

| Area | Issue | Workaround |
|------|-------|------------|
| `re` module | Subtle bugs and gaps vs CPython regex | Use server-side for complex regex |
| `json` module | Minor differences in edge cases | Use server-side for critical JSON processing |
| `datetime` | Arithmetic and timezone handling quirks | All date logic lives in server modules |
| Currency formatting | Client-side formatting unreliable | All money display goes through server functions |
| Number formatting | Locale-specific formatting gaps | Server-side formatting for all financial display |

**Rule of thumb:** Anything touching money or dates lives in server modules only. Client-side Skulpt is used for UI logic, form handling, and display formatting that does not involve calculation.

### 2. No Native URL Routing

Anvil does not provide built-in URL routing. Client-side navigation uses the community routing library (Harry Twose's Anvil Routing dependency).

**Dependency ID:** `3PIDO5P3H4VPEMPL`

**Scope:** Public website pages only. Admin and customer portal navigation uses the lambda/Link/open_form pattern (`navigation-lambda-link-open-form` ADR).

### 3. Data Table Row Access

Anvil Data Table rows are lazy-loaded. Iterating over a large table without `.search()` or `.limit_to()` will timeout on large datasets.

**Rule:** Never iterate over all rows. Always use `.search()` with filters or `.limit_to()` for bounded queries.

### 4. Server Call Latency

Every `anvil.server.call()` is an HTTP round-trip. Over-reliance on server calls degrades UX — each call adds 100-500ms depending on server warm-up and network conditions.

**Rule:** Minimize server calls in user-facing flows. Batch operations where possible. See `form-architecture-and-state` ADR (Form Architecture and State Management).

### 5. Background Task Limits

Anvil's background task system has plan-dependent limits:

| Plan | Timeout | Scheduled Tasks | Notes |
|------|---------|-----------------|-------|
| Free | 30 seconds | None | Core features non-functional |
| Personal | Limited | Limited | Insufficient for production |
| Business | Unlimited timeout | Available | Required for Mybizz |

With 100 client instances each running multiple background tasks (health heartbeat, campaign enrollment, reminders, webhook processing), the per-account task load must be verified against Anvil Business plan limits.

**See:** `real-time-and-background-tasks` ADR (Real-Time and Background Tasks)

### 6. Git Integration Quirks

Anvil's built-in Git integration is one-way friendly (Anvil ← GitHub). Branching and merging workflows within the Anvil IDE are awkward. All branching and merging happens in GitHub, not in the Anvil IDE.

**Rule:** Never merge branches within the Anvil IDE. All merge operations happen in GitHub.

### 7. Secrets Management

`anvil.secrets` works but has size limits and no rotation tooling. The platform uses Anvil Secrets for exactly one item: `encryption_key`. All other secrets are managed through the custom Vault system (`payment-security-boundary-vault` ADR).

### 8. Media/File Handling

`BlobMedia` in Data Tables has row size limits that are not always obvious. Large file uploads may silently fail or cause performance issues.

**Rule:** Store file references (URLs) in Data Tables, not file contents. Use external storage for large media.

### 9. Persistent Server

Anvil's free tier has no persistent server. The Business plan includes persistent server capability, which is required for performance-sensitive operations.

### 10. `app_tables` Resolution Failure

When a server module in `master_template` calls `app_tables`, Anvil resolves the tables to the calling client instance's Data Tables. If the client instance is corrupted, misconfigured, or missing required tables, `app_tables` resolution may fail or return incomplete results.

**Failure behavior (documented):**
- If `app_tables` resolution fails, the server function must not crash with a 500 error
- Show the user a generic, non-technical error message: "Something went wrong. Please try again or contact support."
- Log the failure to `health_log` with status `"unhealthy"` and details of the missing table
- Alert Mybizz_management via the health monitoring system
- Do not expose internal error details to the client user

**Prevention:**
- `blank_client_template` must always have the complete table set
- Schema migrations must follow the checklist in `deployment-procedures.md`
- Health heartbeat task verifies critical table accessibility every 15 minutes

**Source:** CEO review 2026-06-15 Section 2

### 11. Vault TOTP Recovery

Vault access requires TOTP step-up on every open (`payment-security-boundary-vault` ADR). If the Owner loses their TOTP device, the following recovery path applies:

**Recovery process:**
1. Owner contacts Mybizz support (Mybizz_management)
2. Mybizz_management verifies the Owner's identity through an out-of-band process (government ID, registered email, security questions)
3. Mybizz_management resets the Owner's TOTP configuration
4. Owner re-enrolls TOTP on their new device
5. The reset event is logged in the Mybizz_management amendment log (append-only)

**Constraints:**
- Only Mybizz_management can reset TOTP — no self-service recovery
- The reset event is permanently logged for audit
- TOTP recovery is a V1 operational requirement, not a code feature

**Source:** CEO review 2026-06-15 Section 3

---

## Consequences

- ✅ Faster development velocity — no abstraction layers for hypothetical migration
- ✅ Simpler codebase — direct use of Anvil's native capabilities
- ✅ Better Anvil-specific optimisations — lazy loading, background tasks, dependency model
- ⚠️ Migration off Anvil would require a full rewrite — this is an accepted trade-off
- ⚠️ Anvil platform changes (deprecations, limit changes) directly affect the platform
- ⚠️ Team members must learn Anvil-specific patterns before building features

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/navigation-lambda-link-open-form.md`` | Navigation pattern that works within Anvil's constraints |
| ``adr/adr-global/payment-security-boundary-vault.md`` | Vault system replaces Anvil Secrets |
| ``adr/adr-global/client-instance-architecture.md`` | Dependency model — the core architectural pattern |
| ``adr/adr-global/form-architecture-and-state.md`` | How forms work within Anvil's form model |
| ``adr/adr-global/real-time-and-background-tasks.md`` | Background task patterns within Anvil's limits |
| ``adr/adr-global/data-access-patterns.md`` | Data Table query patterns within Anvil's model |
| `docs/platform-overview.md` | Section 6: Technical Stack |
| `docs/deployment-procedures.md` | Anvil account-level failure scenarios and recovery |

---

*End of `anvil-platform-constraints` ADR*
