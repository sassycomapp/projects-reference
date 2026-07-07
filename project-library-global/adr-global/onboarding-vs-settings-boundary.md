# 17 — Onboarding vs Settings Boundary
Date: 2026-05-29
Status: Accepted
Source: User request - ADR creation for architectural enforcement

---

## Context

The Mybizz CS platform distinguishes between initial business establishment (onboarding) and ongoing operational configuration (settings). The boundary between these two concerns needs explicit definition to prevent onboarding from becoming a sprawling, multi-purpose configuration surface.

### Problem Statement

Without clear boundaries:
- Onboarding could accumulate mutable operational controls, making it complex and confusing
- Users might expect to return to onboarding for changes that belong in settings
- The platform could develop inconsistent configuration patterns

### Current State

The platform currently has some operational controls (like palette selection, service catalogue setup, gateway setup) that could be placed in either onboarding or settings. This creates ambiguity about where configuration belongs.

---

## Decision

**Onboarding is for initial establishment and acknowledgement only.**

**Ongoing operational configuration belongs to Settings and other operating forms.**

### Architectural Rules

1. **Onboarding Scope:**
   - Initial business profile establishment (name, timezone, system currency)
   - Required legal acknowledgements (privacy policy, terms & conditions)
   - One-time setup that doesn't change during normal operations
   - Clear completion state tracking

2. **Settings Scope:**
   - All mutable operational controls
   - Palette selection and theme configuration
   - Service catalogue setup and management
   - Gateway setup and configuration
   - Other business-specific operational settings

3. **Boundary Enforcement:**
   - Onboarding forms must not include controls that belong in settings
   - Settings must provide clear access to all operational configuration
   - The transition from onboarding to settings must be explicit

---

## Implementation Guidelines

### Onboarding Flow
1. Focus on essential business establishment
2. Include only one-time configuration (system currency, timezone)
3. Record acknowledgements for legal policies
4. Provide clear completion state
5. Direct users to Settings for ongoing configuration

### Settings Interface
1. Centralized location for all operational controls
2. Clear organization by functional area (palette, services, gateways)
3. Consistent access patterns regardless of onboarding completion
4. RBAC-governed access where appropriate

### Transition Design
1. Onboarding completion should clearly indicate next steps
2. Settings should be easily accessible from main navigation
3. No operational control should require returning to onboarding

---

## Consequences

### Positive
- Clear separation of concerns between establishment and operation
- Simplified onboarding flow focused on essential setup
- Consistent configuration patterns across the platform
- Users know exactly where to find operational controls

### Negative
- Requires careful classification of what belongs where
- May require moving existing controls from onboarding to settings
- Users accustomed to current patterns may need guidance

### Neutral
- No impact on data model or business logic
- Primarily affects UI organization and user flow
- Aligns with common SaaS patterns for onboarding vs. settings

---

## Related Documents

- ``adr/adr-global/system-currency-selection-and-immutability.md`` — System currency as onboarding concern
- `docs/phased-implementation/phase-0-implementation.md` — Implementation phases
- Platform overview documentation — Current configuration patterns

---

*End of `onboarding-vs-settings-boundary` ADR*