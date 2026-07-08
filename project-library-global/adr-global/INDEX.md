# Global ADR Index

Architecture Decision Records that apply to all Anvil.works projects. Check this index before making architecture or technology decisions. Read only the specific ADR relevant to the current decision.

Full index with statuses and dates: `adr-index.md` (95 lines, detailed).

## Quick reference — Key ADRs by category

### Architecture
- `client-instance-architecture.md` — Dependency-based architecture, not multi-tenant
- `dependency-based-not-multi-tenant.md` — Prohibits tenant discriminator columns
- `dependency-update-model.md` — Four-app update flow
- `data-access-patterns.md` — Data access patterns and query limitations

### Security & Payments
- `payment-security-boundary-vault.md` — Payment security via Vault
- `payment-gateway-configuration-is-a-settings-function-and-is-rbac-governed.md` — RBAC-governed settings
- `client-data-management-rights-and-mybizz-retention-boundary.md` — Data rights boundary

### UI & Design
- `design-rules.md` — Design rules
- `material-3-theme-component-scope.md` — M3 component scope
- `responsive-behaviour-mechanism.md` — Responsive via `wrap_on`, not CSS breakpoints
- `htmltemplate-use.md` — HTMLTemplate is banned
- `dark-mode-v1.md` — Dark mode in V1
- `ui-customization-approach.md` — M3+CSS customization

### Data & Currency
- `system-currency-selection-and-immutability.md` — Currency rules
- `timezone-utc-storage-display-conversion.md` — UTC storage, display conversion
- `single-contacts-table.md` — Single contacts table
- `tiers-model.md` — Tiers model

### Integration & Platform
- `anvil-platform-constraints.md` — Platform constraints
- `anvil-extras-exclusion.md` — No Anvil Extras
- `webhook-architecture.md` — Webhook patterns
- `observability-architecture.md` — Observability design

### Onboarding
- `onboarding-finality.md` — Onboarding is resumable
- `onboarding-resumability.md` — Resumability mechanism
- `onboarding-vs-settings-boundary.md` — Onboarding vs settings boundary
- `onboarding-data-schema-alignment.md` — Data schema alignment
