# Navigation Standard: Lambda/Link/open_form

**Date:** 2026-03-18
**Status:** Accepted (partially superseded — see below)
**Source:** Devlog #29 — task-navigation-ruling-propagation

---

## Context

The Mybizz applications use layout shells for authenticated navigation. This ADR established the navigation pattern. Since then, the `htmltemplate-use` ADR has banned `HTMLTemplate` and mandated native M3 Layouts (`NavigationRailLayout`, `NavigationDrawerLayout`) for all layout-shell cases.

This ADR remains authoritative for the **navigation pattern** (lambda click handlers, Link components, open_form). The **layout shell decision** (custom HtmlTemplate vs native M3 Layouts) is governed by the `htmltemplate-use` ADR.

---

## Decision

### Still binding: the navigation pattern

**Sidebar navigation** in AdminLayout and CustomerLayout uses:
- Plain `Link` components in the sidebar — not `NavigationLink`
- Lambda click handlers with **mandatory loop variable capture**:

```python
lambda e, fn=form_name, a=attr_name: self._nav_click(fn, a)
```

The `fn=` and `a=` keyword arguments in the lambda are not optional. Omitting them
causes all nav items to capture the last value of the loop variable — a silent bug.

**Parameterised navigation** (row clicks, detail forms with IDs) uses `open_form()`
called directly:

```python
open_form('BookingViewerForm', booking_id=row['booking_id'])
```

**`navigate_to` is never used.** It is not compatible with the content panel navigation pattern.
**`NavigationLink` is not used for sidebar navigation.** The `nav_` prefix in component naming
refers to Link components styled for sidebar use — not NavigationLink components.

### Superseded by `htmltemplate-use` ADR

The following parts of the original decision are superseded:

| Original decision | Superseded by |
|---|---|
| Custom `HtmlTemplate` layouts for layout shells | `htmltemplate-use` ADR — HTMLTemplate banned, native M3 Layouts required |
| `NavigationDrawerLayoutTemplate` is not used | `htmltemplate-use` ADR — NavigationRailLayout/NavigationDrawerLayout are now standard |

The lambda/Link/open_form navigation pattern remains unchanged and binding.

---

## Consequences

### Rules files requiring verification during refactor

All files below must be checked during their refactor task to confirm the ruling is
correctly applied. Do not blindly reapply — verify first, then correct only what is wrong.

| File | Note |
|---|---|
| `spec_ui_standards.md` | §1 rewritten in devlog #29 — verify correct |
| `spec_ui_standards_forms.md` | Corrected in devlog #29 — verify correct |
| `ref_anvil_navigation.md` | Corrected in devlog #29 — verify correct |
| `policy_development.md` | Corrected in devlog #29 — verify correct |
| `platform_overview.md` | §5.3 — verify nav ruling applied |
| `spec_architecture.md` | Navigation section — verify nav ruling applied |
| `platform_docmap.md` | §2 critical facts — verify nav ruling applied |

---

*Navigation Standard v2.0 — Layout shell decision superseded by `htmltemplate-use` ADR. Navigation pattern (lambda/Link/open_form) unchanged and binding.*
