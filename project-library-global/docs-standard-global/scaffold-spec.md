# Mybizz CS — Scaffold Specification

**Vertical:** Consulting & Services
**Platform:** Anvil.works with Material Design 3 (M3)
**Schema:** 38 tables
**Purpose:** Exact package, form, module, and table specifications for Anvil scaffolding
**Last Updated:** 2026-06-03 v2 — Corrected component classification. Layout variants removed from components/; only true shared components remain.

**Related Documents:**
- `scaffold-workflow.md` — Step-by-step scaffolding instructions
- `authoritative-schema.md` — Complete 38-table schema specifications
- `implementation/build-plan.md` — Phased build sequence
- `gstack-outputs/DESIGN.md` — Canonical design system

---

## Reconciliation Notes (2026-06-03)

This file has been reconciled against:
- `gstack-outputs/eng-review-alignment-report.md` (2026-06-03)
- `docs/authoritative-schema.md` (current)
- `gstack-outputs/DESIGN.md` (current)
- `implementation/build-plan.md` (current)
- `gstack-outputs/design-consultation-plan-workflows.md` (current)

Key changes (v1):
- Table count updated from 36 to 38 (added `privacy_policy`, `t_and_c`, `blog_comments`)
- `business_profile` updated with 11 onboarding fields and 7 design fields
- `blog_posts` updated with 5 template/SEO fields
- Added `onboarding/` package with OnboardingForm
- Added `landing_pages/` package with LandingPageListForm and LandingPageEditorForm
- Added PolicyEditorForm to `settings/`
- Added CommentModerationForm to `blog/`
- Added BookingViewerForm to `bookings/`
- Fixed triplicated server_code block (was pasted 3 times)
- Updated `server_shared/` module list to match actual repo (11 modules)
- Added `comment_service` to `server_blog/`
- Added `export_service` and `policy_service` to `server_shared/`
- Added Build Phase annotations to all forms and tables

Key changes (v2 — component classification correction):
- **PaletteSettingsForm** removed from `components/` — not a custom component (used in SettingsForm only). Build directly into SettingsForm Palette tab.
- **12 template variants** removed from `components/` — not custom components (each used in one parent form only). Build as swappable content panels inside their parent forms:
  - `LeadCaptureTemplate`, `EventRegistrationTemplate`, `VslTemplate` → build into `LandingPage`
  - `BlogListTiledTemplate`, `BlogListListTemplate` → build into `BlogListForm`
  - `BlogPostClassicTemplate`, `BlogPostTwoColumnTemplate`, `BlogPostFullWidthTemplate` → build into `BlogPostForm`
  - `HomePageClassicTemplate`, `HomePageServicesTemplate`, `HomePageBookingTemplate`, `HomePageMinimalistTemplate` → build into `HomePage`
- `components/` package now contains 5 true shared custom components only (used in 2+ different parent forms)
- Custom Component Conventions section updated with classification criteria

---

## Overview

This document specifies the exact package structure, forms, modules, and table definitions for the `mb-3-cs` app. Follow this specification precisely when scaffolding the app.

**M3 Compliance:** All UI work uses Material Design 3 exclusively. Native M3 components from the theme dependency are fully supported. Custom composition is required for components not natively provided (Chips, Snackbar, Bottom Sheet, Tabs).

**Build Phases:** Each form and table is annotated with its build phase (Phase 1–5). Build sequentially — each phase depends on the prior phase being complete. See `implementation/build-plan.md` for full details.

**Note for Developers:** Use `scaffold-workflow.md` for step-by-step instructions. Use `authoritative-schema.md` for complete table specifications.

---

## Package Structure

### Client Code Packages

