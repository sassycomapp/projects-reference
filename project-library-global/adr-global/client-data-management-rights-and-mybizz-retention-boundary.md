# 21 — Client Data Management Rights and Mybizz Retention Boundary
Date: 2026-05-29
Status: Accepted
Source: User request - ADR creation for architectural enforcement

---

## Context

The Mybizz CS platform handles client data with clear separation between business owner data and platform management data. The boundary between these data domains needs explicit definition.

### Problem Statement

Client data management involves:
- Owner-held client instance data (business operations)
- Mybizz-held management-side records (platform operations)
- Export and management rights for business owners
- Retention requirements for legal compliance

Without clear architecture:
- Ambiguity about data ownership and access
- Potential legal compliance issues
- Unclear data management responsibilities

### Current State

The platform needs to clearly distinguish between owner-held client-instance data and Mybizz-held management-side records.

---

## Decision

**The Owner must have access to export and manage eligible client data.**

**Mybizz retains legally required records in the Mybizz_management application under its own obligations.**

**The system must clearly distinguish Owner-held client-instance data from Mybizz-held management-side records.**

### Architectural Rules

1. **Owner Data Rights:**
   - Access to export eligible client data
   - Management capabilities for business operations data
   - Clear data ownership and control

2. **Mybizz Retention:**
   - Retains legally required records in Mybizz_management
   - Platform management data under Mybizz obligations
   - Clear separation from business owner data

3. **Data Distinction:**
   - Clear technical separation between data domains
   - Different access patterns and permissions
   - Explicit labeling of data ownership

---

## Implementation Guidelines

### Owner Data Access
1. Export functionality for eligible client data
2. Management interface for business operations data
3. Clear documentation of data rights and limitations
4. Secure access patterns for data management

### Mybizz Data Management
1. Mybizz_management application for platform records
2. Legally required retention under Mybizz obligations
3. Clear separation from business owner data
4. Appropriate access controls for management data

### Data Distinction Implementation
1. Technical separation of data domains
2. Clear labeling of data ownership
3. Different access patterns and permissions
4. Documentation of data boundaries

---

## Consequences

### Positive
- Clear data ownership and rights for business owners
- Legal compliance for platform retention requirements
- Technical separation of data domains
- Clear documentation and user understanding

### Negative
- Requires implementation of data export and management
- May require additional UI for data distinction
- Legal review of data retention requirements

### Neutral
- Primarily affects data access and management patterns
- No impact on core business logic
- Aligns with data protection best practices

---

## Related Documents

- Data protection and privacy documentation
- Mybizz management application architecture
- Legal compliance requirements

---

*End of `client-data-management-rights-and-mybizz-retention-boundary` ADR*