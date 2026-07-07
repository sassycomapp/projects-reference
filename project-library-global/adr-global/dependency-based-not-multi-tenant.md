# `dependency-based-not-multi-tenant` ADR — Mybizz CS is a Dependency-Based Architecture, Not a Multi-Tenant Architecture

**Status:** Confirmed — Sealed  
**Date:** 2026-06-13  
**Authority:** Derived from `client-instance-architecture` ADR, `dependency-update-model` ADR, `blank-client-template` ADR, live testing 2026-06-13  
**Audience:** All developers and AI coding assistants working on this codebase

---

## ⚠️ READ THIS FIRST

This ADR exists because AI coding assistants (including OpenCode) repeatedly default to multi-tenancy patterns when working on this codebase. This is **architecturally wrong** and must not happen.

**Mybizz CS is not a multi-tenant application. It is a dependency-based SaaS platform.**

Every pattern, every query, every data access function, and every server module must be written to reflect this. If you find yourself writing `client_id`, `tenant_id`, `organisation_id`, or any variant thereof as a filter on a data query, **stop**. You are implementing the wrong architecture.

---

## 1. What Multi-Tenancy Is (and Why It Does Not Apply Here)

Multi-tenancy is an architecture where a single application instance serves multiple clients (tenants), with all clients' data stored in a shared database. Clients are separated by a discriminator value — typically a `tenant_id`, `client_id`, or `organisation_id` column — on every table. Every query must filter by this discriminator to ensure one tenant cannot see another's data.

**Characteristics of a multi-tenant architecture:**
- One application instance
- One database
- All clients' data in the same tables
- Rows separated by `tenant_id` or equivalent
- Every query includes `WHERE tenant_id = X`
- Tenant context injected into sessions or middleware
- Risk of data leakage if a filter is accidentally omitted

**Mybizz CS has none of these characteristics. None.**

---

## 2. What Mybizz CS Actually Is

Mybizz CS is a **dependency-based SaaS platform**. It has the following structure:

### The Five Apps

| App | Role | Contains |
|---|---|---|
| `mb-3-cs` | Development workspace | All code, all forms, all server modules, 36 tables (dev only) |
| `master_template` | Published dependency | All server modules + all UI forms |
| `blank_client_template` | Provisioning clone source | 36 Data Tables + one startup module |
| `[client-name]-cs` | Client instances (one per client) | 36 Data Tables (live data) + one startup module + master_template as dependency |
| `Mybizz_management` | Platform operator tooling | Deferred |

### The Fundamental Principle

**Each client is a completely separate Anvil application with its own isolated database.**

- Client A's data is in Client A's app
- Client B's data is in Client B's app
- These apps are separate Anvil apps on the same account
- They share no database, no tables, no rows
- Neither can access the other's data

### How Code Reaches Clients

`master_template` is published as an Anvil dependency. Every client instance depends on it. When a client instance calls `anvil.server.call('get_invoices')`, Anvil executes that function in `master_template`. Inside that function, `app_tables.invoices` refers to **that specific client instance's own `invoices` table** — not a shared table, not filtered by client_id — just that client's table, containing only that client's data.

This is structural behaviour confirmed by live test. See Section 6.

---

## 3. The Definitive Contrast

| Concern | Multi-Tenancy | Mybizz CS (Dependency-Based) |
|---|---|---|
| Number of app instances | One | One per client |
| Number of databases | One (shared) | One per client (isolated) |
| Data separation mechanism | `tenant_id` column on every table | Separate Anvil apps — structural isolation |
| Query filter required | Yes — `WHERE tenant_id = X` on every query | No — `app_tables` always resolves to the current client's tables |
| Risk of cross-client data access | Yes — if filter is omitted | No — architecturally impossible |
| Client context in session | Required | Not required — there is no "other client" to distinguish from |
| Shared codebase | Yes | Yes — via `master_template` dependency |
| Shared database | Yes | No |

---

## 4. Prohibited Patterns

The following patterns are **explicitly forbidden** in this codebase. They indicate a multi-tenancy implementation and must never appear.

### ❌ Tenant discriminator columns

```python
# FORBIDDEN — never add these columns to any table
client_id = ...
tenant_id = ...
organisation_id = ...
account_id = ...
```

Data Tables in Mybizz CS must never include a column whose purpose is to identify which client a row belongs to. Every row in every table already belongs to one client — the client whose app is running.

### ❌ Tenant-filtered queries

```python
# FORBIDDEN — never filter by client identity
def get_invoices(client_id):
    return app_tables.invoices.search(client_id=client_id)

def get_bookings(tenant_id):
    return app_tables.bookings.search(tenant_id=tenant_id)
```

### ❌ Tenant context in sessions or modules

```python
# FORBIDDEN — there is no tenant context to store or retrieve
def get_current_client():
    return anvil.server.session.get('client_id')

def set_tenant_context(client_id):
    anvil.server.session['tenant_id'] = client_id
```

### ❌ Cross-client data access

```python
# FORBIDDEN — one client instance cannot access another's data
# and there is no mechanism to do so — do not attempt to build one
def get_all_clients_invoices():
    ...
```

