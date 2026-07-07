# `client-instance-architecture` ADR — Client Instance Architecture

**Status:** Confirmed — architecturally tested (Tests A and B)  
**Date:** 2026-06-13  
**Last updated:** 2026-06-13 — revised following Test B (startup form from dependency)  
**Authority:** Derived from platform-overview.md, `payment-security-boundary-vault` ADR, and live testing conducted 2026-06-13

---

## Context

The platform requires complete data isolation between clients, automatic propagation of server logic updates, and a clear separation between application code and client data. The relationship between master_template (the published dependency) and each client instance needed to be defined and tested before any server modules were written.

The critical untested assumption was whether `app_tables` called from within a master_template server module would resolve to the client instance's own Data Tables or to master_template's tables.

---

## Test Conducted — 2026-06-13

Two Anvil apps were created:

- `datatable-test-host-app` — acting as the dependency (master_template equivalent), containing a server module that called `app_tables.table_1.add_row()`
- `datatable-test-client-app` — acting as the client instance, depending on the host app, containing its own `table_1`

**Finding:** The row was written exclusively to the client app's `table_1`. The host app's `table_1` received nothing.

**Conclusion:** `app_tables` in a dependency server module resolves to the calling client instance's Data Tables at runtime. This is structural and automatic — no application-level enforcement is required.

---

## Test B Conducted — 2026-06-13

A second test was conducted to determine whether a client instance with no forms of its own could successfully load and display a form defined in its dependency.

- `test-dependency-forms`: An Anvil app containing one form (`MainForm`) with a visible label. No server modules, no Data Tables.
- `test-client-no-forms`: A bare Anvil app with one startup module and no forms. `test-dependency-forms` set as dependency.

**Startup module code in test-client-no-forms:**
```python
from PACKAGE_NAME.MainForm import MainForm
from anvil import open_form
open_form(MainForm())
```

**Finding:** The app ran without error. MainForm loaded and displayed correctly, served entirely from the dependency.

**Conclusion:** A client instance requires no forms of its own. All UI and form code can reside in master_template and be served at runtime via the dependency. This means UI and form updates propagate automatically to all client instances — identical to server module update propagation.

---

## Decision

Client instances are structured as follows:

| Component | Lives in | Notes |
|---|---|---|
| Server modules | master_template only | All business logic, integrations, background tasks |
| UI forms and form code | master_template only | Served to client instances via dependency at runtime |
| Startup module | Client instance | One minimal module; imports and opens first form from master_template |
| Data Tables (36) | Client instance | All client data; never leaves the instance |
| master_template dependency | Client instance | Set to stable branch at provisioning; active |

**Client instances contain no server modules and no forms of their own.** The startup module is the only code a client instance contains. It imports the first form from master_template and opens it. All subsequent navigation, UI, form code, and server logic is served from master_template via the dependency.

```python
# The complete client-side code of every client instance
from master_template.MainForm import MainForm
from anvil import open_form
open_form(MainForm())
```

---

## Rationale

- `app_tables` resolution to the client instance is confirmed structural behaviour in Anvil (Test A)
- Forms defined in a dependency are served to and rendered in the dependent app at runtime — confirmed by Test B
- Keeping all code (server modules and forms) exclusively in master_template ensures all updates propagate automatically to every client instance simultaneously
- Data isolation between clients is architecturally guaranteed — one client instance cannot access another's tables
- Security boundary is structural, not enforced in application code
- The startup module is the minimum viable client-side code — one import, one open_form call

---

## Consequences

- ✅ Server logic updates propagate to all client instances automatically on master_template release
- ✅ UI and form code updates propagate to all client instances automatically on master_template release
- ✅ Data is fully isolated per client — structural guarantee
- ✅ No cross-client data access is architecturally possible
- ✅ Client instances require no maintenance — only Data Tables and one startup module
- ⚠️ Client instances must never have server modules or forms added — provisioning documentation must make this explicit
- ⚠️ Schema changes (adding/modifying Data Tables) are the only update that does not propagate automatically — requires manual application to blank_client_template and all existing client instances

---

## Related Documents

| Document | Relationship |
|---|---|
| `platform-overview.md` | Section 7.4: Dependency app_tables Resolution — Verified |
| ``adr/adr-global/dependency-update-model.md`` | How updates flow from mb-3-cs to client instances |
| ``adr/adr-global/blank-client-template.md`` | The provisioning source for new client instances |
| `implementation/client-activation-runbook.md` | Step-by-step provisioning process |

---

*End of `client-instance-architecture` ADR*
