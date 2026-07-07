# Mybizz CS — Integration Architecture

**Authority:** Mandatory — all integration implementation must conform

---

## 1. Payment Gateway Architecture

Two payment gateways supported in V1 (Stripe + Paystack). One gateway per client instance, configured at onboarding.

### Gateway-Agnostic Service Layer
Client code never calls a gateway directly. All payment operations go through a gateway-agnostic service layer in `server_payments/`. The service layer determines which gateway to call based on the `active_gateway` value in `payment_config`.

### Gateway Rationale
- **Stripe** — Anvil-native integration, primary international gateway
- **Paystack** — South African and African clients (Stripe does not operate in SA)

### Webhooks
Webhooks from all gateways are received at dedicated HTTP endpoints and verified by signature before processing. Payment events are logged to `webhook_log`. Idempotency must be enforced — duplicate webhook delivery must not create duplicate records.

### Payment Model
Payment is integrated into the booking flow. No separate checkout. Central-account only — all payments go to the business's single account. Per-provider payment accounts are not supported.

System currency is set at onboarding and immutable after the first transaction.

**See:** `webhook-architecture` ADR (Webhook Architecture), `anvil-platform-constraints` ADR (Anvil Platform Constraints)

---

## 2. Three-Tier Email Architecture

### Anvil Built-in Email
System-level platform events: login confirmations, password reset links, account notifications. Accessed via `anvil.email.send()`. Quota: approximately 1,000 emails/month per client instance. Comes from an Anvil-hosted address.

### Brevo SMTP — Transactional Email
Booking confirmations, appointment reminders, invoice delivery, contact form auto-replies. Originates from the client's business identity.

Connection constants (hardcoded in `transactional_email_service.py`):
- Host: `smtp-relay.brevo.com`
- Port: `587` (STARTTLS)
- Authentication: Brevo SMTP key from Vault (`brevo_smtp_key`)
- From address/name: from `email_config` table

Brevo is client-connected: each client creates their own Brevo account. Free tier: 300 emails/day, SMTP included.

### Brevo Campaigns — Marketing Email
Campaign sequences, broadcast emails, newsletter communications. Subject to unsubscribe management, open/click tracking.

Brevo API key stored in Vault (`brevo_api_key`). Separate from SMTP key — two distinct credentials per client.

**See:** `1-brevo-replaces-zoho-email.md`

---

## 3. Brevo CRM Integration

Optional convenience provided to clients. Each Mybizz client instance connects to its own separate Brevo organisation. Mybizz does not operate a shared Brevo org across all clients.

Free tier capabilities: 100,000 contacts, full API access, marketing automation workflows, contact history, pipeline. No per-user limitation.

CRM is automatically updated whenever a booking or service transaction is created via `contact_service.py`'s `update_contact_from_transaction()` function.

### Brevo Credential Monitoring

**Source:** CEO review 2026-06-15 Section 3

Each client creates their own Brevo account and stores SMTP and API keys in the Vault. If a client's Brevo account is compromised, an attacker could send emails as that client.

**Current state (V1):**
- Mybizz has no monitoring for unusual Brevo activity per client
- Brevo credentials are stored securely in Vault (encrypted at rest)
- TOTP step-up required to access Vault

**Documented intention (Mybizz_management scope):**
- Mybizz_management will monitor Brevo API usage per client instance
- Unusual activity patterns (volume spikes, new sender domains) will trigger alerts
- Credential rotation will be available through Mybizz_management
- This is out of V1 scope but is a documented operational requirement

**Mitigation in V1:**
- Vault TOTP gate prevents casual credential access
- Brevo's own account security (2FA if enabled by client) provides additional protection
- Client is responsible for securing their own Brevo account

---

## 4. Background Tasks

Seven scheduled and event-driven background tasks (see `real-time-and-background-tasks` ADR):

| Task | Schedule | Purpose |
|------|----------|---------|
| Health heartbeat | Every 15 minutes | Write health metrics to `health_log` table |
| Campaign enrollment | Hourly | Process active campaign enrollments, send next email |
| Appointment reminders (24h) | Hourly check | Send reminders for appointments in next 24 hours |
| Appointment reminders (1h) | Hourly check | Send reminders for appointments in next 1 hour |
| Lifecycle stage updates | Daily 02:00 | Recalculate contact lifecycle stages |
| Automated task generation | Daily 03:00 | Create follow-up tasks based on upcoming bookings |
| Webhook processing | On event | Defer webhook payload processing from HTTP handler |

Any operation expected to exceed 22 seconds must use `@anvil.server.background_task`. Background tasks have no timeout limit.

**See:** `real-time-and-background-tasks` ADR (Real-Time and Background Task Architecture)

**See:** `pdf-invoice-generation` ADR (PDF Invoice Generation) for invoice PDF approach.

**See:** `data-access-patterns` ADR (Data Access Patterns) for denormalization and pre-computed aggregation patterns.

---

## 5. Timezone Architecture

Each client instance configures its own timezone at onboarding, stored as an IANA timezone string in `business_profile.timezone`. All datetimes are stored in the database as UTC. Conversion to the client's local timezone occurs at display time in server functions.

**See:** `5-timezone-utc-storage-display-conversion.md`

---

## 6. Domain and Provisioning

Mybizz owns the domain `mybizz.live`. Client subdomains are issued from this domain. Cloudflare is used for all DNS, CDN, and security.

**Scenario A:** Client has no domain → receives `businessname.mybizz.live`
**Scenario B:** Client brings their own domain → Mybizz points nameservers to Cloudflare

Business email is not provisioned by Mybizz. Clients use whatever email provider they choose.

---

## 7. Offboarding

On offboarding, the client receives a full export of all Data Tables, their `encryption_key`, and Vault contents in decrypted form. Mybizz retains nothing of the client's data after offboarding.

---

*End of file*
