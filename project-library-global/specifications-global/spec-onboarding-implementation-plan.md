# Mybizz CS Onboarding System - Engineering Plan

> **Document status:** Rewritten 2026-06-01 per plan-tune adjustments. Key design decisions:
> - **Onboarding is a separate form** that links to Settings for specific tasks
> - **Settings is a single tabbed form** using M3 buttons, RBAC-governed
> - **Bidirectional navigation** between Onboarding and Settings
> - **Mybizz_management amendment log** — append-only credential change history
> - **Client data rights** — Owner may remove details from Client Details Table, but not from Retained Records Table
> - All ADR references use full file names (e.g., `1-brevo-replaces-zoho-email.md`)

---

## Executive Summary

This plan defines the onboarding system for new Client instances in Mybizz CS. The system distinguishes between **Mybizz_management provisioning** (Mybizz internal setup) and **Owner onboarding** (in-app guided setup).

**The core architecture:**
- **Onboarding form** — a guided first-run experience with required elements (currency selection, legal acknowledgement)
- **Settings form** — a single tabbed form for all ongoing configuration, RBAC-governed
- **Bidirectional navigation** — Onboarding has buttons to jump to specific Settings tabs; Settings has a "Return to Onboarding" button
- **Mybizz_management** — maintains append-only amendment log of credential changes, with clear data rights (Owner-removable vs legally-retained)

Two distinct flows:
- Mybizz_management provisioning (Mybizz internal app)
- Owner onboarding (client instance)

Authentication and user signup flows are governed by the Authentication & Administration design, not this document.

Onboarding is **resumable and revisitable**. The Owner may return to the onboarding page at any time to review or change any credential. **[Confirmed — 25-onboarding-finality.md]**

---

## 1. Onboarding Architecture

### 1.1 Three Application Surfaces

| Surface | Purpose | Location | RBAC |
|---------|---------|----------|------|
| **Onboarding Form** | Guided first-run with required elements | Client instance (master_template) | Owner |
| **Settings Form** | Single tabbed form for all configuration | Client instance (master_template) | Owner, Manager (varies by tab) |
| **Mybizz_management** | Platform operations, amendment log, data rights | Separate Anvil app | Mybizz internal |

### 1.2 Key Architectural Principles

1. **Onboarding is a separate form** — not embedded in Settings, but links to Settings for specific tasks **[New]**
2. **Settings is a single tabbed form** — uses M3 buttons for tab navigation, RBAC-governed **[New]**
3. **Bidirectional navigation** — Onboarding → Settings (for specific tabs) and Settings → Onboarding (return button) **[New]**
4. **Onboarding is resumable** — Owner may leave and return without losing progress **[23-onboarding-resumability.md]**
5. **Credential changes are append-only** — Mybizz_management never overwrites, always appends **[25-onboarding-finality.md]**
6. **Client data rights** — Owner may remove details from Client Details Table, but not from Retained Records Table **[25-onboarding-finality.md]**
7. **System Currency is immutable after first transaction** — selected in onboarding, locked after first transaction **[16-system-currency-selection-and-immutability.md]**
8. **Payment gateway is configured in Settings** — RBAC-governed, not part of onboarding **[19-payment-gateway-configuration-is-a-settings-function-and-is-rbac-governed.md]**
9. **Palette selection is in Settings** — Onboarding has a button to jump to Settings → Palette tab **[17-onboarding-vs-settings-boundary.md]**
10. **Table and field names match `authoritative-schema.md`** — `business_profile` (with underscore) **[20-onboarding-data-schema-alignment.md]**

---

## 2. Mybizz_management Provisioning Flow

### 2.1 Purpose
Create a new Client instance shell with minimal configuration, then notify the Owner to complete onboarding.

### 2.2 Steps

