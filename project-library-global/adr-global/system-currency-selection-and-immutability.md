# 16 — System Currency, Display Currency, and Immutability
Date: 2026-05-29
Status: Accepted
Source: Consolidated from `013-system-currency-setting.md` and `016-system-currency-selection-and-immutability.md`
Updated: 2026-06-01 — Merged `system-currency-setting` ADR (deleted) (currency architecture, conversion strategy, schema changes) with `system-currency-selection-and-immutability` ADR (immutability enforcement, display currency separation)
Supersedes: `013-system-currency-setting.md`

---

## Context

The Mybizz CS platform supports multi-currency operations for global service businesses. Small service businesses operate in different currencies globally. The system needs to:

1. Have a single system currency for all internal operations (accounting, reporting, analytics)
2. Allow clients to set prices in different currencies for specific services/locations
3. Handle currency conversion when needed
4. Enforce immutability of system currency after the first transaction
5. Clearly separate system currency from display currency

### Existing Design

From `authoritative-schema.md`:
- `business_profile.system_currency` — set at onboarding, immutable after first transaction
- `services.price` — price in system currency
- `services.display_currency` — optional display currency for customer-facing prices
- `invoice.currency` — transaction currency (may differ from system currency)

---

## Decision

**System Currency is selected during onboarding and becomes immutable after the first transaction.**

**Display Currency remains a separate configurable concept outside onboarding.**

**Currency conversion follows Option A (Fixed Exchange Rates) for V1, with Option B (API-Based Rates) as V2 enhancement.**

### Architectural Rules

1. **System Currency Selection:**
   - Configured exclusively during business onboarding
   - Stored in `business_profile.system_currency` as ISO 4217 code
   - Cannot be changed after the first transaction is recorded
   - Used as the base currency for all internal calculations
   - Default currency for all services

2. **Display Currency:**
   - Optional per-service or per-location override
   - Configurable at any time in service management
   - Does not affect internal accounting or reporting
   - Used purely for customer-facing price presentation

3. **Immutability Enforcement:**
   - System currency changes must be blocked at the application layer after the first transaction
   - Database schema should support this constraint
   - Any attempt to modify system currency after immutability point should fail with clear error messaging

4. **Currency Conversion (V1):**
   - Fixed exchange rates (manual entry by business)
   - Rates updated periodically (weekly/monthly)
   - Simple, no external dependencies
   - V2 enhancement: API-based real-time rates (e.g., exchangerate.host)

### Currency Architecture

```
ONBOARDING → system_currency set (immutable after first transaction)
    ↓
ALL INTERNAL OPERATIONS use system_currency
    ↓
SERVICE PRICING:
  - Default: price in system_currency
  - Optional: display_currency override per service/location
    ↓
CUSTOMER SEES: display_currency price (if set)
    ↓
INTERNAL: system_currency calculation
    ↓
INVOICE: transaction currency + system_currency_amount (for reporting)
```

### Schema Changes

**business_profile table:**
```
| Field | Type | Notes |
|-------|------|-------|
| system_currency | string | ISO 4217 (e.g., "ZAR", "USD") — set at onboarding, immutable after first transaction |
```

**services table:**
```
| Field | Type | Notes |
|-------|------|-------|
| price | number | Price in system_currency |
| display_currency | string | ISO 4217 — optional display currency for customer-facing prices |
```

**invoice table:**
```
| Field | Type | Notes |
|-------|------|-------|
| currency | string | ISO 4217 — transaction currency |
| total_amount | number | Amount in transaction currency |
| system_currency_amount | number | Amount converted to system_currency (for reporting) |
```

---

## Implementation Guidelines

### Onboarding Flow
1. System currency selection step during business profile creation (dropdown + confirm button)
2. Clear explanation that this choice is permanent for accounting purposes
3. Default to common local currency based on business location (if available)
4. After first transaction, system_currency becomes immutable

### Service Configuration
1. Display currency setting available in service management
2. Optional field that defaults to system currency
3. Can be changed without affecting existing transactions

### Transaction Processing
1. All internal calculations use system currency
2. Display currency used only for customer-facing price display
3. Currency conversion (when implemented) converts to system currency for storage

### Invoice Generation Flow
1. Customer sees display_currency price (if set)
2. Invoice generated in transaction currency
3. system_currency_amount calculated and stored for reporting
4. Accounting uses system_currency_amount

---

## Consequences

### Positive
- Single source of truth for accounting
- Flexibility for local pricing via display currency
- Clear separation of display vs. internal currency
- Immutability ensures accounting integrity and consistent reporting
- Prevents accidental or unauthorized changes to financial baseline

### Negative
- Businesses cannot change their system currency if they relocate or restructure
- Currency conversion complexity (V1 uses manual fixed rates)
- Need to handle exchange rate updates
- Potential customer confusion if displayed price differs from charged price
- May need migration path for edge cases (though these should be rare)

### Neutral
- System currency immutability simplifies accounting
- Local/display currency is optional, not required
- Currency conversion (when implemented) respects the system currency as target
- No impact on existing transaction history

---

## Alternatives Considered

### Alternative 1: Multi-Currency System
- Allow multiple system currencies
- Cons: Complex accounting, reporting becomes difficult

### Alternative 2: Dynamic Currency Conversion
- Convert all prices at time of transaction
- Cons: Exchange rate volatility, customer disputes

### Alternative 3: No Local/Display Currency Support
- All services priced in system_currency only
- Pros: Simplest
- Cons: Limited global flexibility

---

## Related Documents

- `authoritative-schema.md` — Database schema with currency fields
- `5-timezone-utc-storage-display-conversion.md` — Timezone/UTC storage pattern (similar approach)
- `implementation/onboarding-implementation-plan.md` — System currency selection in onboarding

---

*End of `system-currency-selection-and-immutability` ADR — Consolidated from `system-currency-setting` ADR (deleted) and `system-currency-selection-and-immutability` ADR*
