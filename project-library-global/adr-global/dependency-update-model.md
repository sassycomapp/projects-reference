# `dependency-update-model` ADR — Dependency Update Model

**Status:** Confirmed  
**Date:** 2026-06-13  
**Authority:** Derived from platform-overview.md, `client-instance-architecture` ADR, Anvil documentation

---

## Context

The platform requires a mechanism to release updates from the development workspace (mb-3-cs) to all client instances without manual per-client intervention. The original assumption — that master_template was a separately maintained app requiring manual overwrite or re-cloning — was incorrect and unworkable at scale.

The question was how to transfer a tested, stable version of mb-3-cs into master_template, and how client instances receive that update.

Anvil's dependency system supports depending on a named branch or a specific version tag of another app. This changes the update model entirely — no cloning or overwriting is required.

---

## Decision

### The Four-App Structure

| App | Purpose | GitHub Branch |
|---|---|---|
| `mb-3-cs` | Development workspace | `develop` |
| `master_template` | Published dependency | `stable` |
| `blank_client_template` | Provisioning clone source | tracks `master_template` stable |
| `Mybizz_management` | Platform operator tooling | separate repo |

### Update Flow

```
mb-3-cs  ←→  GitHub (develop branch)
                    ↓  merge when feature is ready and tested
master_template  ←→  GitHub (stable branch)
                    ↓  automatically, immediately
All client instances depending on master_template stable branch
```

1. All feature development happens in mb-3-cs on the `develop` branch
2. When a feature or fix is tested and ready for release, `develop` is merged into `stable` in GitHub
3. master_template, connected to the `stable` branch, reflects the update immediately
4. All client instances depending on master_template's `stable` branch receive the update automatically — no client action required
5. A release notification email is sent to clients via Anvil built-in email (internal notification system deferred to Mybizz_management phase)

### Client Notification

Clients receive a release notification via Anvil built-in email containing:
- What changed (release notes)
- Any action required on their part (rarely, if ever)

The email does not trigger a technical update action. The update has already happened at the point of sending. The email is informational only.

### Staged Rollout Option

Anvil supports version tags using the format `vMAJOR.MINOR.PATCH` (e.g. `v1.1.0`). For high-risk releases:

1. Tag the release commit in master_template's GitHub repo
2. Update a subset of client instances to the version tag instead of tracking `stable`
3. Monitor for issues
4. Once confirmed stable, roll remaining client instances to `stable` (or the tag)

All of the above is managed from the single mybizz Anvil account IDE.

---

## Alternatives Rejected

| Alternative | Reason rejected |
|---|---|
| Re-clone master_template per release | Changes app ID — breaks all existing client dependencies |
| Manual per-client update trigger | Unworkable at 100 clients; no Anvil provisioning API |
| Client-initiated pull via in-app button | No Anvil API to change dependency version from within the app |
| Separate Anvil account per client | 100 IDE logins to push one update; operationally untenable |

---

## Release Discipline

Because all client instances on the `stable` branch update immediately on merge, the quality gate is the merge decision itself. The following must be true before merging `develop` → `stable`:

- Feature is fully tested in mb-3-cs
- No breaking changes to existing Data Table schemas without a migration plan
- Release notes are written and ready to send
- Vault and integration dependencies are documented if changed

---

## Test Required

Branch-based update propagation has not yet been verified by live test. The following test must be conducted before the first client is onboarded:

1. Create `test-dependency-app` connected to GitHub `stable` branch
2. Create `test-client-app` depending on `test-dependency-app` stable branch
3. Make a change in `test-dependency-app`, merge to `stable`
4. Verify `test-client-app` reflects the change without any action on the client app

**ADR status moves to Fully Confirmed upon successful test completion.**

---

## Consequences

- ✅ Updates require no manual per-client action once the merge is made
- ✅ All client instances always run the same version (unless deliberately pinned)
- ✅ Staged rollout available via version tags without infrastructure changes
- ✅ Single Anvil account; single IDE; single billing relationship
- ⚠️ Merge to `stable` is a production action — release discipline is mandatory
- ✅ UI and form updates propagate automatically via master_template dependency (confirmed by Test B — see `client-instance-architecture` ADR)
- ⚠️ Schema changes require separate migration process for existing client instances

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/client-instance-architecture.md`` | Defines what lives in client instance vs master_template |
| ``adr/adr-global/blank-client-template.md`` | Provisioning clone source |
| ``adr/adr-global/free-trial-abandoned.md`` | Trial model replaced; all clients on stable from day one |
| `implementation/client-activation-runbook.md` | Includes dependency configuration step at provisioning |

---

*End of `dependency-update-model` ADR*