```
client_code/
├── auth/                              [Phase 1]
│   ├── LoginForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── SignupForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── PasswordResetForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── PasswordResetCompleteForm/
│       ├── form_template.yaml
│       └── __init__.py
│
├── onboarding/                        [Phase 1] ★ NEW
│   └── OnboardingForm/
│       ├── form_template.yaml
│       └── __init__.py
│
├── dashboard/                         [Phase 1]
│   ├── DashboardForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── MetricCard/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── ActivityFeed/
│       ├── form_template.yaml
│       └── __init__.py
│
├── settings/                          [Phase 1]
│   ├── SettingsForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── VaultForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── EmailSetupForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── PaymentGatewayForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── ThemeConfigForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── UserManagementForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── PolicyEditorForm/              [Phase 4] ★ NEW
│       ├── form_template.yaml
│       └── __init__.py
│
├── layouts/                           [Phase 1]
│   ├── AdminLayout/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── CustomerLayout/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── BlankLayout/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── ErrorLayout/
│       ├── form_template.yaml
│       └── __init__.py
│
├── services/                          [Phase 2]
│   ├── ServicesForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── ServiceEditorForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── ServiceViewerForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── ServiceCategoriesForm/
│       ├── form_template.yaml
│       └── __init__.py
│
├── bookings/                          [Phase 2]
│   ├── BookingCalendarForm/
│   │   ├── form_template.yaml
│   │   ├── __init__.py
│   │   ├── tmpl_month_day_cell/
│   │   │   ├── form_template.yaml
│   │   │   └── __init__.py
│   │   ├── tmpl_week_timeslot/
│   │   │   ├── form_template.yaml
│   │   │   └── __init__.py
│   │   ├── tmpl_week_event/
│   │   │   ├── form_template.yaml
│   │   │   └── __init__.py
│   │   └── tmpl_day_event/
│   │       ├── form_template.yaml
│   │       └── __init__.py
│   ├── BookingListForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── BookingEditorForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── BookingViewerForm/             [Phase 2] ★ NEW
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── AvailabilitySettingsForm/
│       ├── form_template.yaml
│       └── __init__.py
│
├── crm/                               [Phase 3]
│   ├── ContactListForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── ContactEditorForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── ContactViewerForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── SegmentManagerForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── TaskListForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── EmailCampaignListForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── EmailCampaignEditorForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── EmailBroadcastForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── LeadCaptureForm/
│       ├── form_template.yaml
│       └── __init__.py
│
├── invoices/                          [Phase 2]
│   ├── InvoiceListForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── InvoiceEditorForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── InvoiceViewerForm/
│       ├── form_template.yaml
│       └── __init__.py
│
├── landing_pages/                     [Phase 3] ★ NEW PACKAGE
│   ├── LandingPageListForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── LandingPageEditorForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── LandingPage/
│       ├── form_template.yaml
│       └── __init__.py
│
├── blog/                              [Phase 4]
│   ├── BlogListForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── BlogPostForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── BlogEditorForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── CategoryManagementForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── CommentModerationForm/         [Phase 4] ★ NEW
│       ├── form_template.yaml
│       └── __init__.py
│
├── public_pages/                      [Phase 4]
│   ├── HomePage/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── AboutPage/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── ContactPage/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── PrivacyPolicyPage/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── TermsConditionsPage/
│       ├── form_template.yaml
│       └── __init__.py
│
├── analytics/                         [Phase 5]
│   ├── DashboardAnalyticsForm/
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── RevenueReportForm/
│       ├── form_template.yaml
│       └── __init__.py
│
├── components/                        [Phase 3–4] ★ NEW PACKAGE — Shared custom components only
│   ├── ClauseBuilder/                 [Phase 4] — Shared: PolicyEditorForm (privacy_policy + t_and_c)
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── FooterComponent/               [Phase 4] — Shared: all public pages (6+)
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── VideoDisplayComponent/         [Phase 4] — Shared: BlogPostForm, HomePage
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   ├── TimeLapseCarouselComponent/    [Phase 4] — Shared: HomePage, ServicesForm
│   │   ├── form_template.yaml
│   │   └── __init__.py
│   └── ParallaxComponent/             [Phase 4] — Shared: HomePage, multiple public pages
│       ├── form_template.yaml
│       └── __init__.py
│
└── shared/
    ├── navigation_helpers.py
    ├── validation_utils.py
    └── formatting_utils.py
```

### Form Inventory Summary

| Package | Forms | Phase |
|---------|-------|-------|
| auth/ | LoginForm, SignupForm, PasswordResetForm, PasswordResetCompleteForm | 1 |
| onboarding/ | OnboardingForm ★ | 1 |
| dashboard/ | DashboardForm, MetricCard, ActivityFeed | 1 |
| settings/ | SettingsForm (includes PaletteSettingsForm built in), VaultForm, EmailSetupForm, PaymentGatewayForm, ThemeConfigForm, UserManagementForm, PolicyEditorForm ★ | 1, 4 |
| layouts/ | AdminLayout, CustomerLayout, BlankLayout, ErrorLayout | 1 |
| services/ | ServicesForm, ServiceEditorForm, ServiceViewerForm, ServiceCategoriesForm | 2 |
| bookings/ | BookingCalendarForm (+4 sub-templates), BookingListForm, BookingEditorForm, BookingViewerForm ★, AvailabilitySettingsForm | 2 |
| crm/ | ContactListForm, ContactEditorForm, ContactViewerForm, SegmentManagerForm, TaskListForm, EmailCampaignListForm, EmailCampaignEditorForm, EmailBroadcastForm, LeadCaptureForm | 3 |
| invoices/ | InvoiceListForm, InvoiceEditorForm, InvoiceViewerForm | 2 |
| landing_pages/ | LandingPageListForm ★, LandingPageEditorForm ★, LandingPage (includes 3 layout variants) | 3 |
| blog/ | BlogListForm (includes tiled/list view panels), BlogPostForm (includes 3 layout variants), BlogEditorForm, CategoryManagementForm, CommentModerationForm ★ | 4 |
| public_pages/ | HomePage (includes 4 layout variants), AboutPage, ContactPage, PrivacyPolicyPage, TermsConditionsPage | 4 |
| analytics/ | DashboardAnalyticsForm, RevenueReportForm | 5 |
| components/ | 5 shared custom components (ClauseBuilder, FooterComponent, VideoDisplayComponent, TimeLapseCarouselComponent, ParallaxComponent) ★ | 3, 4 |
| shared/ | navigation_helpers, validation_utils, formatting_utils | 1 |

