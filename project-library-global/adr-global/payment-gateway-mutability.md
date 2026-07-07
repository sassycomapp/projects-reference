# 24 — Payment Gateway Mutability
Date: 2026-05-29
Status: Accepted
Source: User request - ADR creation for architectural enforcement

---

## Context

The Mybizz CS platform currently states that each client instance uses one payment gateway configured at onboarding. This creates a question about whether the gateway assignment can be changed after onboarding.

### Problem Statement

The current platform overview describes:
- Each client instance uses one payment gateway
- Gateway is configured at onboarding
- No explicit statement about post-onboarding changes

This creates ambiguity about:
- Whether gateway assignment is immutable after onboarding
- What happens if a client needs to change gateways
- Platform rules for post-onboarding gateway changes

### Current State

From platform documentation:
- Client instance uses one payment gateway
- Gateway configured at onboarding
- No explicit mutability rules defined

---

## Decision

**A client instance has one active payment gateway at a time.**

**Gateway assignment may be changed after onboarding only under approved platform rules.**

### Architectural Rules

1. **Single Active Gateway:**
   - Each client instance has exactly one active payment gateway
   - Only one gateway can process transactions at any time
   - Clear designation of active gateway

2. **Post-Onboarding Changes:**
   - Gateway assignment may be changed after onboarding
   - Changes subject to approved platform rules
   - Not a free-for-all change; requires specific conditions

3. **Platform Rules for Changes:**
   - Define approved conditions for gateway changes
   - Document required approvals or validation
   - Maintain audit trail for gateway changes
   - Ensure continuity of payment processing

---

## Implementation Guidelines

### Gateway Management
1. Clear UI for viewing current active gateway
2. Interface for requesting gateway changes
3. Validation against platform rules for changes
4. Audit logging for all gateway changes

### Platform Rules Definition
1. Define approved conditions for gateway changes
2. Document required approvals or validation steps
3. Implement checks for rule compliance
4. Maintain clear change history

### Change Process
1. Request gateway change through Settings
2. Validate against platform rules
3. Process change with proper validation
4. Update active gateway designation
5. Maintain transaction continuity

---

## Consequences

### Positive
- Clear single-active-gateway architecture
- Flexibility for legitimate gateway changes
- Platform control over change conditions
- Maintains payment processing continuity

### Negative
- Requires definition of platform rules for changes
- May require additional validation logic
- Need to handle existing transactions during changes

### Neutral
- No impact on payment processing logic
- Primarily affects gateway management and change control
- Aligns with common SaaS payment patterns

---

## Related Documents

- ``adr/adr-global/payment-gateway-configuration-is-a-settings-function-and-is-rbac-governed.md`` — Gateway configuration in Settings
- Platform overview documentation — Current gateway description
- Payment processing architecture

---

*End of `payment-gateway-mutability` ADR*