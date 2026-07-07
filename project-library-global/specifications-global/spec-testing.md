# Mybizz CS — Test Specification (Canonical)

**Date:** 2026-06-16
**Source:** CEO review 2026-06-15 Section 6, Engineering review 2026-06-16
**Authority:** Build plan test framework, `anvil-platform-constraints` ADR, `form-architecture-and-state` ADR, `webhook-architecture` ADR, `dependency-based-not-multi-tenant` ADR

**Note:** General testing methodology (the Level 1/2/3 model, pure-function characteristics, the
Uplink safety protocol, generic test structure/naming, common test patterns) lives in
`spec-testing-methodology-standards.md` and is not repeated here. This document contains only this
project's specific flows, roles, and data — the actual test plan.

---

## 1. E2E Booking Flow Test Spec

**Covers:** CEO review Section 6 — "No end-to-end test spec covering the full happy path from
service selection to payment confirmation"

### Happy Path: Service Selection to Confirmation

**Preconditions:**
- Client instance provisioned with all tables
- At least one service configured in `services` table
- At least one provider (Staff user) assigned to the service
- Availability rules configured for the provider
- Payment gateway configured (Stripe or Paystack in test mode)
- Vault contains payment gateway secret keys

**Steps:**

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Navigate to booking form | Booking form loads with service list from Call 1 (`get_booking_context`) |
| 2 | Select a service | Providers assigned to this service are displayed |
| 3 | Select a provider | Available date range is displayed |
| 4 | Select a date | Available time slots are displayed (from Call 2 `get_available_slots`) |
| 5 | Select a time slot | Meeting type options are displayed (from pre-loaded data) |
| 6 | Select meeting type | Intake form is displayed (from pre-loaded data) |
| 7 | Fill intake form | Review/confirmation step is displayed |
| 8 | Confirm booking | Call 3 (`confirm_booking`) processes payment and creates booking |
| 9 | Confirmation screen | Booking confirmation displayed with booking ID, date, time, service, provider |
| 10 | Check `bookings` table | New row exists with status `CONFIRMED`, payment_intent_id set |
| 11 | Check `contacts` table | Contact auto-created (if new) or linked (if existing) |
| 12 | Check `invoice` table | Invoice auto-generated with correct amount |

**Performance gate:** Steps 1-9 complete in under 3 seconds on a standard connection.

**Data validation:**
- Booking amount matches service price
- Currency matches `business_profile.system_currency`
- Timezone conversion is correct (stored UTC, displayed in client timezone)
- Contact has correct email, phone, name from intake form

### Edge Case: Existing Contact Books

Same as happy path, but the contact email matches an existing `contacts` row.

**Expected:** No duplicate contact created. Booking links to existing contact.

### Edge Case: Provider Unavailable

**Steps:**
1. Start booking flow for a service
2. Select a provider
3. Select a date where the provider has no availability

**Expected:** "No available time slots" message. User can select a different date or provider.

### Edge Case: Double-Booking Prevention

**Steps:**
1. Two users simultaneously book the same provider at the same time slot
2. First booking succeeds
3. Second booking attempts the same slot

**Expected:** Second booking gets "Slot no longer available" error. No duplicate booking created.

---

## 2. Error Path Test Cases

**Covers:** CEO review Section 6 — "Build plan specifies happy-path tests but not error-path
tests"

### 2.1 Payment Failure

**Scenario:** Stripe/Paystack returns `payment_intent.payment_failed`

**Steps:**
1. Start booking flow through to payment step
2. Use a test card that triggers payment failure (Stripe: `4000000000000002`)
3. Submit payment

**Expected:**
- User sees "Payment failed" error with retry option
- `bookings` row created with status `PENDING` (not `CONFIRMED`)
- `webhook_log` row has `processed=True`, `error_message` set
- No invoice generated
- User can retry payment without re-entering booking details

### 2.2 Webhook Signature Failure

**Scenario:** Webhook arrives with invalid signature

**Steps:**
1. Send a POST request to `/api/webhooks/stripe` with a valid payload but incorrect signature

**Expected:**
- HTTP 400 returned
- No `webhook_log` row created (or row created with `verified=False`)
- No background task dispatched
- Error logged in `error_log`

### 2.3 Webhook Dispatch Failure

**Scenario:** `launch_background_task()` raises an exception

**Steps:**
1. Simulate task queue full condition
2. Webhook arrives with valid signature