**Total: 49 client forms/modules** (38 existing + 6 new forms + 5 new components)

**Layout variants built INTO parent forms** (not separate forms):
- `SettingsForm` Palette tab contains PaletteSettingsForm content
- `BlogListForm` contains tiled and list view panels
- `BlogPostForm` contains classic, two-column, and full-width layout panels
- `HomePage` contains classic, services, booking, and minimalist layout panels
- `LandingPage` contains lead capture, event registration, and VSL layout panels

★ = New form/package added in this reconciliation

---

### Server Code Packages

Note: Anvil automatically appends `.py` to server modules. Do NOT include `.py` in folder names when creating in Anvil Designer. Create folders named `server_auth`, `server_payments`, etc. — Anvil will treat them as Python modules.

```
server_code/
├── server_auth/
│   ├── service
│   └── rbac
│
├── server_dashboard/
│   └── service
│
├── server_settings/
│   ├── service
│   ├── encryption_service
│   ├── vault_service
│   └── vault_totp_service
│
├── server_services/
│   └── service
│
├── server_bookings/
│   ├── service
│   ├── availability_service
│   ├── time_service
│   └── metadata_service
│
├── server_customers/
│   ├── contact_service
│   └── segment_service
│
├── server_marketing/
│   ├── campaign_service
│   ├── broadcast_service
│   ├── task_service
│   ├── lead_capture_service
│   └── brevo_campaigns_integration
│
├── server_payments/
│   ├── stripe_service
│   ├── paystack_service
│   ├── invoice_service
│   └── webhook_handlers
│
├── server_blog/
│   ├── service
│   └── comment_service              ★ NEW
│
├── server_analytics/
│   └── reporting_service
│
└── server_shared/
    ├── config
    ├── constants
    ├── encryption_service
    ├── exceptions
    ├── logging_config
    ├── metrics
    ├── utilities
    ├── validators
    ├── vault_service
    ├── export_service               ★ NEW — GDPR/POPIA data export
    └── policy_service               ★ NEW — Policy clause CRUD
```

---

## Custom Component Conventions

### Classification Criteria

A form qualifies as a **custom component** only if it meets BOTH criteria:
1. **Used more than once** — instantiated in multiple contexts
2. **Used in more than one place** — loaded into 2+ different parent forms

If a form is used in only one parent form (even with multiple data contexts or view modes), it is a **layout variant** and must be built directly into its parent form as a swappable content panel — not as a standalone Template component.

### Shared Custom Components (5)

| Component | Parent forms | Places | Justification |
|---|---|---|---|
| ClauseBuilder | PolicyEditorForm (privacy_policy + t_and_c) | 2 data contexts, parameterised by document_type | Architecturally defined as shared (`legal-policy-responsibility-acknowledgement-and-clause-builder-architecture` ADR) |
| FooterComponent | HomePage, AboutPage, ContactPage, PrivacyPolicyPage, TermsConditionsPage, LandingPage | 6+ | Same footer on every public page |
| VideoDisplayComponent | BlogPostForm, HomePage | 2 | YouTube embed in blog posts and homepage |
| TimeLapseCarouselComponent | HomePage, ServicesForm | 2 | Auto-rotating carousel on homepage and services |
| ParallaxComponent | HomePage, multiple public pages | 2+ | Parallax scroll effect across public pages |

### Layout Variants (13) — NOT custom components, build INTO parent forms

