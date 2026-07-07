# 18 — Legal Policy Responsibility Acknowledgement and Clause-Builder Architecture
Date: 2026-05-29
Status: Accepted
Source: User request - ADR creation for architectural enforcement

---

## Context

The Mybizz CS platform requires businesses to have legal policies (Privacy Policy and Terms & Conditions) for their operations. The platform needs to clarify responsibility for policy content while providing tools to assist with policy creation.

### Problem Statement

Legal policies involve:
- Content responsibility (who creates and maintains the content)
- Platform support (how the system assists with policy creation)
- Publication management (how policies are made available to clients)

Without clear architecture:
- Ambiguity about who is responsible for policy content
- Inconsistent policy management across the platform
- Legal risk if platform assumes content responsibility

### Current State

The platform needs to support policy creation and publication while maintaining clear separation of responsibilities between the platform and business owners.

---

## Decision

**The Owner is solely responsible for the content of their Privacy Policy and Terms & Conditions.**

**Onboarding records explicit acknowledgement of that responsibility.**

**Document authoring is supported by a reusable clause-builder feature.**

### Architectural Rules

1. **Content Responsibility:**
   - Business Owner is solely responsible for policy content
   - Platform provides tools but does not create or validate content
   - Clear legal separation between platform and business operations

2. **Acknowledgement Recording:**
   - Onboarding includes explicit acknowledgement step
   - Owner confirms understanding of their responsibility
   - Acknowledgement recorded for legal protection

3. **Clause-Builder Feature:**
   - Reusable component for policy document creation
   - Provides common clauses and templates
   - Allows customization and editing
   - Supports publication to corresponding public pages

---

## Implementation Guidelines

### Onboarding Flow
1. Include policy responsibility acknowledgement step
2. Clear explanation of Owner's content responsibility
3. Record acknowledgement with timestamp and user context
4. Provide access to clause-builder for policy creation

### Clause-Builder Architecture
1. Reusable component accessible from onboarding and settings
2. Common clause library for privacy policies and terms
3. Custom editing and customization capabilities
4. Preview and publication workflow

### Publication Management
1. Policies published to corresponding public pages
2. Version management for policy updates
3. Clear publication status and dates
4. Integration with business profile and public-facing pages

---

## Consequences

### Positive
- Clear legal separation of responsibilities
- Platform provides support without assuming content risk
- Consistent policy management across the platform
- Reusable clause-builder reduces policy creation effort

### Negative
- Requires careful legal review of clause library
- Platform must not provide legal advice or content validation
- Owner must understand their responsibility

### Neutral
- Primarily affects legal and compliance aspects
- No impact on core business logic
- Aligns with common SaaS patterns for legal content

---

## Related Documents

- ``adr/adr-global/onboarding-vs-settings-boundary.md`` — Onboarding vs settings boundary
- Legal documentation requirements
- Platform terms of service

---

*End of `legal-policy-responsibility-acknowledgement-and-clause-builder-architecture` ADR*