**Expected:**
- `webhook_log` row created with `processed=False`
- HTTP 200 returned to gateway (gateway won't retry)
- Reconciliation task picks up the orphaned row within 5 minutes
- Background task eventually processes the webhook

### 2.4 Brevo API Rate Limit (429)

**Scenario:** Brevo returns 429 on campaign email send

**Steps:**
1. Campaign enrollment task attempts to send email
2. Brevo API returns 429

**Expected:**
- Contact skipped in this run (not marked as failed)
- Task continues with next contact
- Failed contact retried on next hourly run
- Error logged in `error_log`

### 2.5 Vault Secret Not Found

**Scenario:** Payment gateway key missing from Vault

**Steps:**
1. Attempt to process a payment
2. `get_vault_secret('stripe_secret_key')` raises `KeyNotFoundError`

**Expected:**
- User sees "Unable to retrieve credential. Please contact support."
- Error logged in `health_log` with status `"unhealthy"`
- No 500 error exposed to user
- Mybizz_management alerted (if operational)

### 2.6 Contact Deduplication Conflict

**Scenario:** Same email, different names submitted from landing page

**Steps:**
1. Contact exists with email `jane@example.com`, name `Jane Smith`
2. Landing page form submitted with email `jane@example.com`, name `J. Smith`

**Expected:**
- Existing contact updated (not rejected)
- `leads` row created with `converted_to_contact_id` pointing to existing contact
- Welcome campaign does NOT re-enroll (contact already exists)
- Name discrepancy logged for manual review

### 2.7 Server Call Timeout During Booking

**Scenario:** `get_available_slots()` takes longer than 30 seconds

**Steps:**
1. Start booking flow
2. Select provider and date
3. Server call times out

**Expected:**
- User sees "Connection timed out. Please try again." with retry button
- No partial booking data persisted
- `globals.booking_flow_state` preserved — user can retry without starting over

---

## 3. RBAC Coverage Matrix

**Covers:** CEO review Section 6 — "No test matrix showing which roles can access which
forms/tabs"

### Roles

| Role | Description |
|------|-------------|
| **Owner** | Full access to all forms and settings tabs |
| **Manager** | Most forms and settings; some tabs restricted |
| **Admin** | Operational forms; limited settings access |
| **Staff** | Own bookings, own schedule, limited contacts |
| **Customer** | Own bookings only (V2 — customer portal) |

### Form Access Matrix

| Form | Owner | Manager | Admin | Staff | Customer |
|------|-------|---------|-------|-------|----------|
| DashboardForm | ✅ | ✅ | ✅ | ✅ | ❌ |
| BookingCalendarForm | ✅ | ✅ | ✅ | ✅ (own only) | ❌ |
| BookingEditorForm | ✅ | ✅ | ✅ | ✅ (own only) | ❌ |
| BookingViewerForm | ✅ | ✅ | ✅ | ✅ (own only) | ❌ |
| ContactListForm | ✅ | ✅ | ✅ | ❌ | ❌ |
| ContactEditorForm | ✅ | ✅ | ✅ | ❌ | ❌ |
| ContactViewerForm | ✅ | ✅ | ✅ | ❌ | ❌ |
| InvoiceListForm | ✅ | ✅ | ❌ | ❌ | ❌ |
| InvoiceEditorForm | ✅ | ✅ | ❌ | ❌ | ❌ |
| ServiceListForm | ✅ | ✅ | ✅ | ❌ | ❌ |
| ServiceEditorForm | ✅ | ✅ | ❌ | ❌ | ❌ |
| EmailCampaignListForm | ✅ | ✅ | ❌ | ❌ | ❌ |
| EmailCampaignEditorForm | ✅ | ✅ | ❌ | ❌ | ❌ |
| TaskListForm | ✅ | ✅ | ✅ | ✅ (own only) | ❌ |
| BlogEditorForm | ✅ | ✅ | ❌ | ❌ | ❌ |
| LandingPageListForm | ✅ | ✅ | ❌ | ❌ | ❌ |
| LandingPageEditorForm | ✅ | ✅ | ❌ | ❌ | ❌ |
| VaultForm | ✅ | ❌ | ❌ | ❌ | ❌ |
| OnboardingForm | ✅ | ❌ | ❌ | ❌ | ❌ |

### Settings Tab Access Matrix

| Settings Tab | Owner | Manager | Admin | Staff |
|--------------|-------|---------|-------|-------|
| Business Profile | ✅ | ✅ | ❌ | ❌ |
| Palette | ✅ | ✅ | ❌ | ❌ |
| System Currency | ✅ | ❌ | ❌ | ❌ |
| Legal Acknowledgement | ✅ | ❌ | ❌ | ❌ |
| Display Currency | ✅ | ✅ | ❌ | ❌ |
| Payment Gateway | ✅ | ❌ | ❌ | ❌ |
| Services | ✅ | ✅ | ❌ | ❌ |
| Users & Permissions | ✅ | ❌ | ❌ | ❌ |
| Email Configuration | ✅ | ✅ | ❌ | ❌ |
| Theme | ✅ | ✅ | ❌ | ❌ |
| Feature Flags | ✅ | ❌ | ❌ | ❌ |

### Test Procedure

For each form/tab in the matrix:
1. Log in as the specified role
2. Attempt to navigate to the form/tab via direct URL or navigation link
3. If access granted: verify form loads correctly
4. If access denied: verify redirect to dashboard or "Access Denied" message
5. Verify the navigation link is hidden (not just disabled) for restricted roles

### RBAC Decorator Pattern

Every server function must have an RBAC decorator. The test verifies:

```python
# Expected pattern on every server function
@authenticated_endpoint(roles=['Owner', 'Manager', 'Admin'])
def get_contacts():
    ...
```

**Test:** Grep all server functions in `server_*/` for `@authenticated_endpoint` or equivalent
decorator. Any function without one is a bug.

---

*Test Specification (Canonical) v2.0 — consolidated from test-specifications.md into
spec-testing.md. testing-standards.md's content is superseded by testing-methodology-standards.md
(Standards Library) and is not duplicated here.*