| Step | Action | Data Created | Output |
|------|--------|--------------|--------|
| 1 | Create new Anvil instance (copy from master_template) | New app ID, new database | Instance ready |
| 2 | Create Owner user account | `users` row with Owner role | Owner credentials |
| 3 | Create empty `business_profile` row | `business_profile` row | Profile exists |
| 4 | Set onboarding status | `business_profile.onboarding_status = "in_progress"` | State initialised |
| 5 | Create client record in Mybizz_management | Client Details Table entry | Record exists |
| 6 | Send welcome email with login credentials | Email via Brevo | Owner notified |

> **Note:** Currency and payment gateway are NOT set by Mybizz during provisioning. The Owner configures these in Onboarding and Settings respectively. **[16-system-currency-selection-and-immutability.md, 19-payment-gateway-configuration-is-a-settings-function-and-is-rbac-governed.md]**

### 2.3 Mybizz_management Behaviour

Mybizz_management maintains three data tables for client records:

1. **Client Details Table (Owner-Modifiable):**
   - Contains client details that the Owner has the right to change at any time
   - The Owner may remove entries from this table
   - Updates append new values; old values are retained in the amendment log

2. **Retained Records Table (Legal/Financial):**
   - Contains client details retained by Mybizz for legal or financial purposes
   - The Owner has no right to remove these
   - Held separately and immutable from the Owner's perspective

3. **Amendment Log (Append-Only):**
   - Full history of all credential and configuration changes
   - Never overwritten — always holds the latest values plus the full history
   - Outside the client instance scope but a system-level behaviour

> **Note:** Mybizz_management is outside the scope of this project. The data architecture intention is recorded in `docs/platform-overview.md`.

---

## 3. Owner Onboarding (Onboarding Form)

### 3.1 Purpose
Guide the Owner through required setup elements, with links to Settings for specific configuration tasks.

### 3.2 Onboarding Form Structure

The onboarding form is a separate form from Settings. It contains:

- **Welcome section** — orientation and progress indicator
- **Required elements** — currency selection and legal acknowledgement (one-time, irreversible)
- **Navigation buttons** — links to specific Settings tabs for additional configuration

### 3.3 Onboarding Sequence

| Step | Element | Purpose | Type |
|------|---------|---------|------|
| 1 | Welcome | Brief welcome and orientation | Display only |
| 2 | Business Profile | Core business information | Link to Settings → Business Profile tab |
| 3 | Palette Selection | Brand colour selection | Link to Settings → Palette tab |
| 4 | System Currency | Currency for internal operations | **Required element** (dropdown + confirm) |
| 5 | Legal Acknowledgement | Legal responsibility acknowledgement | **Required element** (text + checkbox + confirm) |
| 6 | Completion | Summary and next steps | Display only |

### 3.4 Required Elements

These are one-time elements that must be completed on the onboarding form. They cannot be deferred.

#### 3.4.1 System Currency Selection

**Purpose:** Select the currency for all internal operations (accounting, reporting, analytics).

**UI Components:**
- `Heading` (headline-small): "System Currency"
- `Text` (body-medium): "Select the currency your business operates in. This will be used for all internal calculations, reporting, and analytics."
- `Text` (caption, warning): "This selection becomes permanent once your first transaction has been recorded."
- `DropdownMenu` — System Currency (required, list of supported ISO 4217 currencies)
- `Button` (filled): "Confirm Currency Selection"

**Inputs:**
- `system_currency` (required, valid ISO 4217 currency code)

**Behaviour:**
- Dropdown shows currency options (e.g., USD, ZAR, GBP, EUR, etc.)
- Owner selects a currency from the dropdown
- Owner clicks "Confirm Currency Selection" button
- Confirmation is recorded: `business_profile.system_currency = selected_value`
- **Once confirmed AND the first transaction is recorded, this selection becomes immutable**
- Before the first transaction, the Owner may change the selection

**Outputs:**
- Updates `business_profile.system_currency`
- Logs confirmation event in client-instance audit log (sync to Mybizz_management amendment log when Mybizz_management is built — out of scope for current project)

#### 3.4.2 Legal Acknowledgement