| Variant | Parent form | Build approach |
|---|---|---|
| PaletteSettingsForm | SettingsForm (Palette tab) | Build directly into SettingsForm Palette tab as a content section |
| BlogListTiledTemplate | BlogListForm | Build as a content panel inside BlogListForm, toggled by view mode |
| BlogListListTemplate | BlogListForm | Build as a content panel inside BlogListForm, toggled by view mode |
| BlogPostClassicTemplate | BlogPostForm | Build as a content panel inside BlogPostForm, selected by `post.template` |
| BlogPostTwoColumnTemplate | BlogPostForm | Build as a content panel inside BlogPostForm, selected by `post.template` |
| BlogPostFullWidthTemplate | BlogPostForm | Build as a content panel inside BlogPostForm, selected by `post.template` |
| HomePageClassicTemplate | HomePage | Build as a content panel inside HomePage, selected by `homepage_template` |
| HomePageServicesTemplate | HomePage | Build as a content panel inside HomePage, selected by `homepage_template` |
| HomePageBookingTemplate | HomePage | Build as a content panel inside HomePage, selected by `homepage_template` |
| HomePageMinimalistTemplate | HomePage | Build as a content panel inside HomePage, selected by `homepage_template` |
| LeadCaptureTemplate | LandingPage | Build as a content panel inside LandingPage, selected by `template_type` |
| EventRegistrationTemplate | LandingPage | Build as a content panel inside LandingPage, selected by `template_type` |
| VslTemplate | LandingPage | Build as a content panel inside LandingPage, selected by `template_type` |

### Shared Component Structure

- Inherit from a base `Template` class (Anvil custom component pattern)
- Have a `form_template.yaml` defining their visual structure
- Are parameterised via constructor properties (e.g., `document_type` for ClauseBuilder)
- Are imported and used inside other forms via `from ..components import ComponentName`
- Must follow the same M3 token, typography, spacing, and role system as all other forms

### Removed Components (do not create)

| Component | Reason |
|---|---|
| OnboardingPaletteForm | Obsolete — palette selection is part of OnboardingForm |
| AnimatedButtonComponent | M3 role-based styling provides sufficient animations |
| AnimatedCardComponent | M3 role-based styling provides sufficient animations |
| AnimatedLinkComponent | M3 role-based styling provides sufficient animations |
| AnimatedImageComponent | M3 role-based styling provides sufficient animations |

---

## Database Schema

Create the following 38 tables in Anvil Data Tables with the exact specifications below.

Tables marked ★ are new additions in this reconciliation. Tables marked with phase annotations indicate when they are first needed.

### Core & Configuration Tables (10)

#### 1. activity_log [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `user_id` | link_single → users | User who performed action |
| `action_type` | string | e.g., "login", "booking_created" |
| `description` | string | Human-readable description |
| `metadata` | simpleObject | Additional context |
| `created_at` | datetime | UTC |
| `ip_address` | string | Client IP |

#### 2. business_profile [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `business_name` | string | |
| `description` | string | |
| `logo` | media | |
| `contact_email` | string | |
| `phone` | string | |
| `website_url` | string | |
| `tagline` | string | |
| `address_line_1` | string | |
| `address_line_2` | string | |
| `city` | string | |
| `country` | string | |
| `postal_code` | string | |
| `timezone` | string | IANA timezone (e.g., "Africa/Johannesburg") |
| `system_currency` | string | ISO 4217 — set at onboarding, immutable after first transaction |
| `social_facebook` | string | |
| `social_instagram` | string | |
| `social_x` | string | X (formerly Twitter) |
| `social_linkedin` | string | |
| `created_at` | datetime | UTC |
| `updated_at` | datetime | UTC |
| **Onboarding fields:** | | ★ NEW |
| `onboarding_status` | string | "in_progress" or "completed" |
| `onboarding_completed` | bool | Overall completion flag |
| `onboarding_completed_at` | datetime | UTC — when onboarding completed |
| `entity_type` | string | "Individual" or "Business" |
| `palette` | string | Selected brand palette identifier |
| `legal_compliance_enabled` | bool | Legal Compliances switch — irreversible once True |
| `legal_compliance_enabled_at` | datetime | UTC — when switch was enabled |
| `legal_ack_accepted` | bool | Legal responsibility acknowledgement |
| `legal_ack_accepted_at` | datetime | UTC — when acknowledgement recorded |
| `data_governance_ack_accepted` | bool | Data governance boundary acknowledgement |
| `data_governance_ack_accepted_at` | datetime | UTC — when acknowledgement recorded |
| **Design fields:** | | ★ NEW |
| `palette_updated_at` | datetime | When palette was last changed |
| `blog_list_view` | string | "tiled" or "list" — default blog list view |
| `enable_comments` | bool | Blog comments toggle |
| `homepage_template` | string | "classic", "services", "booking", "minimalist" |
| `business_logo` | media | Alternative logo for public pages |
| `accreditations` | string | Business accreditations text |
| `google_maps_embed_url` | string | Google Maps embed URL |
| `home_video_url` | string | YouTube URL for homepage video display |
| `home_video_type` | string | Video source type (V1: "youtube") |
| `services_video_url` | string | YouTube URL for services page video display |
| `services_video_type` | string | Video source type (V1: "youtube") |

#### 3. config [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `key` | string | Indexed, unique |
| `value` | simpleObject | JSON value |
| `category` | string | e.g., "features", "settings" |
| `updated_at` | datetime | UTC |
| `updated_by` | link_single → users | |

