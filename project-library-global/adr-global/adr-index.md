# Mybizz — ADR Index

**Last Updated:** 2026-07-06
**Total ADRs:** 40 (39 global, 1 local) — 25 Accepted, 7 Confirmed, 1 Superseded, 7 Approved

---

## Global ADRs (`adr-global/`)

Company-wide architecture, technology, and standards decisions — apply to all Mybizz applications.

| ADR File | Title | Status | Date |
|---|---|---|---|
| `brevo-replaces-zoho-email.md` | Brevo Replaces All Zoho Products | Accepted | 2026-03-21 |
| `navigation-lambda-link-open-form.md` | Navigation Standard: Lambda/Link/open_form | Accepted | 2026-03-18 |
| `payment-security-boundary-vault.md` | Payment Security Boundary: Vault | Accepted | 2026-03-15 |
| `lead-capture-simultaneous-creation.md` | Lead Capture: Simultaneous Creation | Accepted | 2026-03-21 |
| `timezone-utc-storage-display-conversion.md` | Client Timezone: UTC Storage, Display Conversion | Accepted | 2026-03-17 |
| `crm-package-replaces-marketing.md` | Package Name: crm/ Replaces marketing/ | Accepted | 2026-03-18 |
| `single-contacts-table.md` | Single Contacts Table Replaces Dual contacts/customers | Accepted | 2026-05-03 |
| `anvil-extras-exclusion.md` | Anvil Extras Exclusion | Accepted | 2026-05-21 |
| `material-3-theme-component-scope.md` | Material 3 Theme Component Scope | Accepted | 2026-05-21 |
| `system-currency-selection-and-immutability.md` | System Currency, Display Currency, and Immutability | Accepted | 2026-05-29 |
| `onboarding-vs-settings-boundary.md` | Onboarding vs Settings Boundary | Accepted | 2026-05-29 |
| `legal-policy-responsibility-acknowledgement-and-clause-builder-architecture.md` | Legal Policy Responsibility Acknowledgement and Clause-Builder Architecture | Accepted | 2026-05-29 |
| `payment-gateway-configuration-is-a-settings-function-and-is-rbac-governed.md` | Payment Gateway Configuration Is a Settings Function and Is RBAC-Governed | Accepted | 2026-05-29 |
| `onboarding-data-schema-alignment.md` | Onboarding Data Schema Alignment | Accepted | 2026-05-29 |
| `client-data-management-rights-and-mybizz-retention-boundary.md` | Client Data Management Rights and Mybizz Retention Boundary | Accepted | 2026-05-29 |
| `tiers-model.md` | Tiers Model | Accepted | 2026-05-29 |
| `onboarding-resumability.md` | Onboarding Resumability | Accepted | 2026-05-29 |
| `payment-gateway-mutability.md` | Payment Gateway Mutability | Accepted | 2026-05-29 |
| `onboarding-finality.md` | Onboarding Finality | Accepted | 2026-06-01 |
| `design-rules.md` | Design Rules | Accepted | 2026-06-10 |
| `client-instance-architecture.md` | Client Instance Architecture | Confirmed | 2026-06-13 |
| `dependency-update-model.md` | Dependency Update Model | Confirmed | 2026-06-13 |
| `blank-client-template.md` | blank_client_template as Provisioning Clone Source | Confirmed | 2026-06-13 |
| `free-trial-abandoned.md` | 30-Day Free Trial Abandoned | Confirmed | 2026-06-13 |
| `observability-architecture.md` | Observability Architecture | Confirmed | 2026-06-13 |
| `mybizz-management-visibility.md` | Mybizz_management Visibility and Control | Confirmed | 2026-06-13 |
| `anvil-platform-constraints.md` | Anvil Platform Constraints and Design Boundaries | Approved | 2026-06-14 |
| `form-architecture-and-state.md` | Form Architecture and State Management | Approved | 2026-06-14 |
| `real-time-and-background-tasks.md` | Real-Time Updates and Background Task Architecture | Approved | 2026-06-14 |
| `ui-customization-approach.md` | UI Customization Approach | Approved | 2026-06-14 |
| `webhook-architecture.md` | Webhook Architecture | Approved | 2026-06-14 |
| `pdf-invoice-generation.md` | PDF Invoice Generation | Approved | 2026-06-14 |
| `data-access-patterns.md` | Data Access Patterns and Query Limitations | Approved | 2026-06-14 |
| `dependency-based-not-multi-tenant.md` | Dependency-Based Architecture, Not Multi-Tenant | Confirmed | 2026-06-13 |
| `client-instance-readme-five-app-system.md` | Mandatory README in Every Client Instance Documenting the Five-App System | Accepted | 2026-06-28 |
| `role-property-assignment-mechanism.md` | Role Property Assignment Mechanism (Designer vs. Code) | **Superseded** | 2026-06-27 |
| `responsive-behaviour-mechanism.md` | Responsive Behaviour Mechanism | Accepted | 2026-06-27 |
| `htmltemplate-use.md` | HTMLTemplate Use | Accepted | 2026-06-27 |
| `dark-mode-v1.md` | Dark Mode — V1 Inclusion | Accepted | 2026-07-03 |

