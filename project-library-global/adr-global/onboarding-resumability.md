# 23 — Onboarding Resumability
Date: 2026-05-29
Status: Accepted
Source: User request - ADR creation for architectural enforcement

---

## Context

The Mybizz CS platform onboarding flow captures substantial business information that is not realistically a single-sitting process. The system needs to support resumable onboarding.

### Problem Statement

Onboarding involves:
- Multiple configuration steps
- Information gathering that may require research
- Business decisions that take time
- External dependencies (documents, decisions, information)

Without resumability:
- Users forced to complete onboarding in one session
- Progress lost if user needs to step away
- Frustration and potential abandonment

### Current State

The onboarding flow captures substantial business information but may not support pausing and resuming.

---

## Decision

**Onboarding must be resumable so an Owner can leave, gather required information, and continue later without restarting from the beginning.**

### Architectural Rules

1. **Resumability Requirement:**
   - All onboarding progress must be preserved
   - Users can leave and return without data loss
   - Continue from last completed step
   - No need to restart from beginning

2. **Progress Preservation:**
   - Save progress at each step or meaningful checkpoint
   - Store partially completed information
   - Clear indication of completion status
   - Ability to review and modify previous steps

3. **User Experience:**
   - Clear indication of onboarding status
   - Easy access to resume onboarding
   - Progress visualization
   - Clear next steps when returning

---

## Implementation Guidelines

### Progress Saving
1. Save progress at each step or checkpoint
2. Store partially completed information
3. Track completion status for each section
4. Preserve all entered data

### Resume Flow
1. Clear entry point to resume onboarding
2. Display current progress and next steps
3. Allow review and modification of previous steps
4. Smooth continuation experience

### User Interface
1. Progress indicator showing completion status
2. Clear navigation between onboarding steps
3. Ability to save and exit at any point
4. Welcome back experience when resuming

---

## Consequences

### Positive
- Users can complete onboarding at their own pace
- No loss of progress if interrupted
- Reduced frustration and abandonment
- Better user experience for complex setup

### Negative
- Requires progress saving infrastructure
- May increase complexity of onboarding flow
- Need to handle partial data gracefully

### Neutral
- Primarily affects user experience and data persistence
- No impact on business logic
- Aligns with common SaaS onboarding patterns

---

## Related Documents

- ``adr/adr-global/onboarding-vs-settings-boundary.md`` — Onboarding scope definition
- User experience documentation
- Onboarding flow documentation

---

*End of `onboarding-resumability` ADR*