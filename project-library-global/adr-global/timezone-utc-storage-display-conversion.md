# 05 — Client Timezone: IANA String, UTC Storage, Display-Time Conversion
Date: 2026-03-17
Status: Accepted
Source: cs-architectural-specification-D.md §13

---

## Context

Mybizz serves clients globally. Datetimes (bookings, reminders, invoices, activity
logs) must be meaningful to each client in their own local time. A decision was needed
on how to store and display datetimes across client instances with different timezones.

---

## Decision

**Each client instance configures its own timezone at onboarding**, stored as an IANA
timezone string (e.g. `'Africa/Johannesburg'`, `'Europe/London'`) in the
`business_profile` table (`timezone` field).

**All datetimes are stored in the database as UTC.**

**Conversion to the client's local timezone occurs at display time** in server
functions that return datetime data for UI presentation. Client code never handles
timezone conversion.

**The Mybizz platform itself** (devlog timestamps, build artefacts, reference
documents) uses UTC+2 (Africa/Johannesburg) — the developer's timezone. This applies
to build artefacts only, not to client data handling. Do not conflate the two.

### Practical implication for server functions

Any server function returning a datetime for display must convert from UTC to the
client's configured timezone before returning. The `timezone` value is read from
`business_profile` at the start of the function. A helper function for this conversion
should be centralised — not duplicated per function.

---

## Consequences

### Rules files affected

| File | Required content |
|---|---|
| `spec_architecture.md` | Timezone architecture, UTC storage mandate, display-time conversion pattern |
| `spec_database.md` | `business_profile.timezone` field (IANA string) |
| `ref_anvil_coding.md` | Datetime handling — UTC storage rule, display-time conversion |

---

*End of `timezone-utc-storage-display-conversion` ADR*