**Purpose:** Owner acknowledges legal responsibility for Privacy Policy and Terms & Conditions content, and understands the data governance boundary.

**UI Components:**
- `Heading` (headline-small): "Legal Acknowledgement"
- **Data governance section:**
  - `Text` (body-medium): "Mybizz retains certain records (Client identity, transaction records between Client and Mybizz) in Mybizz_management for legal and compliance purposes. Your operational data (bookings, customers, services, etc.) is held in your own instance database."
  - `Text` (body-medium): "You can download a copy of the data Mybizz holds about you at any time from your Dashboard. Go to Dashboard → Account → Download My Mybizz Data."
- **Legal responsibility section:**
  - `Text` (body-medium, critical excerpts): "You are responsible for configuring your own Privacy Policy and Terms & Conditions. Mybizz does not provide legal advice and does not provide these documents."
  - `Link` (body-medium): "Read more about Legal Compliance →" (links to Legal Compliance page)
  - `CheckBox` — `legal_ack_checkbox`: "I understand I must configure my own Privacy Policy and Terms & Conditions in Settings → Website. Mybizz is not responsible for the content of those pages."
- `Button` (filled): "Confirm Acknowledgement" (enabled only when checkbox ticked)

**Inputs:**
- `legal_ack_checkbox` (required, must be True)

**Behaviour:**
- Owner reads the critical excerpts
- Owner may click "Read more" to visit the Legal Compliance page for full details
- Owner ticks the checkbox
- Owner clicks "Confirm Acknowledgement" button
- Confirmation is logged in Mybizz_management amendment log
- **This acknowledgement is recorded once and cannot be reversed**

**Outputs:**
- Updates `business_profile.legal_ack_accepted: True`
- Updates `business_profile.legal_ack_accepted_at: datetime.now()`
- Updates `business_profile.data_governance_ack_accepted: True`
- Updates `business_profile.data_governance_ack_accepted_at: datetime.now()`
- Logs acknowledgement event in client-instance audit log (sync to Mybizz_management amendment log when Mybizz_management is built — out of scope for current project)

### 3.5 Navigation to Settings

The onboarding form includes buttons that link to specific Settings tabs:

| Onboarding Section | Settings Tab Button | Purpose |
|-------------------|---------------------|---------|
| Business Profile | "Configure in Settings →" | Jump to Settings → Business Profile tab |
| Palette Selection | "Choose Palette in Settings →" | Jump to Settings → Palette tab |
| Completion | "Configure Gateway in Settings →" | Jump to Settings → Payment Gateway tab (post-onboarding) |

> **Note:** These are convenience navigation affordances within the onboarding form, not required onboarding steps. Payment Gateway configuration is a post-onboarding Settings concern (see `payment-gateway-configuration-is-a-settings-function-and-is-rbac-governed` ADR). The gateway link appears in the completion step as a suggested next action.

**Navigation behaviour:**
- Clicking the button opens the Settings form with the relevant tab active
- Settings form displays a "Return to Onboarding" button in the header
- Clicking "Return to Onboarding" returns the Owner to the onboarding form with progress preserved

### 3.6 Onboarding Completion

Onboarding is complete when:
1. System Currency has been confirmed
2. Legal Acknowledgement has been confirmed

After completion:
- `business_profile.onboarding_status = "completed"`
- `business_profile.onboarding_completed = True`
- `business_profile.onboarding_completed_at = datetime.now()`
- The Owner is directed to the Dashboard
- The onboarding form remains accessible for credential review and updates

---

## 4. Settings Form Structure

### 4.1 Purpose
Single tabbed form for all ongoing configuration. RBAC-governed.

### 4.2 Tab Structure

The Settings form uses M3 buttons for tab navigation:

