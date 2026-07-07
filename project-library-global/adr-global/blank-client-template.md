# `blank-client-template` ADR — blank_client_template as Provisioning Clone Source

**Status:** Confirmed — revised following Test B (2026-06-13)  
**Date:** 2026-06-13  
**Last updated:** 2026-06-13  
**Authority:** Derived from `client-instance-architecture` ADR, `dependency-update-model` ADR, platform-overview.md, Test B result

---

## Context

Every new client instance requires 36 pre-defined Data Tables with the correct schema before master_template's server modules can function. Anvil provides no programmatic API for creating Data Tables — they must be defined in the IDE. A fresh blank Anvil app has zero tables.

Three options were evaluated for how a new client instance acquires its 36 tables:

1. **Clone master_template** — produces a full code copy; client instance then has its own server modules which take precedence over the dependency, permanently breaking update propagation (see `client-instance-architecture` ADR)
2. **Manual table creation per client** — 36 manual steps per new client; not workable at scale
3. **A dedicated blank provisioning app** — pre-configured with 36 tables, master_template as dependency, and no server modules of its own

Option 3 is the correct solution.

---

## Decision

A fifth app, `blank_client_template`, is maintained on the mybizz Anvil account. It serves exclusively as the clone source for new client instances.

### Contents of blank_client_template

| Component | Present | Notes |
|---|---|---|
| All 36 Data Tables | ✅ | Correct schema; no data |
| Startup module | ✅ | One minimal module; imports and opens first form from master_template |
| master_template as dependency | ✅ | Set to stable branch; present so it is inherited by every clone |
| Server modules | ❌ | Must never be added |
| UI forms | ❌ | Forms live in master_template; not in blank_client_template |
| Anvil Secrets | ❌ | Set individually per client instance at provisioning |
| Custom domain | ❌ | Configured per client instance at provisioning |

**Why no forms:** Test B confirmed that a client instance with only a startup module can successfully load and display forms served from master_template via the dependency. Forms therefore live in master_template — not in blank_client_template or in client instances. This means UI and form updates propagate automatically to all client instances when master_template is updated.

**The startup module in blank_client_template (inherited by every client instance clone):**
```python
from master_template.MainForm import MainForm
from anvil import open_form
open_form(MainForm())
```

### Cloning Process

When a new client is onboarded:

1. Clone `blank_client_template` in the Anvil IDE
2. Rename the clone to the client identifier
3. The clone immediately has all 36 tables, the startup module, and master_template wired as dependency
4. Proceed with remaining activation steps (see `client-activation-runbook.md`)

---

## Schema Migration Responsibility

`blank_client_template` is a maintained asset. When the Data Table schema changes — new table added, column added or modified, column removed — the change must be applied in two places:

| Target | Purpose |
|---|---|
| `blank_client_template` | Ensures future client instances get the correct schema |
| Every existing client instance | Ensures running clients are not broken by the schema change |

There is no automated migration tool in Anvil. Schema migrations to existing client instances are manual IDE operations. This must be factored into the release process for any schema-affecting change.

A schema change must never be merged to `stable` (master_template) without a documented migration plan for existing client instances.

---

## Alternatives Rejected

| Alternative | Reason rejected |
|---|---|
| Clone master_template | Copies server module code; breaks dependency update model (`client-instance-architecture` ADR) |
| Manual table creation | 36 steps per client; error-prone; not scalable |
| Programmatic table creation | Not supported in Anvil — tables cannot be created via code |
| Clone an existing client instance | Risks copying live client data; requires data wipe; error-prone |

---

## Consequences

- ✅ New client provisioning produces a correctly structured instance in minutes
- ✅ No server modules or forms in the clone — full update propagation (server, UI, forms) intact from day one
- ✅ Consistent table schema across all new client instances
- ✅ blank_client_template requires only schema maintenance — no form or UI maintenance needed here
- ⚠️ blank_client_template must be kept in sync with schema changes — it is a maintained asset for Data Tables only
- ⚠️ Schema migrations to existing client instances are manual — this constrains how freely the schema can change post-launch
- ⚠️ The five-app structure requires clear documentation to avoid confusion about each app's role

---

## The Five-App Structure

| App | Role |
|---|---|
| `mb-3-cs` | Development workspace — all code, all forms, all server modules, 36 tables |
| `master_template` | Published dependency — all server modules + all forms + all UI |
| `blank_client_template` | Provisioning clone source — 36 tables + startup module only |
| `[client instances]` | One per client — 36 tables + startup module + master_template dependency (active) |
| `Mybizz_management` | Platform operator tooling (deferred) |

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/client-instance-architecture.md`` | Defines why server modules must not exist in client instances |
| ``adr/adr-global/dependency-update-model.md`` | Update propagation model that blank_client_template preserves |
| `implementation/client-activation-runbook.md` | Uses blank_client_template as step 1 of provisioning |

---

*End of `blank-client-template` ADR*