#### 4. email_config [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `from_email` | string | |
| `from_name` | string | |
| `configured` | bool | |
| `configured_at` | datetime | UTC |
| `sender_domain_verified` | bool | |

#### 5. files [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `path` | string | |
| `file` | media | |
| `file_version` | string | |

#### 6. payment_config [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `active_gateway` | string | "stripe" or "paystack" |
| `stripe_publishable_key` | string | |
| `stripe_connected` | bool | |
| `stripe_connected_at` | datetime | UTC |
| `paystack_connected` | bool | |
| `paystack_connected_at` | datetime | UTC |
| `test_mode` | bool | |
| `configured_at` | datetime | UTC |

**Note:** Secret keys stored in `vault` table only. PayPal deferred to V2.

#### 7. rate_limits [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `identifier` | string | Indexed (IP or user ID) |
| `count` | number | Request count |
| `reset_time` | datetime | When counter resets |
| `last_request` | datetime | UTC |

#### 8. theme_config [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `primary_color` | string | Hex color |
| `accent_color` | string | Hex color |
| `font_family` | string | |
| `header_style` | string | |
| `updated_at` | datetime | UTC |

#### 9. users (Extended Anvil Users) [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `email` | string | Indexed, unique |
| `enabled` | bool | |
| `last_login` | datetime | UTC |
| `password_hash` | string | Anvil managed |
| `n_password_failures` | number | Anvil managed |
| `confirmed_email` | bool | |
| `role` | string | "owner", "manager", "admin", "staff", "customer" |
| `account_status` | string | "active", "suspended", "deleted" |
| `phone` | string | |
| `permissions` | simpleObject | JSON permission flags |
| `created_at` | datetime | UTC |
| `signed_up` | datetime | UTC |
| `email_confirmation_key` | string | |
| `mfa` | simpleObject | TOTP configuration |

#### 10. vault [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `name` | string | Indexed, unique |
| `value` | string | Encrypted |
| `created_at` | datetime | UTC |
| `updated_at` | datetime | UTC |
| `updated_by` | link_single → users | |

---

### Bookings Tables (6)

#### 11. availability_exceptions [Phase 2]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `resource_id` | link_single → users | |
| `exception_date` | date | |
| `start_time` | string | |
| `end_time` | string | |
| `is_available` | bool | |
| `reason` | string | |

#### 12. availability_rules [Phase 2]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `resource_id` | link_single → users | |
| `day_of_week` | number | |
| `start_time` | string | |
| `end_time` | string | |
| `is_active` | bool | |

#### 13. booking_metadata_schemas [Phase 2]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `schema_name` | string | |
| `booking_type` | string | |
| `field_definitions` | simpleObject | |
| `is_active` | bool | |

#### 14. bookings [Phase 2]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `booking_id` | string | |
| `contact_id` | link_single → contacts | |
| `service_id` | link_single → services | |
| `resource_id` | link_single → users | |
| `booking_type` | string | |
| `status` | string | |
| `start_datetime` | datetime | UTC |
| `end_datetime` | datetime | UTC |
| `total_price` | number | |
| `payment_status` | string | |
| `payment_id` | string | |
| `special_requests` | string | |
| `metadata` | simpleObject | |
| `booking_number` | string | |
| `created_at` | datetime | UTC |
| `updated_at` | datetime | UTC |

#### 15. services [Phase 2]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `service_id` | string | |
| `name` | string | |
| `description` | string | |
| `duration_minutes` | number | |
| `price` | number | |
| `category` | string | |
| `provider_id` | link_single → users | |
| `meeting_type` | string | |
| `pricing_model` | string | "duration" or "unit" |
| `images` | simpleObject | |
| `video_url` | string | |
| `video_type` | string | Video source type (V1: "youtube"; V2: "vimeo", "dailymotion", "custom") |
| `is_active` | bool | |
| `created_at` | datetime | UTC |

#### 16. contact_submissions [Phase 1]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `name` | string | |
| `email` | string | |
| `phone` | string | |
| `message` | string | |
| `submitted_date` | datetime | UTC |
| `status` | string | |
| `client_id` | link_single → users | |
| `submission_id` | string | |
| `auto_reply_sent` | bool | |

---

### CRM Tables (9)

#### 17. contacts [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `contact_id` | string | |
| `first_name` | string | |
| `last_name` | string | |
| `email` | string | |
| `phone` | string | |
| `status` | string | |
| `source` | string | |
| `lifecycle_stage` | string | |
| `tags` | simpleObject | |
| `total_spent` | number | |
| `total_transactions` | number | |
| `average_order_value` | number | |
| `last_contact_date` | datetime | UTC |
| `date_added` | datetime | UTC |
| `created_at` | datetime | UTC |
| `updated_at` | datetime | UTC |
| `internal_notes` | string | |
| `preferences` | simpleObject | |

