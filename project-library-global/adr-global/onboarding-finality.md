# 26 — Onboarding Finality
Date: 2026-05-31
Status: Accepted
Source: Plan-tune cross-reference audit — formalising onboarding design decision
Updated: 2026-06-01 — Rewritten to allow onboarding return, credential changes, and Mybizz_management data rights

---

## Context

The Mybizz CS onboarding process was originally designed as a one-time, non-replayable flow. The revised understanding is that Owners have the right to change any credentials they choose, and the onboarding page remains accessible for this purpose.

The question arose of what happens after onboarding is completed, whether the Owner can return to the onboarding page, and how credential changes are tracked in Mybizz_management.

---

## Decision

**Onboarding is resumable and revisitable.** The Owner may return to the onboarding page at any time to review or change any credential they choose.

**Credential changes are append-only in Mybizz_management.** When the Owner updates client details on the onboarding page, the new details are appended to the client's record in Mybizz_management. Previous values are never overwritten.

**Mybizz_management maintains three data tables for client records:**

1. **Client Details Table (Owner-Modifiable):** Contains client details that the Owner has the right to change at any time. The Owner may remove entries from this table. Updates append new values; old values are retained in the amendment log.

2. **Retained Records Table (Legal/Financial):** Contains client details retained by Mybizz for legal or financial purposes. The Owner has no right to remove these. These are held separately and are immutable from the Owner's perspective.

3. **Amendment Log (Append-Only):** A full history of all credential and configuration changes, never overwritten, always holding the latest values plus the full history. This is outside the client instance scope but is a system-level behaviour.

### Architectural Rules

1. **Onboarding is revisitable:**
   - The onboarding page remains accessible after completion
   - The Owner may change any credential at any time
   - No field is permanently locked except where explicitly stated (e.g., system currency after first transaction)

2. **Credential changes are append-only:**
   - When the Owner updates client details, the new values are appended to the client's record in Mybizz_management
   - Previous values are never overwritten
   - The full history is preserved in the amendment log

3. **Owner data rights:**
   - The Owner may remove their details from the Client Details Table in Mybizz_management
   - The Owner may NOT remove entries from the Retained Records Table (legal/financial retention)
   - The Owner may export their data at any time (per GDPR/POPIA compliance)

4. **Mybizz_management is outside client instance scope:**
   - Mybizz_management is a separate Anvil application for platform operations
   - It maintains its own data tables independent of client instances
   - The client instance does not directly access Mybizz_management tables
   - Changes made in the client instance are synced to Mybizz_management via server functions

5. **System Currency exception:**
   - System Currency is immutable after the first transaction (per `16-system-currency-selection-and-immutability.md`)
   - This is the only field that cannot be changed after a specific event
   - All other credentials remain changeable at any time

---

## Consequences

### Positive
- Owners retain full control over their credentials at all times
- Clear data ownership and rights alignment with GDPR/POPIA
- Append-only logging provides complete audit trail
- Separation of Owner-removable data and legally-retained data is explicit
- Onboarding remains accessible for credential review and updates

### Negative
- Mybizz_management requires three separate data tables for client records
- Append-only logging increases storage requirements over time
- System Currency immutability creates a permanent constraint

### Neutral
- Onboarding is a living page, not a one-time event
- Settings form and onboarding form are complementary surfaces
- Mybizz_management data architecture is outside client instance scope

---

## Related Documents

- `17-onboarding-vs-settings-boundary.md` — Settings as the single configuration surface
- `16-system-currency-selection-and-immutability.md` — Currency immutability after first transaction
- `24-payment-gateway-mutability.md` — Gateway change rules
- `23-onboarding-resumability.md` — Progress preservation during onboarding
- `implementation/onboarding-implementation-plan.md` — Settings-based onboarding implementation
- `docs/platform-overview.md` — Mybizz_management data architecture intention

---

*End of `onboarding-finality` ADR (deleted)*
