# Mybizz CS — Deployment Procedures

**Authority:** `dependency-update-model` ADR (Dependency Update Model), `blank-client-template` ADR (blank_client_template), `anvil-platform-constraints` ADR (Anvil Platform Constraints)

---

## Overview

This document defines the deployment procedures for the Mybizz CS platform. It covers the update deployment model, schema migration strategy, staged rollout, rollback procedures, and post-deploy verification.

**Key architectural decision:** The platform uses a branch-based update model. Merging `develop` → `stable` in GitHub releases an update to all client instances automatically. See `dependency-update-model` ADR.

---

## Deployment Model

### The Update Flow

```
mb-3-cs (GitHub develop branch)
    ↓ merge when tested and ready
master_template (GitHub stable branch)
    ↓ automatically — no manual action
All client instances tracking stable branch
```

**There is no manual per-client update action.** Merging to `stable` is a production action. Release discipline — full testing in mb-3-cs before merge — is the quality gate. "A single merge to stable propagates all three categories of change simultaneously: server logic, UI, and forms. The only change that does not propagate automatically is Data Table schema — schema changes are always manual operations per client instance."

### Release Process

1. **Develop** — All feature work happens in mb-3-cs on the `develop` branch
2. **Test** — Feature is fully tested in mb-3-cs
3. **Prepare release notes** — Document what changed, any action required
4. **Merge** — `develop` is merged into `stable` in GitHub
5. **Verify** — Confirm client instances reflect the update
6. **Notify** — Send release notification email to clients via Anvil built-in email

### Staged Rollout (High-Risk Releases)

For high-risk releases, version tags (`v1.0.0`, `v1.1.0`) allow staged rollout:

1. Tag the release commit in master_template's GitHub repo
2. Update a subset of client instances to the version tag (instead of tracking `stable`)
3. Monitor for issues
4. Once confirmed stable, roll remaining client instances to `stable`

All managed from the single mybizz Anvil IDE.

### Client Notification

Clients receive a release notification via Anvil built-in email. The notification is informational — the update has already been applied at the time of sending.

An internal in-app notification system is planned for the Mybizz_management phase.

---

## Schema Migration Strategy

### The Challenge

Anvil has no programmatic API for creating or modifying Data Tables. Schema changes must be applied via the IDE. With 100 client instances, schema migrations are manual operations.

### Migration Rules

1. **Schema changes must never be merged to `stable` without a documented migration plan**
2. **Every schema change must be applied to `blank_client_template`** (for future clients)
3. **Every schema change must be applied manually to all existing client instances** (for running clients)

### Migration Process

**For new columns (additive):**
1. Add the column to `blank_client_template` via IDE
2. Add the column to each existing client instance via IDE
3. Deploy code that reads both old and new fields
4. Merge to `stable` — code propagates automatically
5. Existing rows will have NULL for the new column — code must handle this

**For removed columns (breaking):**
1. Deploy code that no longer references the old column
2. Merge to `stable` — code propagates automatically
3. Remove the column from each existing client instance via IDE
4. Remove the column from `blank_client_template` via IDE

**For modified columns:**
1. Add a new column with the desired schema
2. Deploy code that reads from the new column (with fallback to old)
3. Merge to `stable`
4. Migrate data from old column to new column (manual or script)
5. Remove the old column

### Migration Checklist

- [ ] Schema change documented in release notes
- [ ] `blank_client_template` updated
- [ ] Migration script/procedure documented for existing clients
- [ ] Code handles both old and new schema (backward compatible)
- [ ] Rollback procedure documented
- [ ] Migration tested on at least one existing client instance

---

## Post-Deploy Verification

### Automated Checks

After merging to `stable`, verify:

1. **Health endpoint** — Call `/management/health` on a sample of client instances
2. **Background tasks** — Verify scheduled tasks are running (health heartbeat, campaign enrollment)
3. **Error rate** — Check `error_log` table for new errors

### Manual Verification

For significant releases:

1. **Login** to a sample client instance
2. **Verify** key functionality (booking flow, payment processing, email delivery)
3. **Check** dashboard metrics are displaying correctly
4. **Confirm** no new errors in Anvil App Logs

---

## Rollback Procedures

### Code Rollback

If a release introduces a critical bug:

