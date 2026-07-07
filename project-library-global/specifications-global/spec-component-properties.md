# Mybizz CS — Component Properties Specification

**Vertical:** Consulting & Services  
**Platform:** Anvil.works with Material Design 3 (M3)  
**Purpose:** Definitive list of all component properties with Designer vs Code classification

---

## Classification Rules

- **Designer**: Property can ONLY be set in Anvil Designer. Cannot be set programmatically in Python.
- **Code**: Property can be set in Python code (`self.component.property = value`) or via CSS.

---

## Button Components

### Button, IconButton, Outlined Button

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| text | Code | "" | Button label text |
| enabled | Code | True | Whether button is clickable |
| visible | Code | True | Whether button is visible |
| role | Code | "filled" | M3 role: filled, outlined, text, tonal |
| icon | Code | None | Icon name using `mi:` prefix (e.g., `mi:save`, `mi:delete`) |
| icon_alignment | Code | "leading" | "leading" or "trailing" |
| on_click | Code | None | Event handler function |

**Designer-Only Properties:**
- None — All Button properties can be set in code

---

## TextBox Components

### TextBox

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| value | Code | "" | Text content |
| enabled | Code | True | Whether input is editable |
| visible | Code | True | Whether component is visible |
| placeholder | Code | "" | Placeholder text |
| readonly | Code | False | Whether input is read-only |
| password | Code | False | Whether text is masked |
| hide_text | Designer | False | **Designer-only**: Password masking toggle (W) |
| outlined | Code | True | M3 appearance: outlined or filled |
| on_change | Code | None | Event handler on value change |
| on_enter | Code | None | Event handler on Enter key |

**Designer-Only Properties:**
- **hide_text** — Password masking must be enabled in Designer Data Bindings panel. Cannot be set in code.

---

## TextArea Components

### TextArea

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| value | Code | "" | Text content |
| enabled | Code | True | Whether input is editable |
| visible | Code | True | Whether component is visible |
| placeholder | Code | "" | Placeholder text |
| readonly | Code | False | Whether input is read-only |
| rows | Code | 4 | Number of visible rows |
| outlined | Code | True | M3 appearance: outlined or filled |
| on_change | Code | None | Event handler on value change |

**Designer-Only Properties:**
- None — All TextArea properties can be set in code

---

## Heading Components

### Heading

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| text | Code | "" | Heading text |
| enabled | Code | True | Whether component is enabled |
| visible | Code | True | Whether component is visible |
| role | Code | "headline-small" | M3 semantic role: headline-large, headline-medium, headline-small, title-large, title-medium, title-small, body-large, body-medium, body-small, label-large, label-medium, label-small |
| on_click | Code | None | Event handler on click |

**Designer-Only Properties:**
- None — All Heading properties can be set in code

---

## DropdownMenu Components

### DropdownMenu

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| value | Code | None | Selected item value |
| items | Code | [] | List of available items |
| enabled | Code | True | Whether dropdown is enabled |
| visible | Code | True | Whether component is visible |
| outlined | Code | True | M3 appearance: outlined or filled |
| placeholder | Code | "" | Placeholder text when no selection |
| on_change | Code | None | Event handler on selection change |
| hide_text | Designer | False | **Designer-only**: Write Back toggle (W) |

**Designer-Only Properties:**
- **hide_text** — Write Back toggle must be enabled in Designer for data binding. Cannot be set in code.

---

## DatePicker Components

### DatePicker

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| value | Code | None | Selected date (datetime object) |
| enabled | Code | True | Whether picker is enabled |
| visible | Code | True | Whether component is visible |
| outlined | Code | True | M3 appearance: outlined or filled |
| min_date | Code | None | Minimum selectable date |
| max_date | Code | None | Maximum selectable date |
| on_change | Code | None | Event handler on date change |
| hide_text | Designer | False | **Designer-only**: Write Back toggle (W) |

**Designer-Only Properties:**
- **hide_text** — Write Back toggle must be enabled in Designer for data binding. Cannot be set in code.

---

## Panel Components

### ColumnPanel

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| visible | Code | True | Whether panel is visible |
| horizontal | Code | False | Layout orientation: False = vertical, True = horizontal |

**Designer-Only Properties:**
- None — All ColumnPanel properties can be set in code

### FlowPanel

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| visible | Code | True | Whether panel is visible |
| horizontal | Code | False | Layout orientation: False = vertical, True = horizontal |

**Designer-Only Properties:**
- None — All FlowPanel properties can be set in code

### LinearPanel (Deprecated)

**Note:** LinearPanel is deprecated. Use FlowPanel instead.

---

## Card Components

### Card

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| visible | Code | True | Whether card is visible |
| role | Code | "outlined" | M3 role: outlined, filled, tonal, elevated |
| on_click | Code | None | Event handler on click |

**Designer-Only Properties:**
- None — All Card properties can be set in code

### CardContentContainer

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| visible | Code | True | Whether container is visible |

**Designer-Only Properties:**
- None — Auto-created inside Card, cannot be modified

---

## DataGrid Components

### DataGrid

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| items | Code | [] | List of row data |
| columns | Code | [] | Column definitions |
| visible | Code | True | Whether grid is visible |
| role | Code | "tonal-data-grid" | M3 appearance role |
| row_actions | Code | [] | List of action buttons per row |
| on_row_click | Code | None | Event handler on row click |
| on_row_action | Code | None | Event handler on row action click |

**Designer-Only Properties:**
- None — All DataGrid properties can be set in code

---

## RepeatingPanel Components

### RepeatingPanel

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| items | Code | [] | List of data items |
| visible | Code | True | Whether panel is visible |
| template | Designer | None | **Designer-only**: Template form for repeated items |
| on_item_click | Code | None | Event handler on item click |

