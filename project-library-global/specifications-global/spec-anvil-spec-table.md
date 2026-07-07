# Mybizz CS — Anvil Specification Table

**Purpose:** Consolidated specification and methods reference for Mybizz Anvil development
**Dependencies:** M3 Theme (4UK6WHQ6UX7AKELK), Routing (3PIDO5P3H4VPEMPL)
**Blocked:** Anvil Extras — never used

## Table of Contents

- [1. Navigation & Layout](#1-navigation--layout)
- [2. Component Specifications](#2-component-specifications)
- [3. Data Binding & Form State](#3-data-binding--form-state)
- [4. Data Tables](#4-data-tables)
- [5. Packages, Namespaces & Structure](#5-packages-namespaces--structure)
- [6. Startup & Authentication](#6-startup--authentication)
- [7. Coding Standards](#7-coding-standards)
- [8. Designer Gotchas](#8-designer-gotchas)
- [9. Testing Specifications](#9-testing-specifications)
- [10. Uplink](#10-uplink)
- [11. Background Tasks](#11-background-tasks)
- [12. Design & Styling](#12-design--styling)
- [13. Architecture & Scope](#13-architecture--scope)
- [14. Quick Reference](#14-quick-reference)

---

## 1. Navigation & Layout

### 1.1 Authenticated Navigation Standard

| Topic | Specification |
|---|---|
| ADR | `navigation-lambda-link-open-form.md` (navigation pattern), `htmltemplate-use.md` (layout shell) |
| Layout type | Native M3 Layouts (`NavigationRailLayout` for admin, `NavigationDrawerLayout` for customer portal) |
| Sidebar nav component | Plain `Link` components |
| Sidebar nav creation | Built programmatically in `build_navigation()` |
| Navigation action | `open_form(string)` via lambda click handler |
| Variable capture | **MANDATORY**: `lambda e, fn=form_name, a=attr_name: self._nav_click(fn, a)` |
| NEVER use | `navigate_to`, `HTMLTemplate` (banned per `htmltemplate-use` ADR) |
| Reason | `navigate_to` requires instantiated Forms; `HTMLTemplate` banned — native M3 Layouts and components only |

```python
class AdminLayout(AdminLayoutTemplate):
    def build_navigation(self):
        for attr_name, label, icon, form_name in _NAV_STRUCTURE:
            nav_link = Link(text=label, icon=icon)
            nav_link.set_event_handler(
                'click',
                lambda e, fn=form_name, a=attr_name: self._nav_click(fn, a),
            )
            setattr(self, attr_name, nav_link)
            self.sidebar_panel.add_component(nav_link)
```

**Mandatory:** Loop variable capture `fn=form_name, a=attr_name`. Without it, all lambdas resolve to the last loop value.

### 1.2 Public Routing Standard

| Topic | Specification |
|---|---|
| Dependency | Routing `3PIDO5P3H4VPEMPL` |
| Use for | Public pages, shareable URLs, bookmarkable content, browser history |
| Route decorator | `@router.route(...)` |
| Route constructor signature | `def __init__(self, routing_context, **properties)` |
| Parameter access | `routing_context.params['id']` |
| URL navigation | `navigate(path=..., params=..., query=...)` only in routed public flows |

```python
from routing import router

@router.route("/")
class HomePage(HomePageTemplate):
    def __init__(self, routing_context, **properties):
        self.init_components(**properties)

@router.route("/services/:id")
class ServiceDetail(ServiceDetailTemplate):
    def __init__(self, routing_context, **properties):
        self.service_id = routing_context.params['id']
        self.init_components(**properties)
```

### 1.3 Parameterised Navigation

| Use Case | Standard |
|---|---|
| Detail form from row click | `open_form('ContactDetailForm', contact_id=self.item['id'])` |
| Authenticated internal views | `open_form()` |
| Public URL paths | Routing dependency |

```python
# From content forms (row clicks, detail views)
open_form('ContactDetailForm', contact_id=self.item['id'])
```

### 1.4 Layout Selection

| Need | Layout Used | Notes |
|---|---|---|
| Admin interface | `NavigationRailLayout` | Sidebar via `build_navigation()` + lambda |
| Customer portal | `NavigationDrawerLayout` | Same as admin pattern |
| Auth pages | Custom Form | No layout shell required |
| Error pages | Custom Form | Minimal chrome |
| Guided step workflow | `NavigationRailLayout` with sidesheet | Separate from sidebar navigation |

### 1.5 Sidesheet Specification

| Topic | Specification |
|---|---|
| Correct term | **Sidesheet** (not Slidesheet) |
| Layout | M3 native pattern using `NavigationRailLayout` |
| Enablement | `show_sidesheet = True` |
| Container | `SidesheetContent` |
| Primary use | Guided multi-step workflows, review flows, detail panels |
| State pattern | Pass accumulated data via `self.item` between steps |
| Server calls | Submit once at final step; no server calls between steps |

```python
def btn_next_click(self, **event_args):
    self.content_panel.clear()
    self.content_panel.add_component(Slide2(item=self.item))
```

---

## 2. Component Specifications

### 2.1 Core M3 Components

| Category | Component | Prefix | Standard |
|---|---|---|---|
| Typography | Heading | `hdg_` | Use for page and section headers |
| Buttons | Button | `btn_` | Filled, outlined, text hierarchy |
| Buttons | IconButton | `btn_` | Icon-only actions |
| Buttons | ToggleIconButton | `btn_` or `toggle_` | Toggle state actions |
| Input | TextBox | `txt_` | Use outlined appearance for forms |
| Input | TextArea | `txt_` | Same naming as TextBox |
| Input | DropdownMenu | `dd_` | Leave items empty in Designer; set in code |
| Input | Checkbox | `cb_` | Boolean choices |
| Input | RadioButton | `rb_` | Single choice |
| Input | DatePicker | `dp_` | Use outlined appearance |
| Input | FileLoader | `fu_` | Uploads |
| Containers | ColumnPanel | `col_` | Primary structural container |
| Containers | FlowPanel | `flow_` | Horizontal layouts, responsive wrapping |
| Containers | RepeatingPanel | `rp_` | Lists |
| Containers | Data Grid | `dg_` | Tabular display |
| Display | Card | `card_` | Elevated, filled, outlined |
| Display | CardContentContainer | `col_` | Auto-created inside Card |
| Display | SidesheetContent | `col_` | Container for sidesheets |
| Menus | ButtonMenu / IconButtonMenu / AvatarMenu | `menu_` | Action menus |
| Feedback | LinearProgressIndicator / CircularProgressIndicator | semantic name | Usually single-use |
| Integrations | Plot | `plot_` | Use if multiple charts |

### 2.2 Components vs Widgets

| Type | When to Use | Description |
|---|---|---|
| Custom Component (90%) | Default choice | Reusable Anvil Form written entirely in Python; supports data bindings, events, properties, Designer configuration |
| Widget (10%) | Third-party JS needed | Wrapper around HTML/CSS/JavaScript using the HTML panel; only for third-party JS libraries, browser APIs, or custom DOM rendering |

**Best Practice Pattern:** Custom Component handles layout, data, events, API → Widget (internally, if needed) hidden behind the component interface. Keeps the rest of the app pure Anvil.

### 2.3 Button Hierarchy

| Level | Appearance / Role | Use |
|---|---|---|
| Primary | Filled | One primary action per screen |
| Secondary | Outlined | Cancel, Back, less prominent actions |
| Tertiary | Text | Low-emphasis actions |

### 2.4 Typography Standards

| Use Case | Component | Standard |
|---|---|---|
| Page title | Heading | `headline-large` or headline/large equivalent |
| Section title | Heading | `headline-small` |
| Card title | Heading | title/medium equivalent |
| Body copy | Text | `body-medium` |
| Field label | Text | label/medium equivalent |
| Caption | Text | body/small equivalent |
| NEVER use | Legacy `Label` | M3 apps use Text/Heading instead |

### 2.5 Input Standards

| Component | Standard |
|---|---|
| TextBox / TextArea | Outlined appearance for dense forms |
| DropdownMenu | Outlined appearance; items set in code |
| DatePicker | Outlined appearance |
| Error state | Use component error state and supporting/error text pattern |
| Password field | Set `hide_text = True` in Designer |
| Validation failure | Use `role='outlined-error'` with error in placeholder |

### 2.6 Containers and Layout Rules

| Component | Standard |
|---|---|
| ColumnPanel | Default structural container |
| FlowPanel | Frequently used; orientation set in code |
| FlowPanel | Preferred for responsive wrapping |
| XYPanel | Avoid for primary layout work |
| Card | Place content inside CardContentContainer |

**Card Pattern:** When you add a Card in Designer, Anvil automatically creates a CardContentContainer inside it. Place all card content inside the CardContentContainer.

### 2.7 Avoid / Restricted Components

| Component / Pattern | Rule |
|---|---|
| Anvil Extras | Excluded — see `14-anvil-extras-exclusion.md` |
| `NavigationLink` for sidebar | Not used in current architecture — see `2-navigation-lambda-link-open-form.md` |
| `NavigationDrawerLayoutTemplate` | Not used in current architecture — see `2-navigation-lambda-link-open-form.md` |
| `navigate_to` for internal nav | Not used — see `2-navigation-lambda-link-open-form.md` |
| RichText | Use sparingly; not core M3 |
| Legacy multi-vertical components | Removed under `9-multi-vertical-to-single-vertical-conversion.md` |
| Material Web Components | Excluded — see `14-anvil-extras-exclusion.md` |
| Material Web Components | Excluded — see `14-anvil-extras-exclusion.md` |
| Material Web Components | Excluded — see `14-anvil-extras-exclusion.md` |

---

## 3. Data Binding & Form State

### 3.1 self.item Standard

| Topic | Specification |
|---|---|
| Form record state | Use `self.item` |
| Existing record | `self.item = contact` |
| New record | `self.item = {}` |
| Save action | Pass `self.item` to server call |
| Writeback | Must be enabled in Designer using W toggle |
| Code control | Writeback cannot be enabled in code |

```python
class ContactEditorForm(ContactEditorFormTemplate):
    def __init__(self, contact=None, **properties):
        if contact:
            self.item = contact  # Existing record
        else:
            self.item = {}  # New record
        self.init_components(**properties)

    def btn_save_click(self, **event_args):
        result = anvil.server.call('save_contact', self.item)
        if result['success']:
            open_form('ContactListForm')
```

Write Back must be enabled in the Designer Data Bindings panel (W toggle). Cannot be set in code.

### 3.2 Binding Restrictions

| Form Type | Rule |
|---|---|
| Standard CRUD form | Use `self.item` write-back |
| Settings/config form | Do **NOT** use data binding write-back if form loads and saves explicitly via server calls |
| Multi-step sidesheet | Pass `self.item` between steps |

### 3.3 Refresh Pattern

| Topic | Rule |
|---|---|
| Form-side state | Keep UI thin; bind or map into `self.item` |
| Business logic | Lives in Client Modules or Server Modules, not Forms |
| Button handlers | Call functions or server methods; do not hold core logic |

---

## 4. Data Tables

### 4.1 Core Rules

| Topic | Specification |
|---|---|
| Primary key | Use built-in Row ID via `row.get_id()` / documented Row ID pattern |
| Custom auto-increment PKs | Never create unless explicit human-readable ID required |
| Relationships | Store Row objects, not integer IDs |
| Client access | All tables set to `No access` for client code |

### 4.2 Row Identification

Anvil manages primary keys automatically. Every row has a built-in unique ID accessible via `row.get_id()`. Do not create custom auto-increment columns.

### 4.3 Table Linking

Store Row objects directly in link columns — not integer IDs:

```python
contact = app_tables.contacts.add_row(email='jane@example.com')
booking = app_tables.bookings.add_row(contact=contact)
```

### 4.4 Data Access (Dependency-Based Architecture)

Each client instance is a separate Anvil app with its own isolated database. No tenant discriminator columns or tenant-filtered queries are needed — see `dependency-based-not-multi-tenant` ADR:

```python
@anvil.server.callable
def get_all_contacts():
    user = anvil.users.get_user()
    if not user:
        raise Exception("Not authenticated")
    return app_tables.contacts.search()
```

### 4.5 Query Standards

| Operation | Standard |
|---|---|
| Single record | `get()` — returns `None` if not found |
| Multiple records | `search()` — returns SearchIterator (lazy-loaded) |
| Sorting | `tables.order_by('field', ascending=False)` |
| Pagination | Slicing: `[:50]`, `[50:100]` |
| Query operators | `q.any_of()`, `q.none_of()`, `q.greater_than()`, `q.between()`, `q.full_text_match()`, `q.like()` |

### 4.6 Transaction Standards

| Use Case | Rule |
|---|---|
| Counters | Use `@tables.in_transaction` |
| Multi-step atomic work | Use `@tables.in_transaction` |
| Consistency-critical writes | Use transaction wrappers |

```python
@tables.in_transaction
def get_next_contact_number():
    counter = app_tables.contact_counter.get()
    current = counter['value']
    counter['value'] += 1
    return f"C-{current:05d}"
```

### 4.7 Column Types

| Anvil Type | Python Type | Use Case |
|---|---|---|
| Text | `str` | Names, emails, descriptions |
| Number | `int`, `float` | Prices, quantities |
| True/False | `bool` | Flags |
| Date | `datetime.date` | Birthdays, deadlines |
| Date and Time | `datetime.datetime` | Timestamps |
| Simple Object | `dict`, `list` | JSON-like settings/tags |
| Media | Media object | Files, images |
| Link (Single) | Row object | One relationship |
| Link (Multiple) | List of Rows | Many-to-many |

### 4.8 Mandatory Table Columns

| Column | Rule |
|---|---|
| `created_at` | Mandatory |
| `updated_at` | Mandatory if record is mutable |

### 4.9 Common Mistakes

| Mistake | Rule |
|---|---|
| Store tuple/int IDs instead of Row objects | Never |
| Load all records then take first | Use `get()` |
| Convert SearchIterator too early | Avoid unless needed |
| Modify rows while iterating live search | Convert to list first |

---

## 5. Packages, Namespaces & Structure

### 5.1 Package Model

| Topic | Specification |
|---|---|
| Package meaning | Folder + Python namespace (real Python import namespaces, not just visual grouping) |
| Client package contents | Forms and Client Modules; run in browser; cannot access Data Tables or secrets directly |
| Server package contents | Server Modules only; run on Anvil's Python server; can access Data Tables, secrets, enforce security |
| Boundary | Client/server split is a hard security boundary |

### 5.2 Import Patterns

```python
from .users import user_logic           # Same package
from ..users.user_logic import fn       # Parent package
from ...server_shared.utilities import fn  # Cross-boundary
```

### 5.3 Structure Standard

| Topic | Standard |
|---|---|
| Preferred organization | Feature-based, not layer-based |
| Forms | UI and event handlers only |
| Modules | Reusable logic, helpers, non-visual code |
| Server modules | Security, business logic, persistence orchestration |
| Avoid | Giant utils modules, database access in client code, heavy logic in Forms |

### 5.4 Recommended Structure

```text
client_code/
  auth/          layouts/       dashboard/     settings/
  bookings/      services/      customers/     crm/
  blog/          public_pages/  shared/

server_code/
  server_auth/       server_settings/   server_dashboard/
  server_bookings/   server_services/   server_customers/
  server_marketing/  server_payments/   server_blog/
  server_shared/     server_analytics/  server_emails/
```

---

## 6. Startup & Authentication

### 6.1 Startup Model

| Topic | Specification |
|---|---|
| Startup mechanism | Anvil starts with a Startup Form, not a Startup Module |
| Startup location | Defined in app settings |
| Client modules | Do not self-execute; run only when imported/called |
| Startup logic | Place in Startup Form and design to run once |

### 6.2 Redirect Pattern

| Rule | Standard |
|---|---|
| Authenticated staff roles | `open_form('dashboard.DashboardForm')` then `return` |
| Authenticated customer roles | `open_form('customers.ClientPortalForm')` then `return` |
| Anonymous user | Continue normal startup Form display |
| Critical | Any `open_form()` in `__init__` must be followed by `return` |

```python
class HomePage(HomePageTemplate):
    def __init__(self, **properties):
        user = anvil.users.get_user()
        if user:
            if user['role'] in ['owner', 'manager', 'admin', 'staff']:
                open_form('dashboard.DashboardForm')
                return
            else:
                open_form('customers.ClientPortalForm')
                return
        # If not logged in, continue with HomePage display
        self.init_components(**properties)
```

---

## 7. Coding Standards

### 7.1 Core Philosophy

| Topic | Standard |
|---|---|
| Development | Fail fast |
| Production | Fail safely |
| Code quality | Readable, testable, observable, resilient |
| Tracebacks | Sacred; preserve them |

### 7.2 Mandatory Function Standards

| Topic | Rule |
|---|---|
| Docstrings | Every non-trivial function must have a docstring |
| Type hints | All public functions must use type annotations |
| Inline comments | Explain why, not what |
| Constants | No magic strings, URLs, keys, or duplicated literals |
| Naming | `snake_case` for functions/variables, `PascalCase` for classes |

### 7.3 Logging Standard

| Level | Use |
|---|---|
| DEBUG | Internal state, dev diagnostics |
| INFO | Normal operations |
| WARNING | Unexpected but recoverable |
| ERROR | Operation failed |
| CRITICAL | System integrity at risk |

| Logging Rule | Requirement |
|---|---|
| `print` in server code | Forbidden |
| Failure logging | All server failures must be logged |
| Context | Logs must include enough context to reproduce issues |

### 7.4 Exception Handling Rules

| Rule | Specification |
|---|---|
| Bare `except:` | Never |
| `except pass` | Never |
| Catch scope | Catch only expected exceptions |
| Unexpected errors | Re-raise; fail loudly in development |
| Re-raise | Use `raise` to preserve traceback |
| Exception chaining | Use `raise NewError(...) from e` when wrapping |

### 7.5 Architecture Rules

| Topic | Rule |
|---|---|
| UI code | Presentation only |
| Business logic | Client Modules or Server Modules; server preferred for critical rules |
| Button click handlers | Call functions; do not contain core logic |
| Pure logic extraction | Mandatory for important calculations/decisions |
| Validation | Validate incoming complex dictionaries early |

### 7.6 Global Error Handling

| Topic | Rule |
|---|---|
| Global client error handler | Required |
| Purpose | Catch uncaught errors, log tracebacks server-side, show safe user messages |
| Server crash visibility | Keep App Logs open; do not assume UI errors show everything |

---

## 8. Designer Gotchas

### 8.1 Must Be Set in Code

| Property / Pattern | Rule |
|---|---|
| `FlowPanel.orientation` | Cannot be set in Designer; must be set in code with explanatory comment |

### 8.2 Must Be Set in Designer

| Property / Pattern | Rule |
|---|---|
| TextBox outlined style | Set via `appearance` in Designer (not `role`) |
| TextBox password masking | Set `hide_text = True` in Designer. There is no `type` property. |
| Write Back | Enable W toggle in Designer Data Bindings panel |

### 8.3 Correct but Surprising Behaviour

| Behaviour | Rule |
|---|---|
| CardContentContainer auto-creation | Expected when adding a Card; place all card content inside it |

### 8.4 Problem Patterns

| Pattern | Rule |
|---|---|
| Data bindings on settings forms | Do not use `self.item` write-back on forms with explicit load/save server logic |
| Stray event handlers | Check `__init__.py` for auto-generated empty handlers and remove them |
| DropdownMenu items in Designer | Leave empty in Designer; populate in code |
| YAML verification | `form_template.yaml` is source of truth; verify component-by-component |

---

## 9. Testing Specifications

### 9.1 Testing Levels

| Level | Scope | Runtime | Risk | Rule |
|---|---|---|---|---|
| Level 1 | Pure logic tests | Local Python | Zero | Continuous, first step |
| Level 2 | Integration tests via Uplink | Local Python + Anvil | Medium | Only after pure logic passes and backup exists |
| Level 3 | End-to-end manual verification | Browser | Low | After work increment complete |

### 9.2 Pure Logic Standards

| Rule | Specification |
|---|---|
| Anvil imports | None |
| Side effects | None |
| Determinism | Required |
| Scope | Calculations, validations, transformations |
| Execution | Milliseconds, local only |

### 9.3 Pure Logic Extraction

Business logic with zero Anvil dependencies:

```python
# order_logic.py — NO Anvil imports
def calculate_order_total(price: float, tax_rate: float, max_total: float) -> dict:
    if price < 0:
        raise OrderLogicError("Price must be non-negative")
    subtotal = price
    tax = subtotal * tax_rate
    total = subtotal + tax
    if total > max_total:
        raise OrderLogicError("Order total exceeds maximum")
    return {"subtotal": subtotal, "tax": tax, "total": total}
```

### 9.4 Server Module (Thin Wrapper)

```python
@anvil.server.callable
def create_order(customer_email: str, price: float):
    user = anvil.users.get_user()
    if not user:
        raise Exception("Authentication required")
    order_calc = calculate_order_total(price, DEFAULT_TAX_RATE, MAX_ORDER_TOTAL)
    order = app_tables.orders.add_row(
        total=order_calc["total"], status="pending"
    )
    return order
```

### 9.5 Integration Test Safety Protocol

| Phase | Mandatory Steps |
|---|---|
| Before | Pure logic tests pass, backup created, Uplink verified, dev log updated |
| During | Read before write, mark test data with `source='Test'`, verify writes, log operations |
| After | Clean test data, verify cleanup, document results, backup on success |

### 9.6 Test Structure Standards

| Topic | Standard |
|---|---|
| Function naming | `test_feature_scenario` |
| Module naming | `testmodulename.py` |
| Assertions | Plain `assert` |
| Runner | `run_all_tests()` style function |
| Failure handling | Failed tests stop progress; fix immediately |

### 9.7 Test Execution

```bash
# Level 1: Pure logic (local)
python test_order_logic.py

# Level 2: Integration (Uplink)
python test_orders_integration.py
```

### 9.8 Backup Relationship

| Event | Rule |
|---|---|
| Before integration tests | Backup required |
| After passing integration tests | Backup required |
| Failed tests | Never backup |

---

## 10. Uplink

### 10.1 Uplink Test Usage

| Topic | Specification |
|---|---|
| Use for | Integration testing only |
| Not needed for | Pure business logic |
| Connection prerequisite | Verify Uplink connection before running tests |
| Safety | Backup before tests |
| Data hygiene | Mark and clean test data |

### 10.2 Setup

```bash
pip install anvil-uplink
export ANVIL_UPLINK_KEY="your-server-uplink-key"
```

### 10.3 Connection Pattern

```python
import anvil.server, os
anvil.server.connect(os.environ['ANVIL_UPLINK_KEY'])
try:
    result = anvil.server.call('server_function_name')
finally:
    anvil.server.disconnect()
```

### 10.4 Safety Rules

| Rule | Specification |
|---|---|
| Uplink type | Always use Server Uplink (not Client Uplink) for server-side work |
| Keys | Never hardcode Uplink keys |
| Disconnect | Always disconnect after operations |
| Smoke tests | Use smoke tests before running full integration tasks |
| Editor changes | Restart server runtime after changes in Anvil editor |

### 10.5 Uplink Workflow

| Step | Rule |
|---|---|
| 1 | Run pure logic tests locally |
| 2 | Confirm all passing |
| 3 | Create backup |
| 4 | Verify Uplink connection |
| 5 | Run integration tests |
| 6 | Clean up test data |
| 7 | Document results |
| 8 | Create post-success backup |

---

## 11. Background Tasks

| Topic | Specification |
|---|---|
| Decorator | `@anvil.server.background_task` |
| Timeout limit | No timeout limit (unlike 30-second server call limit) |
| Threshold | Any operation expected to exceed 22 seconds (75% of Anvil's 30-second timeout) must use background tasks |
| Use cases | Email campaigns, batch processing, long-running imports/exports |

```python
@anvil.server.background_task
def process_email_campaigns():
    """Hourly — sends via Brevo Campaigns API. No timeout limit."""
    campaigns = app_tables.email_campaigns.search(status='active')
    for campaign in campaigns:
        process_enrollments(campaign)
```

---

## 12. Design & Styling

### 12.1 M3 Dependency Rules

| Topic | Specification |
|---|---|
| Dependency ID | `4UK6WHQ6UX7AKELK` |
| Import | `import m3.components as m3` |
| Theme mixing | Do not mix M3 with legacy themes |
| Versioning | Track theme version and test after updates |

### 12.2 Styling Standards

| Topic | Rule |
|---|---|
| Theme usage | Use M3 theme roles and tokens; no hardcoded colors |
| Button styling | Use filled / outlined / text hierarchy |
| Inputs | Outlined appearance for forms |
| Typography | Use Text / Heading, not legacy Label |
| Responsive layout | Prefer FlowPanel and proper container layout |

### 12.3 Component Behaviour Rules

| Topic | Rule |
|---|---|
| One primary button | Per screen |
| Error display | Use error state + supporting text pattern |
| Loading state | Show progress indicator, disable action button during save |
| Responsive cards | Use FlowPanel for wrapping dashboards |

---

## 13. Architecture & Scope

### 13.1 Vertical Scope

| Topic | Specification |
|---|---|
| ADR | `9-multi-vertical-to-single-vertical-conversion.md` |
| Platform focus | Consulting & Services only |
| Removed | E-commerce, Hospitality, Membership |

### 13.2 Architectural Boundaries

| Boundary | Rule |
|---|---|
| Client code | UI, validation, server calls |
| Server code | Security, Data Tables, secrets, business logic |
| Form | Presentation and event wiring |
| Client Module | Reusable UI-adjacent logic |
| Server Module | Callable APIs, orchestration, persistence |
| Pure logic module | Framework-free rules and calculations |

### 13.3 Dependency Lock

| Dependency | Rule |
|---|---|
| M3 Theme | Required |
| Routing dependency | Required where public routing is used |
| Anvil Extras | Never used |

---

## 14. Quick Reference

### 14.1 Mandatory Rules Checklist

| Topic | Rule |
|---|---|
| Sidebar nav | Plain `Link` + lambda capture |
| Internal nav | `open_form()` |
| Public nav | Routing dependency |
| Startup redirect | `open_form()` followed by `return` |
| Form state | `self.item` + Designer Write Back |
| Settings forms | No write-back binding |
| Relationships | Store Row objects, not IDs |
| Table permissions | No client access |
| Logging | Use logging, never `print` in server code |
| Exceptions | No bare except, no `except pass` |
| Tracebacks | Preserve with `raise` / `raise ... from e` |
| Testing | Pure logic first, integration second, backup before Uplink |
| M3 layouts | NavigationRailLayout/NavigationDrawerLayout for admin/customer shell |
| Designer | `FlowPanel.orientation` in code; TextBox appearance/hide_text in Designer |
| DropdownMenu | Items in code, not Designer |
| Cards | Use CardContentContainer |
| Sidesheet | Correct term is Sidesheet; use `NavigationRailLayout` + `show_sidesheet` |
| Extras | Never use Anvil Extras |
| Components vs Widgets | Custom Component default (90%); Widget only for third-party JS (10%) |
| Background tasks | Required for operations exceeding 22 seconds |

### 14.2 Prefix Reference

| Prefix | Component Types |
|---|---|
| `col_` | ColumnPanel, CardContentContainer, SidesheetContent |
| `flow_` | FlowPanel |
| `btn_` | Button, IconButton, Outlined Button |
| `hdg_` | Heading |
| `txt_` | TextBox, TextArea |
| `dd_` | DropdownMenu |
| `cb_` | Checkbox |
| `rb_` | RadioButton |
| `dp_` | DatePicker |
| `fu_` | FileLoader |
| `dg_` | Data Grid |
| `rp_` | RepeatingPanel |
| `card_` | Card, InteractiveCard |
| `link_` | Link |
| `nav_` | Sidebar Link components |
| `menu_` | ButtonMenu, IconButtonMenu, AvatarMenu |
| `sw_` | Switch |
| `plot_` | Plot |

### 14.3 Default Answers

| Question | Answer |
|---|---|
| Default app shell? | NavigationRailLayout (admin) / NavigationDrawerLayout (customer) |
| Sidebar component? | Plain `Link` |
| Nav handler? | Lambda with variable capture |
| Internal navigation? | `open_form()` |
| Public URLs? | Routing dependency |
| Form binding? | `self.item` + Write Back ON |
| Default table access? | Server-only |
| Default structure? | Feature-based packages |
| Default test order? | Pure logic → backup → Uplink integration |
| Guided workflow panel? | Sidesheet |

---

**Status:** Consolidated specification and methods reference aligned to active Mybizz Anvil methods, M3 standards, `2-navigation-lambda-link-open-form.md` navigation, `9-multi-vertical-to-single-vertical-conversion.md` single-vertical scope, testing code of practice, startup pattern, packages/namespaces, Designer gotchas, uplink patterns, and background tasks.