#### 18. contact_campaigns [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `id` | string | |
| `contact_id` | link_single → contacts | |
| `campaign_id` | link_single → email_campaigns | |
| `sequence_day` | number | |
| `status` | string | |
| `enrolled_date` | datetime | UTC |
| `last_email_sent_date` | datetime | UTC |
| `completed_date` | datetime | UTC |

#### 19. contact_counter [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `value` | number | |

#### 20. contact_events [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `event_id` | string | |
| `contact_id` | link_single → contacts | |
| `event_type` | string | |
| `event_date` | datetime | UTC |
| `event_data` | simpleObject | |
| `related_id` | string | |
| `user_visible` | bool | |

#### 21. email_campaigns [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `campaign_id` | string | |
| `campaign_name` | string | |
| `campaign_type` | string | |
| `status` | string | |
| `emails_sent` | number | |
| `opens` | number | |
| `clicks` | number | |
| `conversions` | number | |
| `revenue_generated` | number | |
| `campaign_settings` | simpleObject | |
| `created_date` | datetime | UTC |
| `last_run_date` | datetime | UTC |

#### 22. segments [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `segment_id` | string | |
| `segment_name` | string | |
| `segment_type` | string | |
| `filter_criteria` | simpleObject | |
| `contact_count` | number | |
| `is_active` | bool | |
| `created_date` | datetime | UTC |

#### 23. tasks [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `task_id` | string | |
| `contact_id` | link_single → contacts | |
| `task_title` | string | |
| `task_type` | string | |
| `due_date` | datetime | UTC |
| `completed` | bool | |
| `completed_date` | datetime | UTC |
| `notes` | string | |
| `auto_generated` | bool | |
| `created_at` | datetime | UTC |

#### 24. leads [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `lead_id` | string | |
| `email` | string | |
| `name` | string | |
| `phone` | string | |
| `source` | string | |
| `landing_page_id` | link_single → landing_pages | |
| `capture_id` | link_single → lead_captures | |
| `converted_to_contact_id` | link_single → contacts | |
| `status` | string | |
| `captured_at` | datetime | UTC |

#### 25. lead_captures [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `capture_id` | string | |
| `capture_name` | string | |
| `capture_type` | string | |
| `trigger_settings` | simpleObject | |
| `form_fields` | simpleObject | |
| `offer_text` | string | |
| `status` | string | |
| `captures_count` | number | |
| `conversions_count` | number | |

---

### Email Tables (3)

#### 26. email_templates [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `template_id` | string | |
| `client_id` | link_single → users | |
| `name` | string | |
| `subject` | string | |
| `body_html` | string | |
| `body_text` | string | |
| `template_type` | string | |
| `is_active` | bool | |

#### 27. email_log [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `recipient` | string | |
| `subject` | string | |
| `template_id` | link_single → email_templates | |
| `sent_at` | datetime | UTC |
| `status` | string | |
| `error_message` | string | |

#### 28. webhook_log [Phase 2]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `gateway` | string | |
| `event_type` | string | |
| `payload` | simpleObject | |
| `signature` | string | |
| `verified` | bool | |
| `processed` | bool | |
| `error_message` | string | |
| `received_at` | datetime | UTC |

---

### Content Tables (4)

#### 29. blog_categories [Phase 4]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `category_id` | string | |
| `client_id` | link_single → users | |
| `name` | string | |
| `slug` | string | |
| `description` | string | |

#### 30. blog_posts [Phase 4]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `post_id` | string | |
| `client_id` | link_single → users | |
| `title` | string | |
| `slug` | string | |
| `content` | string | |
| `excerpt` | string | |
| `featured_image` | media | |
| `author_id` | link_single → users | |
| `category_id` | link_single → blog_categories | |
| `status` | string | "draft", "published", "archived" |
| `published_at` | datetime | UTC |
| `created_at` | datetime | UTC |
| `updated_at` | datetime | UTC |
| `view_count` | number | |
| **Template/SEO fields:** | | ★ NEW |
| `template` | string | "classic", "two_column", "full_width" |
| `meta_title` | string | SEO title override |
| `meta_description` | string | SEO meta description |
| `keywords` | string | SEO keywords |
| `pullquote` | string | Featured pullquote for full-width template |
| `video_url` | string | YouTube URL for embedded video (V1: YouTube only) |
| `video_type` | string | Video source type (V1: "youtube"; V2: "vimeo", "dailymotion", "custom") |

#### 31. landing_pages [Phase 3]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `page_id` | string | |
| `title` | string | |
| `slug` | string | |
| `template_type` | string | "lead_capture", "event_registration", "vsl" |
| `config` | simpleObject | |
| `status` | string | "draft", "published" |
| `created_at` | datetime | UTC |
| `published_at` | datetime | UTC |
| `views_count` | number | |
| `conversions_count` | number | |
| `updated_at` | datetime | UTC |
| `client_id` | link_single → users | |

