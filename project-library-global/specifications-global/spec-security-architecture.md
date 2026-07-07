# Mybizz — Security & Secrets Architecture

**Related ADRs:** `payment-security-boundary-vault` (Vault), `client-instance-architecture`, `mybizz-management-visibility`

**Note:** Reusable security patterns (the two-level secrets model, secret enforcement/masking,
RBAC/data access pattern, rate limiting) live in `spec-security.md`. The
regulatory baseline (POPIA, GDPR, PCI DSS, standard retention periods) is Mybizz-wide — see
`spec-regulatory-compliance-baseline.md`. This document contains the Mybizz-wide security architecture and token model that applies to every application.

---

## 1. RBAC — Role Limits

Five roles, per the standard RBAC pattern (see `spec-security.md`). The standard per-instance limits that apply to all Mybizz applications:

| Role | Access | Instance Limit |
|---|---|---|
| Owner | Full access including Vault, financial data, user management | 1 per instance |
| Manager | Operational management — bookings, customers, marketing, reporting. No Vault or financial config | 0–3 per instance |
| Admin | Booking and customer management. No financial reporting, settings, or Vault | 0–10 per instance |
| Staff | Own calendar and bookings only | 0–20 per instance |
| Customer | Own bookings, invoices, account in the customer portal | Unlimited |

## 2. Mybizz_management Token Security (per `mybizz-management-visibility` ADR)

Mybizz_management communicates with client instances via bidirectional HTTP endpoints secured
with per-client Bearer tokens. This is the standard Mybizz service-to-service authentication
pattern — there is no reusable pattern here beyond the general "use Bearer tokens for
service-to-service auth" idea, which isn't specific enough to be worth extracting.

### Token Types

| Direction | Token Type | Storage |
|---|---|---|
| Mybizz → Client | Per-client Bearer token (unique per instance) | Mybizz_management client registry (encrypted); client instance management config |
| Client → Mybizz | Shared event push token | Each client instance's Vault |

### Security Properties

- **Per-client isolation:** a unique management Bearer token per client instance — compromise of
  one instance does not affect others.
- **Encrypted storage:** management tokens stored encrypted in Mybizz_management's client
  registry.
- **Server-side validation:** all management endpoints validate the Bearer token before
  processing; invalid tokens receive HTTP 403.
- **Non-blocking event push:** client-to-Mybizz event push calls are non-blocking — failure to
  reach Mybizz_management does not block client instance processing.

### Token Lifecycle

- **Generation:** unique token generated at provisioning (see `client-activation-runbook.md` in
  `specifications/`).
- **Storage:** stored in both the Mybizz_management registry (encrypted) and the client
  instance's own config.
- **Rotation:** not automated in V1 — manual rotation via re-provisioning if compromise is
  suspected.
- **Revocation:** delete from both the Mybizz_management registry and the client instance config.

## 3. Regulatory Compliance

The regulatory baseline (POPIA, GDPR, PCI DSS, standard retention periods) is Mybizz-wide — see
`spec-regulatory-compliance-baseline.md`. The standard applicability:

- **PCI DSS applies** — Mybizz processes payments via Stripe and Paystack, placing
  it in SAQ A scope per the Mybizz-wide standard.
- **GDPR applies conditionally** — if an app serves EU customers (determined by actual client
  base, not fixed at build time).
- No data categories beyond the Mybizz-wide retention defaults are tracked.

---

*Security & Secrets Architecture v3.1 — Elevated from app-specific to Mybizz-wide. Multi-tenant language corrected per `dependency-based-not-multi-tenant` ADR. Regulatory content references `spec-regulatory-compliance-baseline.md`.*
