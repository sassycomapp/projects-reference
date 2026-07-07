***

Material Design 3 (M3) — Dependency ID: 4UK6WHQ6UX7AKE

# m3_component_mapping.md

Title: M3 → Anvil component mapping (Python style)
Version: 1.0  
Generated: 2026-05-21  
Authority: reference for developers, code generators, and automation agents

## Purpose

This document is a canonical, machine‑ and human‑readable mapping from Material Design 3 UI concepts to the exact Anvil Material‑3 dependency classes, properties, and usage patterns your codebase should use. It is intentionally orthogonal to architecture/ADR documents (navigation patterns, layout rules, roles, etc.) and is meant to be referenced by code generators, CI linters, Opencode/GStack agents, and developers when composing UI.

Authoritative dependency: Anvil Material 3 theme and components (install dependency ID `4UK6WHQ6UX7AKELK`). See the official component reference for details: Anvil Material 3 Components. [Anvil M3 components](https://anvil.works/docs/components/material-3/components)

Usage note for agents:
- This file is the single source of truth for component ↔ class mappings.
- Do not embed full policy or ADR text here; link to ADRs instead.
- Each mapping row is one atomic mapping; automated tools should parse lines beginning with code fences or `m3.` prefixes.

***

## Conventions (for readers and agents)

- Python import pattern:
  - import m3.components as m3
- Support levels:
  - Fully supported — direct Anvil `m3` class or layout exists and implements the M3 behavior.
  - Partially supported — can be implemented with native `m3` components plus small layout/CSS work; some M3 behaviors or motions missing.
  - Not supported — requires external libraries, custom JS, or is not feasible with native Anvil components.
- Where applicable, example usage is shown as a single-line Python snippet.
- Keep references minimal; the primary Anvil M3 docs link above is canonical.

***

## Mapping table (grouped by M3 category)

### Typography & Text
- Body text / label
  - Mapping: `m3.Text(text="...", style="body", scale="medium")`
  - Support: Fully supported
  - Notes: `style` ∈ {"body","label"}; `scale` ∈ {"small","medium","large"}.

- Display / Headline / Title
  - Mapping: `m3.Heading(text="...", style="headline", scale="large")`
  - Support: Fully supported
  - Notes: `style` ∈ {"display","headline","title"}.

- Link / Anchor
  - Mapping: `m3.Link(text="...", url="https://...", click=...)`
  - Support: Fully supported
  - Notes: Use `url` to open external links or `click` for in‑app handlers.

### Buttons & Actions
- Elevated Button
  - Mapping: `m3.Button(text="...", appearance="elevated")`
  - Support: Fully supported

- Filled Button
  - Mapping: `m3.Button(text="...", appearance="filled")`
  - Support: Fully supported

- Tonal Button
  - Mapping: `m3.Button(text="...", appearance="tonal")`
  - Support: Fully supported

- Outlined Button
  - Mapping: `m3.Button(text="...", appearance="outlined")`
  - Support: Fully supported

- Text Button
  - Mapping: `m3.Button(text="...", appearance="text")`
  - Support: Fully supported

- Icon Button
  - Mapping: `m3.IconButton(icon="mi:icon_name", appearance="outlined")`
  - Support: Fully supported
  - Notes: Use the `mi:` prefix for Material Icons (e.g. `mi:notifications`, `mi:account_circle`). SVG where supported.

- Toggle Icon Button
  - Mapping: `m3.ToggleIconButton(icon="mi:icon_name", selected=False)`
  - Support: Fully supported

- Icon (decorative / standalone)
  - Mapping: `m3.IconButton(icon="mi:icon_name")`
  - Support: Fully supported
  - Notes: For non-interactive, decorative icon use (e.g. empty-state illustrations). Leave `enabled` at default (`True`); do not bind a `click` event. No hover or click action is intended. Do **not** use `enabled=False` — keep the icon at full opacity and normal color. Use the `mi:` prefix to indicate Material Icons (e.g. `mi:bar_chart`, `mi:receipt`, `mi:event`).

- Floating Action Button (FAB)
  - Mapping: `m3.Button(... )` positioned as FAB via layout/CSS
  - Support: Partially supported
  - Notes: No dedicated floating behavior; position with `absolute` panel or container.

- Extended FAB
  - Mapping: `m3.Button(icon="mi:add", text="New", appearance="filled")` (styled)
  - Support: Partially supported

### Surfaces & Containers
- Card (elevated / filled / outlined)
  - Mapping: `m3.Card(appearance="elevated"|"filled"|"outlined", orientation="row"|"column")`
  - Support: Fully supported

- Interactive Card
  - Mapping: `m3.InteractiveCard(click=..., enabled=True, appearance="elevated")`
  - Support: Fully supported

- Sheet / Surface container (bottom sheet)
  - Mapping: stacked `Panel` + `m3.Card` with show/hide logic
  - Support: Partially supported
  - Notes: No native modal sheet behavior; approximate with panels/forms.

- Avatar
  - Mapping: `m3.Avatar(image=..., user_name="...", appearance="filled")`
  - Support: Fully supported

### Navigation
- Navigation Rail
  - Mapping: `m3.NavigationLink` placed in `Navigation Rail Layout`
  - Support: Fully supported
  - Example: `m3.NavigationLink(text="Home", icon="mi:home", navigate_to=HomeForm)`

- Navigation Drawer
  - Mapping: `m3.NavigationLink` placed in `Navigation Drawer Layout`
  - Support: Fully supported

- Navigation Bar (bottom)
  - Mapping: horizontal `Panel` + `m3.Button` / `m3.NavigationLink` items
  - Support: Partially supported
  - Notes: No dedicated bottom navigation component built into Anvil M3.

- Tabs / Tab bar
  - Mapping: custom `Panel` + `m3.Button` group handling selected state
  - Support: Partially supported

- Segmented Button
  - Mapping: grouped `m3.Button` instances with shared state
  - Support: Partially supported

### Menus & Popups
- Button Menu (dropdown)
  - Mapping: `m3.ButtonMenu(text="...", menu_items=[m3.MenuItem(...), ...])`
  - Support: Fully supported

- Icon Button Menu
  - Mapping: `m3.IconButtonMenu(icon="mi:more_vert", menu_items=[...])`
  - Support: Fully supported

- Avatar Menu (user menu)
  - Mapping: `m3.AvatarMenu(image=..., menu_items=[...])`
  - Support: Fully supported

- Menu Item
  - Mapping: `m3.MenuItem(text="...", leading_icon="mi:...", trailing_icon="mi:...")`
  - Support: Fully supported

- Dialog / Modal
  - Mapping: `anvil.alert()` or custom modal `Form` with `show()` / `close()`
  - Support: Partially supported
  - Notes: Styling and M3 motion/structure may differ from canonical M3 Dialog.

### Input Controls (Form)
- Text Field (filled)
  - Mapping: `m3.TextBox(appearance="filled", label_text="...")`
  - Support: Fully supported

- Text Field (outlined)
  - Mapping: `m3.TextBox(appearance="outlined", label_text="...")`
  - Support: Fully supported

- Text Area (filled)
  - Mapping: `m3.TextArea(appearance="filled", label_text="...")`
  - Support: Fully supported

- Text Area (outlined)
  - Mapping: `m3.TextArea(appearance="outlined", label_text="...")`
  - Support: Fully supported

- Dropdown Menu
  - Mapping: `m3.DropdownMenu(items=[...], label_text="...")`
  - Support: Fully supported

- Checkbox
  - Mapping: `m3.Checkbox(text="...", checked=False, allow_indeterminate=True)`
  - Support: Fully supported

- Radio Button
  - Mapping: `m3.RadioButton(text="...", value="...")` (use in `m3.RadioGroupPanel`)
  - Support: Fully supported

- Radio Group Panel
  - Mapping: `m3.RadioGroupPanel(buttons=[...])`
  - Support: Fully supported

- Switch
  - Mapping: `m3.Switch(selected=False, change=...)`
  - Support: Fully supported

- Slider
  - Mapping: `m3.Slider(min=0, max=100, step=1, value=50)`
  - Support: Fully supported

- File Input / FileLoader
  - Mapping: `m3.FileLoader(text="Upload", change=...)`
  - Support: Fully supported

- Date Picker
  - Mapping: Anvil `DatePicker` (not M3 styled) or custom
  - Support: Partially supported
  - Notes: Anvil has classic DatePicker but Material 3 style calendar is not native. Supports Designer writeback (`selected_date` property).

- Time Picker
  - Mapping: custom `TextBox` or external widget
  - Support: Not supported

### Selection Controls
- Checkbox (multi)
  - Mapping: `m3.Checkbox` items in `RepeatingPanel` or `Column`
  - Support: Fully supported

- Radio Button (single)
  - Mapping: `m3.RadioButton` in `m3.RadioGroupPanel`
  - Support: Fully supported

- Switch
  - Mapping: `m3.Switch`
  - Support: Fully supported

- Chips (filter / input)
  - Mapping: custom composed `m3.Button` or small `m3.Card`
  - Support: Partially supported
  - Notes: No native `Chip` components; implement with buttons/cards and states.

### Feedback & Status
- Linear Progress Indicator
  - Mapping: `m3.LinearProgressIndicator(type="indeterminate"|"determinate", progress=%)`
  - Support: Fully supported

- Circular Progress Indicator
  - Mapping: `m3.CircularProgressIndicator(type="indeterminate"|"determinate", progress=%)`
  - Support: Fully supported

- Snackbar / Toast
  - Mapping: floating `Panel` + `m3.Text` + timed hide
  - Support: Partially supported
  - Notes: No native snackbar component; approximate with panels or custom forms.

- Alert / Confirmation
  - Mapping: `anvil.alert()` or custom modal `Form`
  - Support: Partially supported

### Layout & Dividers
- Divider (full width / inset)
  - Mapping: `m3.Divider(type="full_width"|"inset")`
  - Support: Fully supported

- List / List Item
  - Mapping: `RepeatingPanel` with `m3.Card` or `m3.InteractiveCard` as item template
  - Support: Partially supported
  - Notes: No native `ListItem` component; assemble via repeating templates.

- App Bar / Top App Bar
  - Mapping: `Panel` containing `m3.Text`, `m3.IconButton` (decorative or interactive), etc.
  - Support: Partially supported
  - Notes: Compose with M3 components; no dedicated TopAppBar component.

- Navigation Rail Layout
  - Mapping: `Navigation Rail Layout` template (provided by dependency)
  - Support: Fully supported

- Navigation Drawer Layout
  - Mapping: `Navigation Drawer Layout` template (provided)
  - Support: Fully supported

- DataGrid Row Template (DataRowPanel)
  - Mapping: `DataRowPanel` (used inside DataGrid for custom row layouts)
  - Support: Fully supported
  - Notes: Classic Anvil component. Defines the layout of each row in a DataGrid. Use `drp_` prefix in wireframes. One component per column slot.

### Not supported (native)
List of Material 3 components that are not implementable as native Anvil M3 components without external libraries, custom JS, or extensive custom CSS/motion:
- Material 3 Tabs (native TabBar) — Partially supported via custom panels.
- Material 3 Bottom Sheet (native sliding sheet) — Partially supported (approximate).
- Native Material 3 Date Picker / Time Picker (Material web calendar/clock) — Not supported (calendar exists in Anvil but not M3-style).
- Native Chip set (FilterChip / InputChip) — Partially supported via custom composition.
- Native Snackbar with built-in auto-dismiss & motion — Partially supported via custom panels.

### Designer Data Binding Writeback Support

The following components support the Designer "Write Back" (W) property in the Data Bindings panel. When (W) is enabled, the component's value is automatically bound to a data table column. This is a Designer-only property — it cannot be set or overridden programmatically.

| Component | Writeback Property |
|---|---|
| TextBox | `text` |
| TextArea | `text` |
| Checkbox | `checked` |
| Switch | `selected` |
| RadioGroupPanel | `selected_value` |
| Slider | `value` |
| DropdownMenu | `selected_value` |
| DatePicker | `selected_date` |

**Not writeback-capable** (do NOT add (W) even if they accept user input):
- RadioButton — configured via RadioGroupPanel, not individually
- FileLoader — file selection is event-driven, not data-bound

**Display-only** (never require writeback):
- Text, Link, RichText, Heading
- LinearProgressIndicator, CircularProgressIndicator

Wireframes must annotate all writeback-capable components with `(W)` in the diagram and include a `writeback = W` row in the Properties Table.

***

## Examples (agent / developer snippets)

- Create a filled button:
```python
import m3.components as m3
save_btn = m3.Button(text="Save", appearance="filled")
```

- Create an outlined text field with a leading icon:
```python
email = m3.TextBox(appearance="outlined", label_text="Email", leading_icon="mi:email")
```

- Create an interactive card:
```python
card = m3.InteractiveCard(appearance="elevated", click=self.open_details)
```

- Create a navigation link (in Navigation Drawer Layout):
```python
nav = m3.NavigationLink(text="Dashboard", icon="mi:dashboard", navigate_to=DashboardForm)
```

***

## Agent consumption guidelines

- Agents should parse mappings by extracting `m3.` tokens and the support level that follows the mapping block.
- If a spec or PR tries to reference a component not present here, agents should:
  - If mapping shows Partially supported → create an action item for a human reviewer.
  - If mapping shows Not supported → block or flag the change and request a justification or external widget wrapper.
- When generating code, prefer Fully supported mappings. For Partials, generate a comment indicating which behaviors are missing and link to the ADR or policy file that governs allowed approximations.

***

## Change log
- v1.0 — 2026-05-21 — Initial mapping extraction for Mybizz codebase.

***

End of file.