| Tab | Content | RBAC |
|-----|---------|------|
| **Business Profile** | Name, email, phone, address, timezone, entity type | Owner, Manager |
| **Palette** | Brand colour selection | Owner, Manager |
| **System Currency** | Currency display (locked after first transaction) | Owner only |
| **Legal Acknowledgement** | Read-only view of acknowledgements | Owner only |
| **Display Currency** | Optional per-service currency override | Owner, Manager |
| **Payment Gateway** | Stripe/Paystack configuration (API keys stored in The Vault) | Owner only |
| **Services** | Service catalogue management | Owner, Manager, Admin |
| **Users & Permissions** | User management and RBAC | Owner only |
| **Email Configuration** | Brevo SMTP and API settings | Owner only |
| **Theme** | M3 theme customisation | Owner, Manager |
| **Feature Flags** | Feature activation toggles | Owner only |

### 4.3 Return to Onboarding Button

The Settings form includes a "Return to Onboarding" button in the header:

- Visible only when `business_profile.onboarding_status = "in_progress"`
- Hidden after onboarding is completed
- Clicking the button returns the Owner to the onboarding form with progress preserved

### 4.4 RBAC Governance

Each tab is RBAC-governed:
- Owner: Full access to all tabs
- Manager: Access to Business Profile, Palette, Display Currency, Services, Theme
- Admin: Access to Services only
- Staff: No access to Settings
- Customer: No access to Settings

---

## 5. Data State Transitions

### 5.1 Initial state (after Mybizz_management provisioning)

```
business_profile:
  - onboarding_status: "in_progress"
  - onboarding_completed: False
  - onboarding_completed_at: null
  - legal_ack_accepted: False
  - legal_ack_accepted_at: null
  - data_governance_ack_accepted: False
  - data_governance_ack_accepted_at: null
```

### 5.2 After onboarding completion

```
business_profile:
  - onboarding_status: "completed"
  - onboarding_completed: True
  - onboarding_completed_at: 2026-XX-XX XX:XX:XX
  - system_currency: "USD"
  - legal_ack_accepted: True
  - legal_ack_accepted_at: 2026-XX-XX XX:XX:XX
  - data_governance_ack_accepted: True
  - data_governance_ack_accepted_at: 2026-XX-XX XX:XX:XX
```

---

## 6. Failure Modes and Recovery

### 6.1 Partial Completion Handling

| Scenario | Behaviour | Recovery |
|----------|-----------|----------|
| Owner exits mid-onboarding | `onboarding_status = "in_progress"`, data saved | Owner re-enters onboarding on next login |
| Owner completes onboarding | `onboarding_status = "completed"` | Owner can still return to onboarding to change credentials |
| Owner changes credentials post-completion | Append-only log in Mybizz_management | New values appended, old values retained |

### 6.2 Re-entry Rules

- **Before completion:** Owner can re-enter onboarding at any time; form state is preserved
- **After completion:** Owner can still return to onboarding to review or change credentials
- **System Currency:** Cannot be changed after the first transaction; any exception requires Mybizz support **[16-system-currency-selection-and-immutability.md]**
- **Legal Acknowledgement:** Recorded once, cannot be reversed **[25-onboarding-finality.md]**
- **Payment Gateway:** Can be changed under approved platform rules **[24-payment-gateway-mutability.md]**

---

## 7. Data Table Changes Required

> **Note:** All table and field names match `authoritative-schema.md`. `business_profile` (with underscore) is used throughout. **[20-onboarding-data-schema-alignment.md]**

### 7.1 business_profile Table Additions

| Column | Type | Required | Default | Notes |
|--------|------|----------|---------|-------|
| `onboarding_status` | Text | Yes | `"in_progress"` | `"in_progress"` or `"completed"` |
| `onboarding_completed` | True/False | Yes | False | Overall completion flag |
| `onboarding_completed_at` | Date/Time | No | null | When onboarding completed |
| `entity_type` | Text | No | null | `Individual` or `Business` |
| `palette` | Text | No | null | Selected brand palette identifier |
| `legal_ack_accepted` | True/False | Yes | False | Legal responsibility acknowledgement |
| `legal_ack_accepted_at` | Date/Time | No | null | When acknowledgement recorded |
| `data_governance_ack_accepted` | True/False | Yes | False | Data governance boundary acknowledgement |
| `data_governance_ack_accepted_at` | Date/Time | No | null | When acknowledgement recorded |

