# Mybizz CS — globals.py Contract

**Authority:** `form-architecture-and-state` ADR (Form Architecture and State Management)
**Status:** Living document — update as new shared state is added during implementation

---

## Purpose

This document defines every shared state variable in `globals.py`. Each entry specifies the variable name, type, which form sets it, which forms read it, and what happens when it's `None`.

**Rule:** Every form that reads from `globals.py` must use `getattr(globals, 'var_name', None)` and handle the `None` case defensively. Never assume a globals variable is set.

---

## State Variables

### current_contact

| Field | Value |
|-------|-------|
| **Type** | `app_tables.contacts` row or `None` |
| **Set by** | `ContactListForm` (on contact selection), `BookingEditorForm` (on contact auto-create) |
| **Read by** | `ContactViewerForm`, `ContactEditorForm`, `BookingEditorForm` |
| **When None** | Redirect to `ContactListForm` |

### current_booking

| Field | Value |
|-------|-------|
| **Type** | `app_tables.bookings` row or `None` |
| **Set by** | `BookingListForm` (on booking selection), `BookingEditorForm` (on booking creation) |
| **Read by** | `BookingViewerForm`, `InvoiceEditorForm` |
| **When None** | Redirect to `BookingListForm` |

### booking_flow_state

| Field | Value |
|-------|-------|
| **Type** | `dict` with keys below |
| **Set by** | `BookingEditorForm` (during multi-step booking flow) |
| **Read by** | `BookingEditorForm` (each step reads prior selections) |
| **When None** | Initialize empty dict at booking flow start |

**Keys:**

| Key | Type | Set at step | Read at step |
|-----|------|-------------|--------------|
| `service` | `app_tables.services` row | Step 1 (service selection) | Steps 2-7 |
| `provider` | `app_tables.users` row | Step 2 (provider selection) | Steps 3-7 |
| `date` | `str` (ISO date) | Step 3 (date selection) | Steps 4-7 |
| `time_slot` | `str` (HH:MM) | Step 3 (time selection) | Steps 4-7 |
| `meeting_type` | `str` | Step 4 (meeting type) | Steps 5-7 |
| `intake_data` | `dict` or `None` | Step 5 (intake form) | Steps 6-7 |
| `cached_context` | `dict` or `None` | Call 1 response cache | Steps 2-5 (avoid re-fetch) |

**Reset:** Clear `booking_flow_state` to `{}` on booking completion or cancellation.

### current_invoice

| Field | Value |
|-------|-------|
| **Type** | `app_tables.invoice` row or `None` |
| **Set by** | `InvoiceListForm` (on invoice selection), `BookingViewerForm` (on invoice link) |
| **Read by** | `InvoiceViewerForm`, `InvoiceEditorForm` |
| **When None** | Redirect to `InvoiceListForm` |

---

## Defensive Loading Pattern

Every form that reads from `globals.py` must follow this pattern:

```python
def __init__(self, **event_args):
    import globals
    self.contact = getattr(globals, 'current_contact', None)
    if self.contact is None:
        from anvil import open_form
        open_form('content_panel', 'contact_list')
        return
    # Continue with form initialization
```

**Do not** use `globals.current_contact` directly — it raises `AttributeError` if the variable was never set.

---

*End of globals.py Contract*