### ❌ Tenant-aware middleware or decorators

```python
# FORBIDDEN
@require_tenant_context
def get_invoices():
    ...
```

---

## 5. Required Patterns

The following patterns are correct and must be used consistently.

### ✅ Direct, unfiltered queries

```python
# CORRECT — app_tables always resolves to the current client's tables
def get_invoices():
    return app_tables.invoices.search()

def get_bookings():
    return app_tables.bookings.search(
        tables.order_by('start_time', ascending=True)
    )

def get_active_contacts():
    return app_tables.contacts.search(is_active=True)
```

No `client_id` filter. No tenant context. The function retrieves this client's data — because there is no other client's data in this database.

### ✅ Row operations without tenant qualification

```python
# CORRECT
def create_invoice(data):
    return app_tables.invoices.add_row(**data)

def update_booking(booking_row, updates):
    booking_row.update(**updates)

def delete_contact(contact_row):
    contact_row.delete()
```

### ✅ User-based access control where needed

The only legitimate identity-based filtering in this codebase is **role-based access control** within a single client instance. This is not tenancy — it is user permissions within one client's own data.

```python
# CORRECT — filtering by user role, not by tenant
def get_invoices_for_current_user():
    user = anvil.users.get_user()
    if user['role'] == 'Owner':
        return app_tables.invoices.search()
    elif user['role'] == 'Staff':
        return app_tables.invoices.search(assigned_to=user)
```

---

## 6. Verified Architectural Evidence

### Test A — app_tables Resolution (2026-06-13) ✅

**Setup:**
- `datatable-test-host-app`: dependency app with a server module calling `app_tables.table_1.add_row(name=name, some_number=number)`
- `datatable-test-client-app`: client instance app with its own `table_1`, depending on host app

**Result:** Row appeared in `datatable-test-client-app`'s `table_1` only. `datatable-test-host-app`'s `table_1` was unaffected.

**Conclusion:** `app_tables` in a dependency server module resolves to the **calling client instance's own tables** at runtime. This is structural, automatic, and irrevocable. It does not require code to enforce it. It cannot accidentally fail to enforce it.

### Test B — Forms from Dependency (2026-06-13) ✅

**Setup:**
- `test-dependency-forms`: dependency app with one form (`MainForm`)
- `test-client-no-forms`: client instance with only a startup module, no forms of its own

**Result:** `MainForm` loaded and displayed correctly, served from the dependency.

**Conclusion:** Client instances require no code of their own. The startup module and Data Tables are the complete contents of a client instance. Everything else — server logic and UI — is served from `master_template`.

---

## 7. Why This Architecture Was Chosen Over Multi-Tenancy

| Consideration | Multi-Tenancy | Dependency-Based |
|---|---|---|
| Data isolation | Enforced in application code — can fail | Structural — cannot fail |
| Security breach blast radius | All clients exposed if breach occurs | One client affected |
| Regulatory compliance | Complex — all clients' data co-mingled | Simple — each client's data is physically separate |
| Update mechanism | Immediate for all tenants (single instance) | Immediate for all clients (via dependency branch tracking) |
| Client onboarding | Add a row to a tenants table | Clone blank_client_template |
| Schema migration | Single migration | Applied per client instance — more effort, but isolated |
| Suitability for consulting/services clients | Poor — clients expect data sovereignty | Strong — data sovereignty is structural |

The consulting and services vertical has strong expectations around data privacy and sovereignty. A client whose financial data, client records, and booking history live in a physically separate database — not co-mingled with other businesses' data — is a significantly easier conversation than explaining row-level security in a shared database.

---

## 8. What to Do If You Are Unsure

If you are writing a server module and you are unsure whether to include a client or tenant identifier, ask yourself:

> *"Could there ever be data from a different client in this database?"*

The answer is always **no**. There is only one client per database. Write the query without any tenant filter.

If you find an existing function that includes a `client_id`, `tenant_id`, or equivalent parameter used for data filtering, it is a bug. Remove it and rewrite the query without the filter.

---

## 9. Consequences

- ✅ Every server module in `master_template` is written for a single-client context — no tenant logic
- ✅ No Data Table in any client instance contains a `client_id` or `tenant_id` column
- ✅ Data isolation is guaranteed by architecture, not by developer discipline
- ✅ Code is simpler — no tenant middleware, no context injection, no discriminator filters
- ✅ Security is stronger — cross-client data access is structurally impossible
- ⚠️ AI coding assistants (including OpenCode) must be given this ADR as context before any code generation task — defaulting to multi-tenancy patterns will produce incorrect code

---

## 10. Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/client-instance-architecture.md`` | Defines client instance structure and app_tables test |
| ``adr/adr-global/dependency-update-model.md`` | How master_template updates reach all client instances |
| ``adr/adr-global/blank-client-template.md`` | Provisioning — how client instances are created |
| `docs/platform-establishment-report-2026-06-13.md` | Full architectural narrative and rationale |
| `HANDOVER-opencode-2026-06-13.md` | Technical handover for OpenCode |

---

*End of `dependency-based-not-multi-tenant` ADR*
