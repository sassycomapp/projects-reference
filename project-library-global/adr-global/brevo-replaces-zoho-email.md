# 01 — Brevo Replaces All Zoho Products
Date: 2026-03-21
Status: Accepted
Source: Devlog #34 — plan-email-architecture-brevo-and-session-cleanup

---

## Context

The original architecture used three separate Zoho products across two concerns:

**Email:**
- **Zoho Mail SMTP** — transactional email (booking confirmations, reminders, invoices)
- **Zoho Campaigns** — marketing email (sequences, broadcasts, lead capture)

**CRM:**
- **Zoho CRM** — contact database, interaction history, deal pipeline, contact sync

During integration research, the email architecture was reviewed and a fundamental
problem was discovered with Zoho Mail. This triggered a broader review of the entire
Zoho dependency, which concluded that all three Zoho products should be replaced by
a single unified platform.

### Why Zoho Mail SMTP was broken

Zoho Mail's free plan does not include SMTP access. Every new Mybizz client starts on
the free tier. Transactional email — booking confirmations, reminders, invoices — is
therefore non-functional for every new client from day one. This is not a degraded
experience or a limitation to work around. It is completely broken. The core value
proposition of the platform depends on transactional email functioning from day one,
and Zoho Mail cannot deliver this without a paid plan — which clients are not expected
to purchase at onboarding.

### Why Zoho Campaigns was also replaced

Zoho Campaigns is a separate product from Zoho Mail with separate credentials, a
separate API, and a separate configuration path. Maintaining two distinct Zoho email
integrations — each with its own Vault credentials, settings UI, and error surface —
for what is fundamentally one concern (client email) adds unnecessary complexity to
onboarding, maintenance, and the credential management architecture.

### Why Zoho CRM was also replaced

Once Zoho email was removed, retaining Zoho CRM as the contact management layer
created an illogical split: Brevo would own email sending and marketing, while Zoho
CRM would separately own the contact database that those emails depend on. This is
architecturally fragmented and operationally counterproductive.

More importantly, Brevo is not purely an email sending service. Brevo is a unified
customer engagement platform that includes a full built-in CRM — contact storage,
interaction history, deal pipeline, and audience segmentation — tightly integrated
with its email and automation engine. Separating email from CRM when a single platform
handles both natively is unnecessary complexity with no benefit.

Zoho CRM's free tier also imposes constraints that become limiting as clients grow:
5,000 total records across all modules, 3 users per organisation, 10MB storage, and
5,000 API credits per 24-hour rolling window. Brevo's free tier supports 100,000
contacts — twenty times more — with no per-user restriction.

The correct architecture is one platform for all client-facing communication and
contact management. That platform is Brevo.

---

## Decision

Replace all three Zoho products — Zoho Mail SMTP, Zoho Campaigns, and Zoho CRM —
with **Brevo** as the single unified platform for transactional email, marketing
email, and CRM.

**Zoho is entirely removed from the Mybizz application architecture. No Zoho product
remains. No Zoho credentials remain. No Zoho integration code remains.**

This is a permanent decision. It is not subject to partial retention or phased
coexistence. Any future reference to Zoho in rules files, spec files, or code is
an error requiring correction.

---

## Why Brevo

Brevo was selected over the following alternatives researched during this session:
Google Workspace, Microsoft Outlook, SendGrid, Mailgun, Postmark, and Zoho (all
products). The evaluation criteria were: functional free tier, client owns their
own account, genuine start-free-upgrade-when-ready model, simplest integration to
maintain, and best combined coverage of transactional email, marketing, and CRM.

Brevo meets all criteria:

**Free tier is genuinely functional from day one:**
- 300 emails/day (9,000/month) — SMTP included, no credit card required
- 100,000 contacts stored
- Marketing automation workflows included
- Full API access included
- Built-in CRM included (contacts, interaction history, deals, pipeline)
- Open and click tracking included
- Unsubscribe management included

**Single integration replaces three:**
One platform, one set of credentials, one settings UI, one onboarding step. The
client creates one Brevo account and connects it to Mybizz once. There is no concept
of "email credentials" and "CRM credentials" as separate concerns.

**Client model is clean and consistent with the Mybizz architecture:**
Each client creates and manages their own Brevo account. Mybizz has no involvement in
provisioning or administering client accounts. The client owns their data. When a
client leaves Mybizz, their Brevo account and contacts are entirely theirs.

**Upgrade path is transparent and requires no Mybizz code changes:**
The SMTP credentials, API key structure, and integration code are identical across
all Brevo tiers. When a client upgrades from Free to Starter ($9/month) to remove
the daily sending limit and Brevo branding, nothing changes in the Mybizz codebase.
Tier upgrades are entirely between the client and Brevo.

