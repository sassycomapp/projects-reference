# Mybizz — Five-App Architecture Model (Specification)

**Scope:** Mybizz-wide. Applies to every Mybizz product, not just this app. The specific app
names/slugs below are placeholders — a given app's actual instance names live in that app's own
`docs/platform-architecture.md`.

---

## The Five Apps

| App | Role | Contains | Updated via |
|---|---|---|---|
| App 1 — `{dev-workspace}` | Development workspace. All feature work happens here. Never a dependency for any client instance. | All Data Tables (dev/testing), all UI forms, all server modules, all application code | Daily development on GitHub `develop` branch |
| App 2 — `master_template` | Published dependency. Sole source of all server-side logic and UI forms for every client instance. | All server modules + all forms + all UI | Merge `develop` → `stable` in GitHub; tracks `stable` automatically |
| App 3 — `blank_client_template` | Provisioning clone source. The sole clone source for creating new client instances. | Full Data Table schema (correct, no data), one startup module, `master_template` as dependency. Zero server modules, zero forms. | Only when schema changes occur |
| App 4 — `[client-name]-{vertical}` | One per paying client — a dedicated, isolated app instance. | Data Tables + one startup module + active `master_template` dependency. Nothing else. | Automatic — server logic, UI, and forms all propagate via the dependency. Schema changes are manual only. |
| App 5 — `{Product}_management` | Platform operator's internal tooling — provisioning, health monitoring, billing automation, update distribution. | Client registry, monitoring dashboard, billing status, update distribution controls | Deferred — built post-launch |

## Why This Shape

- **Data isolation is structural.** Each client instance is a separate Anvil app with a separate
  database. `app_tables` always resolves to the current client's data. Cross-client access is
  architecturally impossible.
- **`master_template` is stateless.** It contains zero client data. All state lives in each
  client's own Data Tables.
- **Clients never see source code.** `master_template` is consumed as a compiled dependency.
- **`blank_client_template` exists because Anvil cannot create tables programmatically** —
  cloning it gives each new client the required schema without copying any server code or forms,
  which would break the update model.

## ⚠️ The Critical Constraint — Never Add Forms or Server Modules to a Client Instance

Because a client instance (App 4) inherits all logic and UI live from `master_template`, adding a
form or server module directly to a client instance does not extend that instance — it silently
and permanently overrides the dependency for that instance only, with no error or warning. That
instance stops receiving any future update to `master_template`.

**If a change is needed for a specific client, it belongs in the development workspace (App 1)**,
tested against a private client instance, then merged into `master_template` — after which every
client instance, including the one that prompted the change, receives it automatically.

Every provisioned client instance should carry a README stating this explicitly from the moment
of provisioning (cloned in via `blank_client_template`), since this is not a mistake that reliably
gets caught by intuition alone — it does not transfer to an AI coding agent scoped to "fix
something for client X," a new team member, or anyone working under time pressure.

## Update Deployment Model

```
{dev-workspace} (develop branch)
    ↓ merge when tested and ready
master_template (stable branch)
    ↓ automatically
All client instances tracking stable branch
```

No manual per-client update action exists. Merging to `stable` is a production action — release
discipline (full testing in the dev workspace before merge) is the quality gate. For high-risk
releases, version tags allow staged rollout: a subset of clients updated first, remainder
following once stability is confirmed.

---

*Five-App Architecture Model v1.0. Extracted from platform-architecture.md as Mybizz-wide
content, confirmed applicable to every Mybizz product.*
