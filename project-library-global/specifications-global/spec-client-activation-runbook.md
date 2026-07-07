# Client Instance Activation Runbook

**Authority:** `client-instance-architecture` ADR, `dependency-update-model` ADR, `blank-client-template` ADR, `mybizz-management-visibility` ADR  
**Version:** 1.1 — 2026-06-13 (updated following Test B — startup module and form architecture confirmed)  
**Applies to:** Every new client onboarding. No exceptions. Complete all steps in order.

---

## Prerequisites

Before beginning activation:

- [ ] Client has signed subscription agreement
- [ ] Payment method confirmed (Stripe or Paystack)
- [ ] Client's primary contact email confirmed
- [ ] Client's business name and preferred subdomain/domain confirmed
- [ ] `blank_client_template` is up to date with current schema

---

## Phase 1 — Provision the Anvil Instance

### Step 1 — Clone blank_client_template

1. Open the mybizz Anvil account IDE
2. Navigate to `blank_client_template`
3. Clone the app
4. Rename the clone to the client identifier format: `[client-name]-cs` (e.g. `acme-cs`)
5. Verify the clone contains:
   - [ ] All 36 Data Tables (empty)
   - [ ] Startup module present (one client module only — opens first form from master_template)
   - [ ] master_template set as dependency (stable branch)
   - [ ] No server modules of its own
   - [ ] No forms of its own
   - [ ] No Anvil Secrets set yet

### Step 2 — Set Anvil Secrets

In the new client instance, set the following in Anvil Secrets (IDE only):

- [ ] `encryption_key` — generate a new Fernet symmetric key; do not reuse from any other instance

> ⚠️ This is the only item that ever goes in Anvil Secrets. See `payment-security-boundary-vault` ADR and vault-system.md.

### Step 3 — Configure Domain

Choose one:

- [ ] **Custom domain** — configure DNS, add to Anvil app settings, verify SSL
- [ ] **Anvil subdomain** — set a meaningful subdomain in Anvil app settings (e.g. `acmeconsulting.anvil.app`)

Record the final URL — it is required in Phase 3.

---

## Phase 2 — Management Integration

### Step 4 — Generate Management Token

1. Generate a unique Bearer token for this client instance (use a cryptographically secure method)
2. Store the token in Mybizz_management client registry (encrypted field)
3. Store the token in the client instance management configuration table

### Step 5 — Register in Client Registry

In Mybizz_management, create a new client registry entry:

| Field | Value |
|---|---|
| `client_id` | Unique identifier (e.g. UUID) |
| `app_name` | Anvil app name (e.g. `acme-cs`) |
| `base_url` | Client instance URL (from Step 3) |
| `management_token` | Token generated in Step 4 (encrypted) |
| `subscription_status` | `active` |
| `plan` | Applicable pricing tier |
| `onboarded_date` | Today's date |

> Until Mybizz_management is built, maintain a manual register in a secure document.

### Step 6 — Verify Management Endpoints

Call the client instance's health endpoint to confirm it responds correctly:

```
GET [base_url]/management/health
Authorization: Bearer [management_token]
```

Expected: HTTP 200 with health payload.

- [ ] Health endpoint verified

---

## Phase 3 — Create Owner Account

### Step 7 — Create Owner User

In the client instance:

1. Create the Owner user account using the client's confirmed email address
2. Assign the `Owner` role
3. Confirm only one Owner account exists in the instance

> The Owner role is the only role with access to the Vault and Settings. See vault-system.md.

---

## Phase 4 — Guided Setup Session

Schedule a 30-minute onboarding call with the client. During the session, the client completes the following in their instance:

### Step 8 — Vault Configuration

Walk the client through setting up their Vault secrets. For each item, the client enters the value; you do not handle it.

**Payment Gateway (choose one or both):**
- [ ] `stripe_secret_key` — from client's Stripe dashboard
- [ ] `paystack_secret_key` — from client's Paystack dashboard

**Email:**
- [ ] `smtp_password` — from client's Brevo account (SMTP credentials)

**Other integrations as applicable.**

After each entry: confirm it is saved; the value is never displayed again.

### Step 9 — Payment Gateway Webhook Registration

For each active payment gateway, the client must register the webhook URL in their gateway dashboard:

**Stripe:**
- Webhook URL: `[base_url]/api/webhooks/stripe`
- Events: `payment_intent.succeeded`, `payment_intent.payment_failed`, `customer.subscription.updated`, `customer.subscription.deleted`

**Paystack:**
- Webhook URL: `[base_url]/api/webhooks/paystack`
- Events: `charge.success`, `subscription.disable`, `invoice.payment_failed`

- [ ] Stripe webhook registered (if applicable)
- [ ] Paystack webhook registered (if applicable)

### Step 10 — Brevo Configuration

In the client instance Settings:

- [ ] SMTP host, port, and username entered
- [ ] SMTP password confirmed present in Vault (from Step 8)
- [ ] Sender name and sender email address configured
- [ ] Test email sent and received successfully

### Step 11 — Business Profile and Branding

- [ ] Business name
- [ ] Business address and contact details
- [ ] Logo uploaded
- [ ] Brand colours configured (if applicable)
- [ ] Time zone set (stored as IANA identifier; UTC used for all Data Table storage)
- [ ] Currency and locale settings confirmed

### Step 12 — Services and Team Setup

Guide the client through initial data entry as appropriate:

- [ ] At least one service category created
- [ ] At least one service created
- [ ] Team members invited (if applicable)

---

## Phase 5 — Completion

### Step 13 — Send Welcome Communication

Send the client:

- [ ] Welcome email via Anvil built-in email (system notification tier)
- [ ] Platform URL and login credentials (if not already provided)
- [ ] Link to getting started guide
- [ ] Reminder of 30-day money-back guarantee terms and process

### Step 14 — Internal Records

- [ ] Client activation date recorded in client registry
- [ ] Subscription billing confirmed active in Stripe or Paystack
- [ ] Activation checklist filed/archived

---

## Post-Activation Notes

- The client instance will receive all master_template updates automatically — server logic, UI, and forms — as they are released to the `stable` branch. No client action is required. The only exception is Data Table schema changes, which must be applied manually per instance in the Anvil IDE.
- Release notification emails are sent by mybizz via Anvil built-in email.
- The client never has access to the Anvil IDE. All management of the instance is through the application UI (Owner role) or through mybizz operator tooling.
- Schema migrations to this instance must be applied manually if the Data Table schema changes after activation. This includes new tables, new columns, modified columns, and deleted columns.

---

## Rollback

If activation cannot be completed (client withdraws, payment fails, technical blocker):

1. Delete the client instance app from the mybizz Anvil account
2. Remove the client registry entry from Mybizz_management (or manual register)
3. Revoke the management token
4. Document the reason in mybizz records

---

*End of client-activation-runbook.md*
