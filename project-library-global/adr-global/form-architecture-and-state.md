# `form-architecture-and-state` ADR — Form Architecture and State Management

**Status:** Approved  
**Date:** 2026-06-14  
**Authority:** Derived from platform-overview.md, `navigation-lambda-link-open-form` ADR, `client-instance-architecture` ADR, `anvil-platform-constraints` ADR

---

## Context

The Mybizz CS admin and customer portal uses Anvil's form-based UI model. Navigation uses lambda click handlers with `open_form()` (`navigation-lambda-link-open-form` ADR). Each `open_form()` call instantiates a fresh form — state does not carry across automatically.

Three problems must be solved:

1. **Form architecture** — How are forms composed and navigated?
2. **State management** — How does data flow between forms?
3. **Booking flow latency** — The multi-step booking flow (service → provider → date/time → meeting type → intake → payment → confirmation) has multiple server round-trips. How is sluggish UX avoided?

---

## Decision

### Form Architecture: Content Panel Pattern

Forms are composed using a content panel pattern. The `AdminLayout` (or `CustomerLayout`) contains a fixed sidebar and a `content_panel` `ColumnPanel`. Navigation swaps the content of this panel rather than replacing the entire form.

**Pattern:**

```
AdminLayout (permanent form)
├── Sidebar (Link components with lambda click handlers)
└── content_panel (ColumnPanel)
    └── Active content form (swapped on navigation)
```

**Why content_panel over open_form():**
- `open_form()` replaces the entire form — sidebar re-renders, layout state is lost
- Content panel keeps the layout stable while swapping content
- Back-button behavior is the navigation library's responsibility, not the form's

**Back-button behavior:** Handled by the Anvil Routing dependency for public pages. Admin/customer portal back-button behavior is a known limitation — browser back may not map cleanly to content panel state. This is acceptable for V1.

### State Management: globals.py Module

A `globals.py` client module holds shared state across forms. When a form needs to pass data to the next form, it writes to `globals.py`. The next form reads from `globals.py` on load.

**Pattern:**

```python
# globals.py
current_contact = None
current_booking = None
booking_flow_state = {}
```

```python
# In a form's navigation handler:
import globals
globals.current_contact = selected_contact
open_form('content_panel', 'contact_detail')

# In contact_detail form's __init__:
import globals
self.contact = globals.current_contact
```

**Trade-offs acknowledged:**
- `globals.py` is a flat module, not a structured store — it can become messy at scale
- State is global to the client session — no scoping or cleanup mechanism
- Alternative: passing data via `self.item` on content panel forms (more structured but verbose)

**For V1:** `globals.py` is sufficient. The number of shared state items is manageable (current contact, booking flow state, current notification). If state complexity grows beyond V1, a structured state service can be introduced.

### Booking Flow Latency: Batched Server Calls

The booking flow has 6+ steps, each potentially requiring a server call. To avoid sluggish UX:

1. **Pre-load data where possible** — Load provider availability, service details, and intake form templates in fewer, larger server calls
2. **Batch related operations** — Combine "get available times" + "get provider details" into a single server call
3. **Use client-side caching** — Cache server responses in `globals.py` or form-level state for the duration of the booking flow
4. **Defer non-critical operations** — Payment processing happens at the end; confirmation email is sent after the flow completes
5. **Background task for post-booking work** — Calendar event creation, reminder scheduling, and campaign enrollment happen asynchronously after confirmation

**Target:** No more than 3 server round-trips in the core booking path (service selection → confirmation).

---

## Alternatives Considered

### Form Architecture Alternatives

| Alternative | Reason rejected |
|---|---|
| Single form with toggle visibility | Form becomes bloated; hard to maintain |
| Multiple open_form() calls | Layout re-renders on every navigation; state loss |
| NavigationDrawerLayoutTemplate | M3 native component; not compatible with custom layout approach (`navigation-lambda-link-open-form` ADR) |

### State Management Alternatives

| Alternative | Reason rejected |
|---|---|
| Passing data via open_form() arguments | Not available with content panel pattern |
| URL parameters | Exposes state in URL; not suitable for sensitive data |
| Anvil Storage | Persistent across sessions; not suitable for transient flow state |
| Server-side session state | Requires server call to read; adds latency |

---

## Consequences

- ✅ Stable layout across navigation — sidebar and header persist
- ✅ Simple state sharing between forms via globals.py
- ✅ Booking flow designed for minimum latency from the start
- ⚠️ Back-button behavior in admin portal is limited in V1
- ⚠️ globals.py can become messy if state items grow beyond V1 scope
- ⚠️ Content panel forms must be designed to work without guaranteed state (defensive loading)

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/navigation-lambda-link-open-form.md`` | Navigation pattern that form architecture builds on |
| ``adr/adr-global/client-instance-architecture.md`` | Forms live in client instances; server calls resolve to master_template |
| ``adr/adr-global/anvil-platform-constraints.md`` | Server call latency constraint driving batch design |
| ``adr/adr-global/data-access-patterns.md`` | Data access patterns used in form data loading |
| `docs/architecture.md` | Section 5: Three-Tier Module Pattern; Section 6: Navigation Architecture |
| `docs/user-flows.md` | §2: Services Appointment Booking; §3: Admin Daily Operations |

---

*End of `form-architecture-and-state` ADR*