---

## 8. Validation Rules Summary

### 8.1 Required Validations (Onboarding Form)

| Element | Field | Validation |
|---------|-------|------------|
| System Currency | `system_currency` | Required, valid ISO 4217 currency code, must click Confirm |
| Legal Acknowledgement | `legal_ack_checkbox` | Required, must be ticked, must click Confirm |

### 8.2 Required Validations (Settings Form — Business Profile Tab)

| Field | Validation |
|-------|------------|
| `business_name` | Required, 2–100 chars |
| `contact_email` | Required, valid email |
| `phone` | Required, E.164 format |
| `timezone` | Required, valid IANA timezone |

---

## 9. Canonical V1 Onboarding Flow

### 9.1 Summary

```
Mybizz_management Provisioning
├── Create instance shell
├── Create Owner account
├── Set onboarding_status = "in_progress"
├── Create client record in Mybizz_management
├── Append-only amendment log entry
└── Send welcome email with login credentials

Owner Onboarding (Onboarding Form)
├── Step 1: Welcome and orientation
├── Step 2: Business Profile → link to Settings → Business Profile tab
├── Step 3: Palette Selection → link to Settings → Palette tab
├── Step 4: System Currency → dropdown + confirm (required, immutable after first transaction)
├── Step 5: Legal Acknowledgement → text + checkbox + confirm (required, logged in Mybizz_management)
├── Step 6: Completion → onboarding complete
└── → Dashboard

Post-Onboarding (accessible at any time)
├── Owner may return to Onboarding to change credentials
├── Settings form for ongoing configuration (tabbed, RBAC-governed)
├── Mybizz_management amendment log tracks all changes
└── Client Details Table: Owner may remove entries
    Retained Records Table: Owner may NOT remove (legal/financial)
```

### 9.2 Required Forms

**Mybizz_management (Provisioning):**
- Instance provisioning interface (Mybizz_management app — outside client instance scope)

**Client Instance (Owner Onboarding):**
1. `OnboardingForm` — the guided first-run experience with required elements
2. `SettingsForm` — single tabbed form for all configuration (RBAC-governed)
3. `BusinessProfileForm` — business profile section within Settings
4. `PaletteSettingsForm` — palette selection within Settings (custom component)
5. `VaultForm` — payment gateway configuration in Settings (RBAC-governed)

### 9.3 Wireframes Required

**Onboarding form wireframe (must be created):**
- `wireframe-onboarding-OnboardingForm.html` — the main onboarding form

**Existing wireframes reused:**
- `wireframe-settings-SettingsForm.html` — the Settings form (tabbed)
- `wireframe-settings-BusinessProfileForm.html` — business profile section
- `wireframe-custom-component-PaletteSettingsForm.html` — palette selection

**Mybizz_management wireframes (outstanding):**
- `wireframe-management-InstanceProvisionForm.html` — provisioning interface
- `wireframe-management-ProvisionConfirmationForm.html` — provisioning confirmation

---

## 10. Architectural Decisions (ADRs Referenced)

> All ADR references use full file names from `adr/`.

