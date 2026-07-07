# `ui-customization-approach` ADR — UI Customization Approach

**Status:** Superseded by `htmltemplate-use` ADR  
**Date:** 2026-06-14  
**Superseded:** 2026-06-28 — NavigationRailLayout/NavigationDrawerLayout are now used per `htmltemplate-use` ADR. The project policy is Anvil Material 3 Theme compliance; native M3 Layouts are the standard for layout shells.  
**Authority:** Derived from platform-overview.md, `material-3-theme-component-scope` ADR, `design-rules` ADR

---

## Context

The platform needs a consistent, professional UI across all client instances. Anvil provides Material Design 3 (M3) as a theme dependency, a drag-and-drop designer, and the ability to modify `Standard-page.html` and `theme.css`. The question is how far to push Anvil's UI capabilities.

Three approaches were evaluated:

1. **Strict M3 theme** — Use M3 components and tokens as-is
2. **M3 with custom CSS overrides** — Modify `theme.css` and `Standard-page.html` for brand-specific styling
3. **MUI design styles without React** — Apply Material-UI (MUI) design patterns using Anvil's native capabilities

---

## Decision

### Primary Approach: M3 Theme with Controlled Customization

The platform uses the M3 theme dependency as the foundation. Customization is achieved through:

1. **M3 tokens** — Color palette, typography, spacing, and elevation are configured via M3 design tokens
2. **CSS overrides** — `theme.css` modifications for brand-specific styling that M3 tokens cannot express
3. **Standard-page.html** — Minimal modifications for layout structure (sidebar, content panel, header)
4. **Component composition** — Complex UIs built by composing M3 components, not by replacing them

### What M3 Provides

| Capability | M3 Support | Customization |
|---|---|---|
| Color system | Dynamic color, light/dark theme | Palette selector (`design-rules` ADR) |
| Typography | Material type scale | Font selection via tokens |
| Components | Buttons, cards, dialogs, data grids, etc. | Styling via CSS classes |
| Elevation | Shadow system | Token-based |
| Layout | Navigation drawer, rail, top app bar | Not used (custom layout, `navigation-lambda-link-open-form` ADR) |

### What Requires Custom Work

| Area | Approach |
|---|---|
| Admin layout sidebar | Custom HtmlTemplate + plain Link components (`navigation-lambda-link-open-form` ADR) |
| Content panel | ColumnPanel inside AdminLayout |
| Public page layouts | Anvil Routing dependency with custom layouts |
| Landing page templates | BlankLayout with template-specific design |
| Brand-specific colors | M3 palette tokens configured per client |

### Drag-and-Drop Designer Limitations

Anvil's drag-and-drop designer is fast for initial layout but fights precise positioning. The approach:

- Use the designer for initial component placement and structure
- Override layout details in code when precise control is needed
- Never fight the designer for more than 10 minutes — switch to code

### MUI Design Styles Evaluation

MUI (Material-UI) design patterns were evaluated as a potential source of design inspiration. The `sx` prop pattern (https://mui.com/system/getting-started/the-sx-prop/) was reviewed.

**Decision:** MUI design patterns are studied for inspiration but not directly implemented. Anvil's component model is different from React's — MUI's prop-based styling does not map cleanly. The visual outcomes (spacing, hierarchy, color usage) can be achieved through M3 tokens and CSS overrides.

---

## Consequences

- ✅ Consistent design language across all client instances via M3 tokens
- ✅ Brand customization achievable through palette tokens and CSS overrides
- ✅ No external UI framework dependency — stays within Anvil's native capabilities
- ✅ Designer usable for rapid prototyping; code used for precision
- ⚠️ Complex layouts require code overrides — designer is a starting point, not the final word
- ⚠️ M3 navigation components (NavigationDrawerLayout, NavigationRailLayout) — **now used per `htmltemplate-use` ADR** (this item superseded)
- ⚠️ CSS overrides must be maintained across M3 theme updates

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/material-3-theme-component-scope.md`` | Which M3 components are in scope |
| ``adr/adr-global/design-rules.md`` | Design rules and palette system |
| ``adr/adr-global/navigation-lambda-link-open-form.md`` | Why M3 navigation components are not used |
| `docs/ui-standards.md` | UI standards and component usage |
| `docs/spec-material-3-theme.md` | M3 theme specification |

---

*End of `ui-customization-approach` ADR*
