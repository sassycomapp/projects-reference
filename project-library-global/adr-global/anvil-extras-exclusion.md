# 14 — Anvil Extras Exclusion
Date: 2026-05-21
Status: Accepted
Source: M3 component mapping alignment task

---

## Context

Anvil Extras is a third-party package that provides additional components and utilities for Anvil applications. The Mybizz CS application uses the official Anvil Material 3 Theme dependency (`4UK6WHQ6UX7AKELK`) as the sole UI framework.

---

## Decision

**Anvil Extras is out of scope and excluded from the Mybizz CS project.**

This exclusion applies to:
- All Anvil Extras components (Tabs, MultiSelectDropDown, Autocomplete, Quill, Switch, Slider, RadioGroup, CheckBoxGroup, Popover, etc.)
- Any third-party packages not included in the official Anvil Material 3 Theme dependency
- Any custom JavaScript frameworks or non-standard widget dependencies

**Native M3 components** provided by the Anvil Material 3 Theme dependency are the only permitted UI components. If a required component is not available in the M3 theme, it must be:
1. Composed from available M3 primitives (e.g., Chips from Buttons/Cards)
2. Built as a Custom Component using native M3 components
3. Rejected as out of scope if composition is not feasible

---

## Rationale

1. **CSS conflicts**: Mixing Anvil Extras with M3 produces hard-to-diagnose CSS conflicts
2. **Maintenance burden**: Third-party packages introduce external dependencies and update risks
3. **Consistency**: Using only M3 components ensures consistent styling and behavior
4. **Support**: Official Anvil components have direct support from Anvil.works

---

## Consequences

### Blocked components (always excluded)
- Tabs, MultiSelectDropDown, Autocomplete, Quill, Switch, Slider, RadioGroup, CheckBoxGroup, Popover
- Any component not in the official Anvil Material 3 Theme dependency

### Allowed patterns
- Native M3 components from the theme dependency
- Custom Components built from native M3 primitives
- Material Web Components only if explicitly required and approved via ADR

---

*End of `anvil-extras-exclusion` ADR*