**API and SMTP structure are stable and well-documented:**
Brevo's API and SMTP relay are production-grade and have been stable across versions.
The integration pattern is standard and well-supported.

---

## What Brevo Provides

| Concern | Previous solution | Brevo replacement |
|---|---|---|
| Transactional email (SMTP) | Zoho Mail SMTP | Brevo SMTP relay |
| Marketing campaigns and automation | Zoho Campaigns | Brevo Campaigns + Automation |
| Contact database | Zoho CRM | Brevo CRM (built-in) |
| Interaction history | Zoho CRM | Brevo CRM (built-in) |
| Deal pipeline | Zoho CRM | Brevo CRM (built-in) |
| Contact sync from Mybizz | Zoho CRM API | Brevo API |
| System email (login, password reset) | Anvil built-in | Anvil built-in — unchanged |

System emails (login links, password reset) continue to use Anvil's built-in email
service. These are platform-level emails, not client-controlled, and are outside the
scope of Brevo.

---

## Vault Credentials

Two secrets per client instance:

| Vault key | Purpose |
|---|---|
| `brevo_smtp_key` | SMTP authentication for transactional email |
| `brevo_api_key` | REST API for CRM sync, campaigns, automation, and webhooks |

**SMTP constants** (hardcoded in `transactional_email_service.py` — identical for all
clients, not stored in Vault or database):
- Host: `smtp-relay.brevo.com`
- Port: `587` (STARTTLS)

All Zoho Vault credentials (`zoho_crm_client_id`, `zoho_crm_client_secret`,
`zoho_crm_refresh_token`, and any Zoho email credentials) are removed entirely.

---

## Business Email Addresses

Business email addresses (admin@clientdomain.com, hello@clientdomain.com, etc.) are
not provisioned by Mybizz. Clients use whatever email provider they choose for their
mailboxes — Google Workspace, Microsoft 365, or any other. Mybizz is not in the
business of provisioning or managing client mailboxes.

Brevo sends email on behalf of the client's domain via SMTP relay or API. The client's
from address and display name are configured in Brevo sender settings. If the client
configures DKIM and SPF records on their domain (via Cloudflare or their DNS provider),
emails will be sent authenticated from their own domain. If not, Brevo sends from a
@brevosend.com relay address. Either approach works; domain authentication is the
recommended professional setup and Mybizz can guide clients through it during premium
onboarding.

---

## Consequences

All Zoho references must be removed from every rules file, spec file, devref file,
schema file, and codebase reference. This includes Zoho Mail, Zoho Campaigns, and
Zoho CRM without exception.

### Rules and spec files requiring correction

| File | Required change |
|---|---|
| `spec_integrations.md` | §2 replace Zoho Mail SMTP with Brevo SMTP. §3 replace Zoho Campaigns with Brevo Campaigns API. §4 remove Zoho CRM integration entirely and replace with Brevo CRM API pattern. §6 secrets table: remove all Zoho entries, add `brevo_smtp_key` and `brevo_api_key`. — Todo #125 |
| `spec_domain_email_offboarding.md` | Full rewrite — built around Zoho email provisioning model. Remove business email provisioning entirely. — Todo #126 |
| `spec_database_schema.md` | Table 36 (`email_config`): replace Zoho SMTP column structure with Brevo five-column structure. Table 49 (`email_campaigns`): remove `zoho_campaign_id`. Remove any `zoho_crm_*` column references. — Todo #127 |
| `spec_architecture.md` | Update `server_emails/` and `server_marketing/` labels. Remove Zoho CRM server module references. — Todo #128 |
| `platform_overview.md` | §3.7 and tech stack table — update entire communication and CRM stack to Brevo. — Todo #129 |
| `spec_crm_campaigns.md` | Replace Zoho Campaigns implementation with Brevo. Remove Zoho CRM references. — Todo #130 |
| `spec_crm.md` | Remove all Zoho CRM references. Replace with Brevo CRM API pattern. |

### Devref files

- `@devref/brevo-email-reference.md` — to be updated to cover full Brevo scope
  including CRM, not just email. Full implementation detail, tier comparison, sender
  identity options, Vault credential structure, CRM API pattern, DKIM guidance,
  graceful degradation, and onboarding flow.
- `@devref/zoho-crm-reference.md` — **retired and deleted**. This file was created
  during the same session as this decision and was written before the full scope of
  the Zoho replacement was settled. It is incorrect and must not be used.

### What must not happen

Any future document, rules file, spec, or code that references Zoho CRM, Zoho Mail,
or Zoho Campaigns is in error. The phrase "Zoho CRM is unaffected" or any equivalent
carve-out for any Zoho product is incorrect and contradicts this decision. If such
language is encountered in any file, it must be treated as a documentation error and
corrected immediately.

---

*End of `brevo-replaces-zoho-email` ADR*
