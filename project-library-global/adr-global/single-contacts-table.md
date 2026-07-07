# 10 — Single Contacts Table Replaces Dual contacts/customers
**Consolidated Contact Model for CRM**

Date: 2026-05-03
Status: Accepted
Source: YC Office Hours session — schema review and design consolidation

---

## Context

The live schema originally contained two overlapping tables: `contacts` (CRM-focused with lifecycle stages, tags, source tracking) and `customers` (booking/invoice-focused with lifetime value, total bookings). This created referential chaos:

- `bookings.customer_id` → `customers` but also `bookings.contact_id` → `contacts`
- `invoice.customer_id` → `customers`
- `time_entries.customer_id` → `customers`
- `client_notes.customer_id` → `customers`
- `customers.contact_id` → `contacts` (partial migration link)

Every query needed to know which table to join. The dual-table model was a half-migrated schema that added complexity without benefit.

This decision builds on **`4-lead-capture-simultaneous-creation.md`** (Lead Capture: Simultaneous Creation), which established that `leads` and `contacts` are created simultaneously at capture time, with `leads.converted_to_contact_id` set immediately. The `contacts` table is the authoritative person record for all CRM operations.

---

## Decision

**A single `contacts` table serves as the unified person record for the entire platform.**

All customers are contacts but not all contacts are customers. The `contacts` table absorbs all fields and relationships previously split between `contacts` and `customers`.

### Migration actions completed:
- `bookings.customer_id` removed; `bookings.contact_id` retained as sole person reference
- `invoice.customer_id` removed; `invoice.contact_id` added (note: column name in anvil.yaml is `contact-id` with hyphen — requires correction)
- `time_entries.customer_id` removed; `time_entries.contact_id` added
- `client_notes.customer_id` removed; `client_notes.contact_id` added
- `customers` table dropped from schema
- `zoho_crm_id` column removed from `contacts` (`1-brevo-replaces-zoho-email.md` compliance)

### `contacts` table fields (consolidated):
- Identity: `contact_id`, `first_name`, `last_name`, `email`, `phone`
- CRM: `status`, `source`, `lifecycle_stage`, `tags`, `internal_notes`, `preferences`
- Financial: `total_spent`, `total_transactions`, `average_order_value`
- Tracking: `date_added`, `last_contact_date`, `created_at`, `updated_at`


---

## Consequences

### Benefits:
- Single source of truth for all person records
- Simplified queries — no need to join two tables
- Clean CRM integration — Brevo sync targets one table
- Consistent with `4-lead-capture-simultaneous-creation.md` lead capture pattern

### Files affected:
- `anvil.yaml` — `customers` table removed, FK references updated
- `spec_database.md` / `spec_database_schema.md` — dual-table references removed
- `spec_crm.md` — contact model simplified
- All server modules — FK references updated from `customer_id` to `contact_id`

---

*End of `single-contacts-table` ADR*