---

## Local ADRs (`adr-local/`)

App-specific decisions — apply to mb-3-cs (Consulting & Services) only.

| ADR File | Title | Status | Date |
|---|---|---|---|
| `schema-scope-reduction.md` | Schema Scope Reduction | Accepted | 2026-05-03 |

---

## Notes

### Reorganization (2026-07-06)

- **Numeric prefixes dropped.** ADRs are now referenced by descriptive slug, not number. All files renamed from `NN-slug.md` to `slug.md`.
- **Global/local split.** 39 ADRs apply company-wide; 1 ADR is CS-specific. Contents validated per-ADR before classification.
- **4 ADRs deleted** as superseded/historical: `multi-vertical-to-single-vertical-conversion` (historical, no longer relevant), `consolidated-build-sequence` (superseded), `system-currency-setting` (superseded by `system-currency-selection-and-immutability`), `onboarding-finality` (cancelled — fully reversed by replacement `onboarding-finality`).

### Legacy numeric references

- **ADR-07 and ADR-08:** Permanently skipped — numbers assigned to obsolete decisions, files never created. These numbers are now retired; no ADRs occupy these slots.

### Notable ADRs

- **`system-currency-selection-and-immutability.md`:** Consolidated from original ADR-13 and ADR-16. Covers system currency, display currency, immutability enforcement, and currency conversion strategy.
- **`onboarding-finality.md`:** Created 2026-05-31, updated 2026-06-01. Onboarding is resumable and revisitable. Owners may change any credential at any time. Mybizz_management maintains an append-only amendment log with three data tables. Reversed and fully replaced the original `onboarding-finality` (deleted).
- **`client-instance-architecture.md`:** Defines the dependency-based architecture. Confirmed by live testing (Test A: app_tables resolution, Test B: forms from dependency). Foundation for all data access patterns.
- **`dependency-update-model.md`:** Four-app update flow (mb-3-cs → master_template → client instances). Branch propagation confirmed.
- **`anvil-platform-constraints.md` through `data-access-patterns.md`:** Created 2026-06-14. Consolidated decisions from the "Issues to be resolved" section of platform-overview.md.
- **`dependency-based-not-multi-tenant.md`:** Definitive statement that Mybizz CS is a dependency-based SaaS platform, not a multi-tenant application. Prohibits tenant discriminator columns and tenant-filtered queries.
- **`client-instance-readme-five-app-system.md`:** Every client instance must contain a standardized README documenting the five-app architecture and the constraint against adding forms or modules directly to client instances.
- **`role-property-assignment-mechanism.md`:** Superseded. Original decision: set `role` via Designer Properties Panel. Reversed by project-wide property-setting rule: if a property can be set programmatically, it must be set programmatically. Current decision: set `role` in code (`self.component.role = "role-name"`).
- **`responsive-behaviour-mechanism.md`:** Resolves contradicting breakpoint models in design-direction.md. Responsive behaviour uses `wrap_on` per-container, not CSS breakpoint tables. Nav collapse is automatic and separate.
- **`htmltemplate-use.md`:** `HTMLTemplate` is banned. All current use cases have native M3 alternatives. Wireframes using it require reworking.
- **`dark-mode-v1.md`:** Dark mode incorporated into V1. No longer deferred to V2. All screens require `@media (prefers-color-scheme: dark)` blocks.
- **`ui-customization-approach.md`:** Partially superseded by `htmltemplate-use.md` on layout components (native M3 Layouts are standard). M3+CSS customization approach for styling remains valid.

---

*End of file — adr-index.md*
