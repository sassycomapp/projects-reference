# 15 — Material 3 Theme Component Scope
Date: 2026-05-21
Status: Accepted
Source: M3 component mapping alignment task

---

## Context

The Mybizz CS application uses the official Anvil Material 3 Theme dependency (`4UK6WHQ6UX7AKELK`) as the foundational design system. This dependency provides a defined set of M3-compliant components, layouts, and design tokens.

---

## Decision

**Only components included in the Anvil Material 3 Theme dependency are permitted as native components.**

### Native M3 Components (Permitted)
Components explicitly provided by the M3 theme dependency:
- `m3.Button`, `m3.IconButton`, `m3.ToggleIconButton`
- `m3.TextBox`, `m3.TextArea`, `m3.DropdownMenu`
- `m3.Card`, `m3.InteractiveCard`, `m3.Avatar`
- `m3.Checkbox`, `m3.RadioButton`, `m3.Switch`, `m3.Slider`
- `m3.NavigationLink`, `m3.NavigationRailLayout`, `m3.NavigationDrawerLayout`
- `m3.Divider`, `m3.LinearProgressIndicator`, `m3.CircularProgressIndicator`
- `m3.ButtonMenu`, `m3.IconButtonMenu`, `m3.AvatarMenu`, `m3.MenuItem`
- All typography components (`m3.Text`, `m3.Heading`)
- All layout containers (`ColumnPanel`, `FlowPanel`, `RepeatingPanel`, `DataGrid`, `DataRowPanel`)

### Custom Composition (Permitted)
Components not natively provided by M3 must be composed from permitted primitives:
- **Chips** (FilterChip, InputChip, ActionChip): Built from `m3.Button` or `m3.Card` with custom styling
- **Snackbar/Toast**: Built from `Panel` + `m3.Text` with timed hide logic
- **Bottom Sheet**: Built from `Panel` + `m3.Card` with show/hide logic
- **Tabs/TabBar**: Built from grouped `m3.Button` instances with shared state
- **Date Picker**: Uses Anvil's classic `DatePicker` (not M3-styled) or custom composition

### Excluded (Not Permitted)
- Any component from Anvil Extras or other third-party packages
- Material Web Components (`@material/web`) unless explicitly approved via ADR
- Custom JavaScript frameworks or widget libraries
- Native HTML elements wrapped as Anvil components (unless part of M3 dependency)

---

## Implementation Rules

1. **Prefer native M3**: Always use native M3 components when available
2. **Document composition**: When composing custom components, document which M3 primitives are used
3. **Reference mapping**: Use `m3_component_mapping.md` as the authoritative reference for component availability
4. **No external imports**: Do not import UI components from external packages without ADR approval

---

## Consequences

### Fully supported (native M3)
All components in the M3 theme dependency are fully supported with no limitations.

### Partially supported (custom composition)
Composed components may have limitations in:
- Built-in animations/motions
- Auto-dismiss behavior (Snackbar)
- Native keyboard handling (Date Picker)
- Consistent styling across platforms

These limitations must be documented when the component is implemented.

### Not supported (excluded)
Components from excluded categories cannot be used without explicit ADR approval.

---

*End of `material-3-theme-component-scope` ADR*
