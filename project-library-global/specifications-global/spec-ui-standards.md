# Mybizz CS — UI Standards

**UI Framework:** Material Design 3 (M3) — Dependency ID: 4UK6WHQ6UX7AKELK
**Authority:** Mandatory — all UI work uses M3 exclusively
**MUI Aesthetic Overlay:** See `DESIGN.md` Section "MUI Aesthetic Implementation" for the MUI aesthetic variables applied on top of M3.

---

## 1. The M3 Mandate

All user interface work uses Material Design 3 exclusively. The M3 theme dependency (`4UK6WHQ6UX7AKELK`) is the only UI framework. Mixing M3 with non-M3 components produces CSS conflicts that are difficult to diagnose.

**Native M3 components** (provided by the M3 theme dependency) are fully supported:
- `m3.Switch`, `m3.Slider`, `m3.RadioButton`, `m3.Checkbox`
- `m3.Button`, `m3.IconButton`, `m3.NavigationLink`
- All input components (`m3.TextBox`, `m3.TextArea`, `m3.DropdownMenu`)
- All layout components (`ColumnPanel`, `FlowPanel`, `DataGrid`, `Card`)

**Partially supported M3 components** (use custom composition):
- Chips, Snackbar, Bottom Sheet, Tabs, Date Picker

These require custom composition via Panels, Cards, and Buttons. See `m3_component_mapping.md` for implementation guidance.

When a component appears absent, compose from available M3 primitives — do not import from outside the M3 system.

---

## 2. Custom Component vs Widget

**Prefer Custom Components** (90% of cases). Custom Components are Python-only, use Anvil components internally, support data bindings, and are Designer-friendly.

**Use Widgets only** (10% of cases) when wrapping external JS/CSS or browser-level behavior (maps, rich code editors, canvas rendering).

**Best practice:** If a Widget is unavoidable, hide it behind a Custom Component interface — the Custom Component handles layout, data, and events; the Widget handles only the specific DOM or JS requirement.

---

## 3. Component Naming

All components follow the prefix convention:

| Component | Prefix | Example |
|---|---|---|
| Button | `btn_` | `btn_save`, `btn_cancel` |
| TextBox / TextArea | `txt_` | `txt_customer_name`, `txt_notes` |
| Heading | `hdg_` | `hdg_page_title` |
| DropdownMenu | `dd_` | `dd_status` |
| DatePicker | `dp_` | `dp_booking_date` |
| FileLoader | `fu_` | `fu_upload` |
| Link / Navigation | `nav_` | `nav_dashboard` |
| Checkbox | `cb_` | `cb_agree_terms` |
| RadioButton | `rb_` | `rb_payment_method` |
| ColumnPanel | `col_` | `col_main` |
| FlowPanel | `flow_` | `flow_header` |
| Card | `card_` | `card_details` |
| DataGrid | `dg_` | `dg_customers` |
| RepeatingPanel | `rp_` | `rp_items` |
| Plot | `plot_` | `plot_revenue` |

Good-to-have prefixes apply in complex forms with 15+ components.

---

## 4. Designer-Only Properties

Two categories of properties cannot be set in Python code:

**TextBox password masking:** Set via `hide_text = True` in the Designer Properties Panel. There is no `type` property on Anvil TextBox.

**TextBox outlined style:** Set via the `appearance` property in the Designer. NOT via `role`.

**Write Back toggle (W):** Must be enabled in the Designer Data Bindings panel. Cannot be set in code. Required for the `self.item` pattern.

---

## 6. Form Types and Layout Patterns

**List forms** (ContactListForm, BookingListForm): Heading (headline-large) for page title, horizontal FlowPanel header with search TextBox and filter DropdownMenu, filled Button for primary action, DataGrid for list body with IconButton row actions.

**Editor forms** (ContactEditorForm, ServiceEditorForm): Card (outlined) as container, Heading (headline-small) for section headers, outlined TextBox/TextArea for inputs, outlined DropdownMenu/DatePicker for selections, FlowPanel action row with filled Save Button and outlined Cancel Button.

**Dashboard forms:** Heading (headline-large), FlowPanel for metrics row holding elevated Cards, Plot components for charts, DataGrid for summary tables.

**Authentication forms:** Custom Form with no layout wrapper, outlined Card as centred container, outlined TextBox components (password type set in Designer), filled Button for submit, Link components for forgot password and terms/privacy navigation.

---

## 7. Data Binding — the self.item Pattern

All editor forms use Anvil's data binding system with `self.item` as the data container.

- `self.item` must be set before `self.init_components()` is called
- Data bindings are configured in the Designer with Write Back enabled
- With Write Back enabled, the component automatically updates `self.item` on focus loss — no change event handlers needed
- When `self.item` is reassigned entirely, data bindings refresh automatically
- When fields of `self.item` are modified in place, call `refresh_data_bindings()` manually

**Do NOT use data bindings on settings/config forms** that manage their own read/write explicitly via server calls. Data bindings fight with this pattern.

---

## 8. Colour and Typography

All colours use the `theme:` prefix. Hardcoded hex or RGB values are never used. Typography uses Text and Heading components with M3 style and scale properties — the legacy Label component is not used in M3 apps.

Button hierarchy: one filled button per screen for the primary action, outlined buttons for secondary actions, text buttons for tertiary actions.

All form input components use `role='outlined'`. Validation failure state uses `role='outlined-error'` with the error message in the placeholder. The field is restored to `role='outlined'` on valid input.

---

*End of file*