**Designer-Only Properties:**
- **template** — Template form must be set in Designer. Cannot be set in code.

---

## InteractiveCard Components

### InteractiveCard

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| visible | Code | True | Whether card is visible |
| role | Code | "outlined-card" | M3 role: outlined-card, filled-card, elevated-card. Note: `tonal-card` is NOT a valid InteractiveCard appearance. |
| on_click | Code | None | Event handler on click |

**Designer-Only Properties:**
- None — All InteractiveCard properties can be set in code

---

## FileLoader Components

### FileLoader

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| media | Code | None | Uploaded media object |
| enabled | Code | True | Whether loader is enabled |
| visible | Code | True | Whether component is visible |
| outlined | Code | True | M3 appearance: outlined or filled |
| accept | Code | "*" | File type filter (e.g., "image/*", ".pdf") |
| max_size | Code | None | Maximum file size in bytes |
| on_upload | Code | None | Event handler on upload complete |

**Designer-Only Properties:**
- None — All FileLoader properties can be set in code

---

## Link Components

### Link

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| text | Code | "" | Link text |
| url | Code | "" | Target URL |
| enabled | Code | True | Whether link is clickable |
| visible | Code | True | Whether link is visible |
| on_click | Code | None | Event handler on click |

**Designer-Only Properties:**
- None — All Link properties can be set in code

---

## Checkbox Components

### Checkbox

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| value | Code | False | Whether checkbox is checked |
| enabled | Code | True | Whether checkbox is enabled |
| visible | Code | True | Whether component is visible |
| label | Code | "" | Checkbox label text |
| on_change | Code | None | Event handler on change |

**Designer-Only Properties:**
- None — All Checkbox properties can be set in code

---

## RadioButton Components

### RadioButton

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| value | Code | False | Whether radio button is selected |
| enabled | Code | True | Whether radio button is enabled |
| visible | Code | True | Whether component is visible |
| label | Code | "" | Radio button label text |
| group_name | Code | "" | Radio group name (for mutual exclusion) |
| on_change | Code | None | Event handler on change |

**Designer-Only Properties:**
- None — All RadioButton properties can be set in code

---

## HTMLTemplate Components

### HTMLTemplate — BANNED (`htmltemplate-use` ADR)

**HTMLTemplate is banned.** Use stock Anvil M3 components and the two native M3 Layouts (NavigationRailLayout, NavigationDrawerLayout) in all cases. See ``adr/adr-global/htmltemplate-use.md`` for the full decision.

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| html | Code | "" | HTML string content |
| visible | Code | True | Whether template is visible |
| anvil_slot_repeat | Designer | None | **Designer-only**: Slot name for repeating content |

**Designer-Only Properties:**
- **anvil_slot_repeat** — Slot name for dynamic content injection. Must be set in Designer.

---

## SidesheetContent Components

### SidesheetContent — NOT APPROVED

**SidesheetContent is not approved for use in this project.** It is not a standard Anvil M3 component and has not been evaluated for compliance.

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| visible | Code | True | Whether sidesheet is visible |
| position | Code | "right" | "left" or "right" |
| modal | Code | False | Whether sidesheet is modal (blocks background) |

**Designer-Only Properties:**
- None — All SidesheetContent properties can be set in code

---

## Plot Components

### Plot

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| data | Code | None | Plot data configuration |
| visible | Code | True | Whether plot is visible |
| width | Code | 600 | Plot width in pixels |
| height | Code | 400 | Plot height in pixels |
| on_click | Code | None | Event handler on plot click |

**Designer-Only Properties:**
- None — All Plot properties can be set in code

---

## ProgressIndicator Components

### LinearProgressIndicator, CircularProgressIndicator

| Property | Classification | Default | Description |
|----------|---------------|---------|-------------|
| visible | Code | True | Whether indicator is visible |
| indeterminate | Code | True | Whether indicator is indeterminate (spinning) |
| progress | Code | 0.0 | Progress value (0.0 to 1.0) for determinate mode |

**Designer-Only Properties:**
- None — All ProgressIndicator properties can be set in code

---

## Summary: Designer-Only Properties

The following properties can ONLY be set in Anvil Designer and cannot be set programmatically:

| Component | Property | Purpose |
|-----------|----------|---------|
| TextBox | hide_text | Password masking toggle (W) |
| DropdownMenu | hide_text | Write Back toggle (W) |
| DatePicker | hide_text | Write Back toggle (W) |
| RepeatingPanel | template | Template form assignment |

**Critical Note:** The Write Back toggle (hide_text) must be enabled in the Designer Data Bindings panel for all text input components (TextBox, TextArea, DropdownMenu, DatePicker) to support the `self.item` data binding pattern. This toggle cannot be set in Python code.

---

## Summary: Writeback-Capable Components

The following components support two-way data bindings (Write Back) in the Designer Data Bindings panel. When Write Back is enabled, the component's value is automatically written back to the bound data source when relevant input events fire (e.g., `pressed_enter`, `lost_focus`).

| Component | Writeback Property | Source |
|---|---|---|
| TextBox | `text` | Anvil Data Bindings docs |
| TextArea | `text` | Anvil Data Bindings docs |
| DropdownMenu | `selected_value` | Anvil M3 docs |
| DatePicker | `selected_date` | Anvil Data Bindings docs |
| Checkbox | `checked` | Anvil M3 docs |
| Switch | `selected` | Anvil M3 docs |
| RadioGroupPanel | `selected_value` | Anvil M3 docs |
| Slider | `value` | Anvil M3 docs |

**Not writeback-capable:** RadioButton, FileLoader. Display-only: Text, Link, RichText, Heading, LinearProgressIndicator, CircularProgressIndicator.

---

*End of file — spec-component-properties.md*
