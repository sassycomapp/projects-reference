# 22 — Tiers Model
Date: 2026-05-29
Status: Accepted
Source: User request - ADR creation for architectural enforcement
Updated: 2026-05-31 — Tier names, pricing, and definitions revised per plan-tune audit

---

## Context

The Mybizz CS platform uses commercial tiers for pricing and business model differentiation. The tier model needs clear definition to replace inconsistent framing in platform documentation.

### Problem Statement

The platform currently has inconsistent onboarding-tier framing in the platform overview. Commercial tiers need clear definition separate from setup-service options.

---

## Decision

**Commercial tiers are Launch, Pioneer, and Business.**

These are pricing/business-model constructs. They are separate from setup-service options such as guided self-setup and $100 DFY setup.

### Tier Definitions

| Tier | Clients | Monthly Price | Lifetime Price | Features |
|---|---|---|---|---|
| **Launch** | First 10 clients | $25/month | Yes — locked at $25/month for life | Full service, full feature access |
| **Pioneer** | 11th to 50th clients | $35/month | Yes — locked at $35/month for life | Full service, full feature access |
| **Business** | 51st client onwards | $50/month | No — price may change | Full service, full feature access |

### Architectural Rules

1. **Commercial Tiers:**
   - Launch, Pioneer, and Business
   - Pricing and business-model constructs
   - Define pricing and feature access
   - Launch and Pioneer tiers are grandfathered at their signup price for life

2. **Setup Services:**
   - Guided self-setup (included with all tiers)
   - $100 DFY (Done For You) setup
   - Separate from commercial tiers
   - Service options for onboarding assistance

3. **Clear Separation:**
   - Commercial tiers define pricing and features
   - Setup services define onboarding assistance options
   - No confusion between the two concepts

---

## Implementation Guidelines

### Tier Implementation
1. Clear tier definitions in business logic
2. Feature limits and pricing per tier
3. Upgrade/downgrade paths between tiers
4. Grandfathered pricing for Launch and Pioneer tiers
5. Clear documentation of tier differences

### Setup Service Implementation
1. Separate selection for setup service options
2. Clear pricing and description for each option
3. Integration with onboarding flow
4. No confusion with commercial tier selection

---

## Consequences

### Positive
- Clear commercial tier definitions with explicit pricing
- Consistent pricing and feature messaging
- Grandfathered pricing creates loyalty and early-adopter incentive
- No confusion between tiers and setup services

### Negative
- Requires tracking which tier each client is on for grandfathered pricing
- Launch and Pioneer pricing creates revenue ceiling for first 50 clients
- Business tier pricing is not fixed, requiring periodic review

### Neutral
- Primarily affects business model and documentation
- No impact on technical architecture
- Aligns with common SaaS tier patterns

---

## Related Documents

- `docs/platform-overview.md` — Current tier framing
- `docs/positioning-strategy.md` — Pricing and business model
- `implementation/onboarding-implementation-plan.md` — Onboarding flow

---

*End of `tiers-model` ADR*