#### 32. pages [Phase 4]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `name` | string | |
| `slug` | string | |
| `components` | simpleObject | |
| `is_published` | bool | |
| `view_count` | number | |
| `created_at` | datetime | UTC |
| `updated_at` | datetime | UTC |
| `page_name` | string | |
| `page_title` | string | |

---

### Policy Tables (2) ★ NEW

#### 33. privacy_policy [Phase 4] ★

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `category` | string | e.g., "Data Collection", "Cookies", "Third Parties" |
| `title` | string | Clause heading |
| `clause` | string | Body text / rich text |
| `sort_order` | number | Display order within category |
| `is_seeded` | bool | True if from master template |
| `is_active` | bool | Whether clause is published |
| `created_at` | datetime | UTC |
| `updated_at` | datetime | UTC |

#### 34. t_and_c [Phase 4] ★

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `category` | string | e.g., "Service Terms", "Payment Terms", "Liability" |
| `title` | string | Clause heading |
| `clause` | string | Body text / rich text |
| `sort_order` | number | Display order within category |
| `is_seeded` | bool | True if from master template |
| `is_active` | bool | Whether clause is published |
| `created_at` | datetime | UTC |
| `updated_at` | datetime | UTC |

#### 35. blog_comments [Phase 4] ★

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `post_id` | link_single → blog_posts | Parent post |
| `commenter_name` | string | Required |
| `commenter_email` | string | Required |
| `comment_text` | string | Required |
| `status` | string | "pending", "approved", "spam", "deleted" |
| `created_at` | datetime | UTC |
| `updated_at` | datetime | UTC |

---

### Finance Tables (3)

#### 36. invoice [Phase 2]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `invoice_id` | string | |
| `invoice_number` | string | Human-readable (INV-2026-001) |
| `contact_id` | link_single → contacts | |
| `invoice_date` | datetime | UTC |
| `due_date` | datetime | UTC |
| `total_amount` | number | |
| `currency` | string | ISO 4217 |
| `paid_at` | datetime | UTC |
| `status` | string | "draft", "sent", "paid", "overdue", "cancelled" |
| `payment_method` | string | "stripe", "paystack" |
| `payment_id` | string | Gateway transaction ID |
| `pdf_url` | string | |
| `notes` | string | |
| `created_at` | datetime | UTC |

#### 37. invoice_items [Phase 2]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `invoice_id` | link_single → invoice | |
| `description` | string | |
| `quantity` | number | |
| `unit_price` | number | |
| `amount` | number | quantity x unit_price |

#### 38. time_entries [Phase 2 — deferred to V2]

| Column | Anvil Type | Notes |
|--------|-----------|-------|
| `entry_id` | string | |
| `contact_id` | link_single → contacts | |
| `staff_id` | link_single → users | |
| `service_id` | link_single → services | |
| `booking_id` | link_single → bookings | Optional |
| `start_time` | datetime | UTC |
| `end_time` | datetime | UTC |
| `duration_hours` | number | Calculated |
| `hourly_rate` | number | |
| `total_amount` | number | duration x hourly_rate |
| `description` | string | |
| `is_billable` | bool | |
| `is_invoiced` | bool | |
| `created_at` | datetime | UTC |

---

## M3 Component Standards

### Form Types

**List Forms:**
- Heading (headline-large) for page title
- FlowPanel (horizontal) header with search TextBox and filter DropdownMenu
- Filled Button for primary action
- Outlined Button for secondary action (e.g., "Calendar View" in BookingListForm)
- DataGrid for list body with IconButton row actions

**Editor Forms:**
- Card (outlined) as container
- Heading (headline-small) for section headers
- Outlined TextBox/TextArea for inputs
- Outlined DropdownMenu/DatePicker for selections
- FlowPanel action row with Filled Save Button and Outlined Cancel Button

**Dashboard Forms:**
- Heading (headline-large)
- FlowPanel for metrics row holding elevated Cards
- Plot components for charts
- DataGrid for summary tables

**Authentication Forms:**
- Custom Form with no layout wrapper
- Card (outlined) as centred container
- Outlined TextBox components (password type set in Designer)
- Filled Button for submit
- Link components for forgot password and terms/privacy navigation

### Component Naming

All components follow the prefix convention:
- Button: `btn_`
- TextBox: `txt_`
- Heading: `hdg_`
- DropdownMenu: `dd_`
- DatePicker: `dp_`
- FileLoader: `fu_`
- Link/Navigation: `nav_`
- Checkbox: `cb_`
- RadioButton: `rb_`
- ColumnPanel: `col_`
- FlowPanel: `flow_`
- Card: `card_`
- DataGrid: `dg_`
- RepeatingPanel: `rp_`
- Plot: `plot_`

### M3 Requirements

