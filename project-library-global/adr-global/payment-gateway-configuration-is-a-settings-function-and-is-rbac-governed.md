# 19 — Payment Gateway Configuration Is a Settings Function and Is RBAC-Governed
Date: 2026-05-29
Status: Accepted
Source: User request - ADR creation for architectural enforcement

---

## Context

Payment gateway configuration is a critical operational function that requires secure access control. The platform needs to establish clear ownership and access patterns for gateway configuration.

### Problem Statement

Payment gateway configuration involves:
- Sensitive credentials and configuration
- Multiple potential administrators
- Ongoing maintenance and updates
- Security implications for financial operations

Without clear architecture:
- Unclear who can configure payment gateways
- Potential security risks from improper access control
- Inconsistent configuration patterns

### Current State

Payment gateway configuration could be placed in onboarding or settings, with unclear access control patterns.

---

## Decision

**Payment gateway configuration is performed through the Settings form.**

**Access is governed by RBAC (Role-Based Access Control).**

**Onboarding may only direct the Owner to the relevant Settings section and record completion state where needed.**

### Architectural Rules

1. **Configuration Location:**
   - All payment gateway configuration in Settings
   - Not part of onboarding flow
   - Clear access from main navigation

2. **Access Control:**
   - RBAC-governed access to gateway configuration
   - Only authorized roles can modify gateway settings
   - Clear audit trail for configuration changes

3. **Onboarding Role:**
   - Onboarding may indicate gateway setup is needed
   - Directs Owner to Settings for configuration
   - Can record completion state for onboarding progress
   - Does not perform actual gateway configuration

---

## Implementation Guidelines

### Settings Interface
1. Dedicated payment gateway configuration section
2. Clear RBAC controls for access
3. Secure credential storage and management
4. Configuration validation and testing

### Onboarding Flow
1. Indicate that payment gateway setup is required
2. Provide clear direction to Settings
3. Record completion state for onboarding progress
4. Do not include actual gateway configuration

### Access Control
1. Define roles that can access gateway configuration
2. Implement RBAC checks for all gateway operations
3. Audit logging for configuration changes
4. Secure credential handling

---

## Consequences

### Positive
- Clear separation of concerns (onboarding vs. configuration)
- RBAC ensures proper access control for sensitive operations
- Consistent configuration patterns across the platform
- Secure handling of payment credentials

### Negative
- Requires RBAC implementation for gateway access
- May require moving existing gateway configuration from onboarding
- Users need to navigate to Settings for gateway changes

### Neutral
- No impact on payment processing logic
- Primarily affects configuration and access patterns
- Aligns with security best practices for financial operations

---

## Related Documents

- ``adr/adr-global/onboarding-vs-settings-boundary.md`` — Onboarding vs settings boundary
- Security architecture documentation
- RBAC implementation guidelines

---

*End of `payment-gateway-configuration-is-a-settings-function-and-is-rbac-governed` ADR*