| ADR File | Title | Relevance |
|----------|-------|-----------|
| `16-system-currency-selection-and-immutability.md` | System Currency Selection and Immutability | Currency selected in onboarding, immutable after first transaction |
| `17-onboarding-vs-settings-boundary.md` | Onboarding vs Settings Boundary | Onboarding is a separate form; Settings is the single configuration surface |
| `18-legal-policy-responsibility-acknowledgement-and-clause-builder-architecture.md` | Legal Policy Responsibility Acknowledgement | Legal acknowledgement logged in Mybizz_management |
| `19-payment-gateway-configuration-is-a-settings-function-and-is-rbac-governed.md` | Payment Gateway Configuration Is Settings | Gateway configured in Settings, not onboarding |
| `20-onboarding-data-schema-alignment.md` | Onboarding Data Schema Alignment | All field names match authoritative-schema.md |
| `21-client-data-management-rights-and-mybizz-retention-boundary.md` | Client Data Management Rights | Owner may remove details from Client Details Table, not Retained Records Table |
| `22-tiers-model.md` | Tiers Model | Tiers (Free Trial, Launch, Pioneer, Business) are pricing constructs |
| `23-onboarding-resumability.md` | Onboarding Resumability | Progress preserved, can leave and return |
| `24-payment-gateway-mutability.md` | Payment Gateway Mutability | Gateway can change under approved rules |
| `25-onboarding-finality.md` | Onboarding Finality | Onboarding is resumable; credential changes are append-only |

---

## 11. Tier and Setup Support Model

- **Tiers** (Free Trial, Launch, Pioneer, Business) differ by price and feature access; they do not alter the onboarding flow. **[22-tiers-model.md]**
- **Setup support** is a separate operational offering:
  - All tiers include a 30-minute interactive setup session with Mybizz
  - A $100 DFY (Done For You) setup option is available; this is an operational service, not a different software onboarding flow
- The onboarding implementation plan does not vary by tier or support option

---

## 12. Next Steps

### 12.1 Immediate Actions Required

1. **Implement OnboardingForm** — welcome, required elements (currency + legal), navigation to Settings
2. **Implement tabbed SettingsForm** — M3 button tabs, RBAC-governed, "Return to Onboarding" button
3. **Implement system currency selection** — dropdown + confirmation button, immutability after first transaction
4. **Implement legal acknowledgement** — text with critical excerpts, "Read more" link, checkbox, confirmation button
5. **Implement client-instance audit log** — append-only credential change logging (sync to Mybizz_management amendment log when Mybizz_management is built — out of scope for current project)
6. **Add new columns to `business_profile` table** (9 columns listed in Section 7.1)

### 12.2 Dependencies

- **Mybizz_management app** is out of scope for current project. Client-instance audit log captures events locally; sync to Mybizz_management deferred.
- **`business_profile` table schema** must be updated with new columns
- **`PaletteSettingsForm` custom component** must be available for palette selection in Settings
- **The Vault** must be available for post-onboarding API key storage
- **Authentication system** must be in place (governed by separate Authentication & Administration design)

---

## 13. Final Conclusion

This plan defines an updated, implementation-ready onboarding system for Mybizz CS that:

1. **Uses Onboarding as a separate form** — guided first-run with required elements, links to Settings for specific tasks
2. **Uses Settings as a single tabbed form** — RBAC-governed, M3 button tabs, all configuration in one place
3. **Enables bidirectional navigation** — Onboarding → Settings and Settings → Onboarding
4. **Supports credential changes** — Owner may return to onboarding to change any credential at any time
5. **Uses append-only logging** — Mybizz_management amendment log never overwrites
6. **Respects client data rights** — Owner may remove details from Client Details Table, but not from Retained Records Table
7. **Uses authoritative schema names** — `business_profile` (with underscore), aligned with `20-onboarding-data-schema-alignment.md`

**The canonical V1 onboarding flow is:**
- Mybizz provisions instance shell and Owner account (no currency or gateway pre-set)
- Owner logs in → Onboarding form opens with welcome and progress indicator
- Owner configures Business Profile and Palette via links to Settings tabs
- Owner selects System Currency (dropdown + confirm, immutable after first transaction)
- Owner confirms Legal Acknowledgement (text + checkbox + confirm, logged in Mybizz_management)
- Onboarding complete → Owner directed to Dashboard
- Owner may return to Onboarding at any time to change credentials
- Settings form accessible at any time for ongoing configuration

**This plan is ready for implementation.**

---

*End of file — onboarding-implementation-plan.md*