- All colours use `theme:` prefix — never hardcoded hex
- All inputs use `role='outlined'`
- Validation failure: `role='outlined-error'` with error in placeholder
- Button hierarchy: one filled button per screen (primary), outlined (secondary), text (tertiary)
- TextBox password masking: `hide_text = True` in Designer
- Write Back toggle (W): enabled in Designer Data Bindings panel

**Native M3 components** (fully supported): `m3.Switch`, `m3.Slider`, `m3.RadioButton`, `m3.Checkbox`, `m3.Button`, `m3.IconButton`, `m3.NavigationLink`, all input and layout components.

**Custom composition required** (partially supported): Chips, Snackbar, Bottom Sheet, Tabs, Date Picker. See `m3_component_mapping.md` for implementation guidance.

---

## Navigation Architecture

### AdminLayout (NavigationRailLayout)

```
Dashboard                          (always)
▼ Sales & Operations
  nav_bookings  — BookingListForm   (bookings_enabled)
  nav_services  — ServicesForm   (services_enabled)
▼ Customers & Marketing
  nav_contacts   — ContactListForm       (always)
  nav_campaigns  — EmailCampaignListForm (marketing_enabled)
  nav_broadcasts — EmailBroadcastForm    (marketing_enabled)
  nav_segments   — SegmentManagerForm    (marketing_enabled)
  nav_tasks      — TaskListForm          (marketing_enabled)
▼ Content & Website
  nav_blog   — BlogListForm     (blog_enabled)
  nav_pages  — PageListForm     (always)
▼ Finance & Reports
  nav_invoices  — InvoiceListForm   (always)
  nav_payments  — PaymentListForm   (always)
  nav_reports   — ReportsForm       (always)
  nav_analytics — AnalyticsForm     (always)
  nav_time      — TimeEntriesForm   (always)
▼ Settings
  nav_settings  — SettingsForm (always)
  nav_vault     — VaultForm    (owner role only)
[Sign out — bottom of sidebar, all roles]
```

**Navigation Pattern:**
- Plain `Link` components — not `NavigationLink`
- Lambda click handlers with mandatory loop variable capture:
  ```python
  lambda e, fn=form_name, a=attr_name: self._nav_click(fn, a)
  ```
- `open_form(string)` for parameterised navigation
- `NavigationDrawerLayoutTemplate` not used in current architecture
- `navigate_to` not used

**Note:** Native M3 navigation components are fully supported by the M3 theme dependency but not used in the current Mybizz architecture. See `2-navigation-lambda-link-open-form.md`.

### CustomerLayout (Deferred to V2)

```
nav_my_dashboard  — My Dashboard    (always)
nav_my_bookings   — My Bookings     (always)
nav_my_invoices   — My Invoices     (always)
nav_my_reviews    — My Reviews      (always)
nav_support       — Support Tickets (always)
nav_account       — Account         (always)
[Sign out — bottom of sidebar]
```

---

## Startup Configuration

**Startup Form:** `auth.LoginForm`

**Routing:** Anvil Routing dependency (`3PIDO5P3H4VPEMPL`) configured for public pages:
- `/` — HomePage
- `/services` — Services listing
- `/services/:id` — Service detail
- `/booking` — Booking form
- `/blog` — Blog listing
- `/blog/:slug` — Blog post
- `/contact` — Contact page
- `/privacy` — PrivacyPolicyPage
- `/terms` — TermsConditionsPage
- `/landing/:slug` — Landing page

---

## Dependencies

**Installed Dependencies:**
1. M3 Theme: `4UK6WHQ6UX7AKELK`
2. Routing: `3PIDO5P3H4VPEMPL`

**Set in Anvil Secrets:**
- `encryption_key` — Fernet encryption key (generate once, never change)

---

## Implementation Notes

### Form File Structure
- Forms are folders: `FormName/`
- Code lives in `FormName/__init__.py`
- Designer config in `FormName/form_template.yaml`
- Never modify `form_template.yaml` programmatically

### Data Binding Pattern
- `self.item` set before `self.init_components()`
- **Write Back toggle (W) must be enabled in Designer Data Bindings panel** — cannot be set in code
- Call `refresh_data_bindings()` when modifying `self.item` in place
- Do NOT use data bindings on settings forms that manage their own server calls
- Applies to: TextBox, TextArea, DropdownMenu, DatePicker, and all data-bound inputs

### Server Function Pattern
- Every server function returns response envelope:
  ```python
  {'success': True, 'data': result}   # on success
  {'success': False, 'error': msg}    # on failure
  ```
- Every server function has docstring with Args, Returns, Raises
- Type hints required on all public functions
- Logging via `logging.getLogger(__name__)`
- Authentication check at entry: `user = anvil.users.get_user()`

---

*End of file — scaffold-spec.md (Reconciled 2026-06-03)*
