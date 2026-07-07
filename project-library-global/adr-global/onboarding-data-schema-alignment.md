# 20 — Onboarding Data Schema Alignment
Date: 2026-05-29
Status: Accepted
Source: User request - ADR creation for architectural enforcement

---

## Context

The Mybizz CS platform has an authoritative schema that defines the database structure. Onboarding processes must align with this schema to maintain data integrity and consistency.

### Problem Statement

Onboarding processes could potentially:
- Invent substitute table names or field structures
- Create client-side fields that conflict with the authoritative schema
- Lead to data inconsistencies between onboarding and main application

Without strict alignment:
- Data integrity issues between onboarding and main application
- Maintenance complexity from divergent schemas
- Potential data migration challenges

### Current State

The authoritative schema (`authoritative-schema.md`) defines the database structure. Onboarding must align with this schema.

---

## Decision

**All onboarding-related schema must align strictly with `authoritative-schema.md`.**

**`authoritative-schema.md` remains the sole source of truth for table and field structure.**

**No onboarding plan may invent substitute table names or client-side fields that conflict with the authoritative schema.**

### Architectural Rules

1. **Schema Authority:**
   - `authoritative-schema.md` is the sole source of truth
   - All database structures must match this schema
   - No exceptions for onboarding or any other process

2. **Onboarding Alignment:**
   - Onboarding must use exact table and field names from schema
   - No substitute or temporary structures allowed
   - Client-side fields must not conflict with schema

3. **Validation Enforcement:**
   - Onboarding plans must be validated against schema
   - Any deviations must be corrected before implementation
   - Schema changes must be reflected in onboarding

---

## Implementation Guidelines

### Onboarding Development
1. Reference authoritative schema for all data structures
2. Use exact table and field names from schema
3. Validate all onboarding data against schema
4. No client-side fields that conflict with schema

### Schema Management
1. Maintain authoritative schema as single source of truth
2. Any schema changes must be reflected in onboarding
3. Regular validation of onboarding against schema
4. Clear process for schema updates

### Validation Process
1. Onboarding plans must include schema validation step
2. Automated checks for schema alignment
3. Manual review for complex onboarding flows
4. Clear error messaging for schema conflicts

---

## Consequences

### Positive
- Data integrity between onboarding and main application
- Simplified maintenance with single schema source
- Consistent data structures across the platform
- Clear validation and enforcement process

### Negative
- Requires strict adherence to schema during onboarding development
- May require additional validation steps
- Schema changes require coordinated updates

### Neutral
- Primarily affects development and validation processes
- No impact on business logic
- Aligns with database best practices

---

## Related Documents

- `authoritative-schema.md` — Sole source of truth for database schema
- ``adr/adr-global/onboarding-vs-settings-boundary.md`` — Onboarding scope definition
- Database architecture documentation

---

*End of `onboarding-data-schema-alignment` ADR*