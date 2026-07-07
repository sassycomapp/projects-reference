# Mybizz — The Vault System

**Scope:** Mybizz-wide. This is the single authoritative document on the Vault system — its architecture, pattern, enforcement rules, and implementation. All other documents reference this one; none duplicate or restate its content.

**Authority:** `payment-security-boundary-vault` ADR (`payment-security-boundary-vault.md`)

**Source:** Consolidated from `spec-security.md` (the generic security pattern) and the original `docs/vault-system.md` (the CS-specific implementation), merged into one authoritative document.

---

## 1. Two-Level Secrets Model

- **Anvil Secrets** holds exactly one item: a master encryption key, set at provisioning, never changed, used exclusively inside a dedicated vault-service module to encrypt/decrypt application-level secrets. No other code calls `anvil.secrets.get_secret()` directly.
- **The Vault** (an application-level encrypted Data Table) holds every other credential — client API keys, integration tokens. Retrieved only via a `get_vault_secret(name)` function, never accessed directly by integration modules.
- **Access control:** restrict Vault access to the Owner role, require step-up authentication (e.g. TOTP) on every open with no grace period, and send a notification on every access.

## 2. Secret Enforcement and Masking

Enforce at the point of persistence, not just at the point of retrieval — use a hard-fail assert so a secret can never be accidentally saved outside the Vault:

```python
_PAYMENT_SECRET_COLUMNS = frozenset({
    'stripe_secret_key', 'paystack_secret_key', 'paypal_secret'
})
assert not (_PAYMENT_SECRET_COLUMNS & set(data.keys())), (
    f"Secret key columns must not be saved here: "
    f"{_PAYMENT_SECRET_COLUMNS & set(data.keys())}"
)
```

`AssertionError` must always be re-raised before any broader exception handler — never swallowed.

**Masking:** any function returning configuration containing a secret field returns `'***'` for that field, never the real value. Client-side save methods must never include a secret field in an outbound payload, even a masked one — omit the field entirely rather than round-trip it.

## 3. What's Stored in the Vault (per application)

Each application defines its own Vault contents. The following is the mb-3-cs (Consulting & Services) implementation:

| Secret | Used by |
|---|---|
| `stripe_secret_key` | Payment Gateway (Stripe) |
| `paystack_secret_key` | Payment Gateway (Paystack) |
| `paypal_secret` | Payment Gateway (deferred to V2) |
| SMTP credentials / `smtp_password` | Email Configuration (Brevo SMTP) |
| Brevo API key (`brevo_api_key`) | Brevo CRM integration |
| Shared event push token | Mybizz_management event forwarding |

Publishable keys (e.g. Stripe publishable key) are safe for client-side use and are stored in the `payment_config` table, not the Vault.

**SMTP constants** (hardcoded — identical for all clients, not stored in Vault):
- Host: `smtp-relay.brevo.com`
- Port: `587` (STARTTLS)

## 4. VaultForm UI (mb-3-cs)

Standalone form accessed from Settings, visible to Owner only:
- TOTP verification notice, displayed prominently at the top.
- Stored Secrets list — name, last updated date, usage context. View (masked reveal) and Delete actions per item.
- Add New Secret form — name + password input (`hide_text=True`) at the bottom. Value never displayed again after saving.

Visual references: `wireframes/wireframe-settings-VaultForm.html`, `screens/screen-settings-VaultForm.html`.

## 5. Build Sequence (mb-3-cs)

Built in Phase 1, Stage 1.5 of `docs/build-plan.md`, after authentication and settings (Stages 1.0–1.4). Phase 2 (payments, bookings, services) depends on the Vault being functional first.

### Deliverables
- `vault` Data Table with encrypted secrets storage
- `encryption_key` set in Anvil Secrets
- `server_shared/encryption_service.py`
- `server_shared/vault_service.py`
- `server_shared/vault_totp_service.py`
- `settings/VaultForm`

### Completion Criteria
- `encryption_key` set in Anvil Secrets (only item there)
- VaultForm requires TOTP step-up on every open, no grace period
- Email notification sent to Owner on every Vault access
- Secret keys stored encrypted in `vault` table
- `get_vault_secret()` retrieves secrets correctly
- Payment config uses Vault for secret keys (not stored in `payment_config` table)
- Secret masking in `get_payment_config()` returns `'***'`

## 6. TOTP Recovery

If the Owner loses their TOTP device, the following recovery path applies:

1. Owner contacts Mybizz support (Mybizz_management)
2. Mybizz_management verifies the Owner's identity through out-of-band process
3. Mybizz_management resets the Owner's TOTP configuration
4. Owner re-enrolls TOTP on their new device
5. The reset event is logged in the Mybizz_management amendment log (append-only)

**Constraints:**
- Only Mybizz_management can reset TOTP — no self-service recovery
- The reset event is permanently logged for audit

**Source:** `anvil-platform-constraints` ADR, CEO review 2026-06-15 Section 3

## Related Documents

| Document | Relationship |
|---|---|
| `adr/adr-global/payment-security-boundary-vault.md` | Architectural decision — accepted 2026-03-15 |
| `adr/adr-global/anvil-platform-constraints.md` | Anvil secrets limits, Vault TOTP recovery |
| `adr/adr-global/brevo-replaces-zoho-email.md` | Vault credentials for Brevo |
| `adr/adr-global/observability-architecture.md` | Vault access audit logging |
| `adr/adr-global/mybizz-management-visibility.md` | Vault access event forwarding |
| `spec-security-architecture.md` | RBAC role limits, token security, regulatory posture |
| `docs/DESIGN.md` | VaultForm design specification |
| `docs/build-plan.md` | Stage 1.5: The Vault |
| `docs/scaffold-spec.md` | File/folder structure for vault components |
| `docs/authoritative-schema.md` | `vault` Data Table schema |
| `specifications/spec-testing.md` | Vault test scenarios |
| `specifications/spec-client-activation-runbook.md` | Client provisioning — Vault setup steps |

---

*spec-vault-system.md v3.0 — consolidated from spec-security.md (generic pattern) and vault-system.md (CS implementation) into the single authoritative Vault document.*