1. **Revert** the merge commit in GitHub (`develop` branch)
2. **Merge** the revert to `stable`
3. **All client instances update automatically** to the reverted code
4. **Notify** clients of the rollback

**Note:** Rollback via merge is the safest approach. It preserves git history and is reversible.

### Schema Rollback

If a schema change causes issues:

1. **Revert** the code change that depends on the new schema
2. **Merge** the revert to `stable`
3. **Manually remove** the new column from affected client instances (if it was added)
4. **Update** `blank_client_template` if the schema change was applied there

---

## Deployment Checklist

### Pre-Deployment

**Code:**
- [ ] All tests passing in mb-3-cs
- [ ] Feature fully tested
- [ ] Release notes prepared
- [ ] Schema changes documented (if applicable)

**Database:**
- [ ] Schema migration plan documented (if applicable)
- [ ] `blank_client_template` updated (if applicable)
- [ ] Rollback procedure documented

**Communication:**
- [ ] Release notes ready to send
- [ ] Stakeholders notified (if significant release)

### Deployment

1. [ ] Merge `develop` → `stable` in GitHub
2. [ ] Verify master_template reflects the update
3. [ ] Verify sample client instances reflect the update
4. [ ] Monitor error rates for 15 minutes
5. [ ] Send release notification email to clients

### Post-Deployment

**Monitoring:**
- [ ] Monitor error rates for 1 hour
- [ ] Check background task health
- [ ] Review client support tickets

**Documentation:**
- [ ] Update deployment log
- [ ] Document any issues encountered

---

## Rollback Triggers

**Immediate Rollback Required:**
- Error rate > 10% across client instances
- Critical functionality broken (booking flow, payments)
- Data corruption detected
- Security vulnerability introduced

**Rollback Considered:**
- Error rate > 5%
- Performance degradation > 50%
- Customer complaints increasing

---

## Anvil Account-Level Failure Scenarios

**Source:** CEO review 2026-06-15 Section 1

All 100 client instances are hosted on a single Anvil Business account. This creates a single-point-of-failure at the account level. The following scenarios and recovery procedures address this operational risk.

### Scenario 1: Anvil Account Suspension

**Cause:** Terms of service violation, billing failure, or Anvil policy enforcement.

**Impact:** All client instances become inaccessible simultaneously. No client can log in, process bookings, or access data.

**Detection:**
- Health heartbeat background tasks stop writing to `health_log`
- Mybizz_management monitoring dashboard shows all instances offline
- Client support tickets spike

**Recovery:**
1. Contact Anvil support immediately to understand the suspension reason
2. Resolve the underlying issue (billing, policy, etc.)
3. Request account reinstatement
4. Once reinstated, verify all client instances are accessible
5. Run health checks across all instances
6. Notify affected clients with a status update

**Prevention:**
- Maintain billing payment method with redundancy
- Stay current with Anvil Terms of Service changes
- Maintain direct contact relationship with Anvil support team

### Scenario 2: Anvil Account Compromise

**Cause:** Credential theft, social engineering, or Anvil platform breach.

**Impact:** Attacker may access the Anvil IDE, modify code, access Data Tables, or delete apps.

**Detection:**
- Unexpected code changes visible in git history
- Unknown user sessions in Anvil IDE
- Unusual API activity or data access patterns

**Recovery:**
1. Immediately change all Anvil account credentials
2. Enable 2FA on the Anvil account (if not already enabled)
3. Audit git history for unauthorized commits
4. Audit Data Table changes for unauthorized modifications
5. If code was modified: revert to last known-good commit
6. If data was accessed: notify affected clients per data breach protocol
7. Review and rotate all Vault encryption keys

### Scenario 3: Anvil Platform Outage

**Cause:** Anvil infrastructure failure, DDoS, or cloud provider outage.

**Impact:** All client instances are temporarily unavailable.

**Detection:**
- Health heartbeat tasks fail
- Client reports of inaccessible platform

**Recovery:**
1. Monitor Anvil status page and support channels
2. Communicate expected resolution timeline to clients
3. Once restored: verify all instances are healthy
4. Review `health_log` for any data gaps during outage

**Note:** This is an accepted risk of the platform decision (`anvil-platform-constraints` ADR). No self-hosted fallback is planned.

---

*End of file — Deployment Procedures*
