# 03 — Payment Security Boundary: Secret Keys Deferred to the Vault
Date: 2026-03-15
Status: Accepted
Source: @authref/mybizz-architecture.md — Stage 1.4 discovery

---

## Context

During Stage 1.4 (Settings & Configuration), the SettingsForm collects payment gateway
configuration. The question arose of where to store gateway secret keys (Stripe secret
key, Paystack secret key, PayPal secret). Storing them in the `payment_config` Data
Table alongside public keys would expose secrets in a standard table with no encryption.

---

## Decision

**Stage 1.4 SettingsForm does not persist secret keys to the `payment_config` table.**
Secret keys are deferred entirely to Stage 1.5 — The Vault.

The SettingsForm shows the user a note explaining this deferral for each gateway.
This is a documented architectural boundary — not an oversight or incomplete work.

The Vault (`vault` Data Table) stores all API keys and credentials as encrypted values.
Only the `encryption_key` in Anvil Secrets is used for encryption/decryption — nothing
else goes in Anvil Secrets. The Vault is accessible to the Owner role only and requires
2FA verification before access.

### Enforcement pattern

The following guard must be present in any server function that saves payment config,
before any broader exception handler:

```python
_PAYMENT_SECRET_COLUMNS = frozenset({
    'stripe_secret_key', 'paystack_secret_key', 'paypal_secret'
})
assert not (_PAYMENT_SECRET_COLUMNS & set(data.keys())), (
    f"Secret key columns must not be saved here: "
    f"{_PAYMENT_SECRET_COLUMNS & set(data.keys())}"
)
```

`AssertionError` must always be re-raised before any broader handler — it must never
be swallowed:

```python
except AssertionError:
    raise  # Hard fail — architectural violation, must not be swallowed
except Exception as e:
    ...
```

### Secret masking rule

`get_payment_config()` must return `'***'` for any secret key that is set — never
plaintext. The same rule applies to `get_email_config()` for SMTP credentials.
The client must not transmit secret fields in outbound dicts even when masked —
defence in depth. `_save_payments()` on the client must exclude secret key fields
from the outbound dict entirely.

---

## Consequences

### Rules files affected

| File | Required content |
|---|---|
| `spec_vault.md` | Primary spec for Vault architecture, credential structure, Vault access pattern |
| `policy_security.md` | Secret masking rule, Vault-only credential storage mandate |
| `ref_anvil_coding.md` | AssertionError guard pattern — already present in §6 ✓ |
| `spec_architecture.md` | Payment security boundary, Vault architecture overview |

---

*End of `payment-security-boundary-vault` ADR*
