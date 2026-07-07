# `dark-mode-v1` ADR: Dark Mode — V1 Inclusion

## Status

**Accepted**

## Context

Dark mode was previously deferred from V1 to V2 across multiple project documents:

- `docs/DESIGN.md` line 87: "Dark mode as the primary design target. (Dark mode is deferred to V2, not rejected.)"
- `wip/anvil-ui-build-standard.md` §8: "Dark mode: Deferred from V1 to V2 — don't build dark-mode variants unless explicitly asked."

However, screen production practice already requires `@media (prefers-color-scheme: dark)` blocks with token overrides on every screen. The CSS architecture for dark mode is already built into the token system — each palette's 32-token set includes surface/neutral tokens that can be swapped for dark mode values. The deferral creates a contradiction between the written standard and the actual production workflow.

The token-driven palette system makes dark mode implementation straightforward: dark mode is simply another set of token overrides, not a separate design system.

## Decision

**Dark mode is incorporated into V1.** It is no longer deferred to V2.

All screens produced for V1 must include a `@media (prefers-color-scheme: dark)` block with alternate token values for the following CSS custom properties:

- `--surface`
- `--on-surface`
- `--surface-variant`
- `--on-surface-variant`
- `--outline`
- `--outline-variant`
- `--surface-container-low`

Dark mode token values are palette-specific. Each of the 4 palettes must define its own dark mode surface/neutral tokens.

## Supersedes

The following references to dark mode deferral are superseded by this ADR:

| Document | Line | Old Text | Status |
|---|---|---|---|
| `docs/DESIGN.md` | 87 | "Dark mode as the primary design target. (Dark mode is deferred to V2, not rejected.)" | **Superseded** |
| `wip/anvil-ui-build-standard.md` | §8 | "Dark mode: Deferred from V1 to V2" | **Superseded** |
| `docs/internal-standards-anvil-wireframe-conversion` | §4.2 | "Dark mode deferred from V1 design direction" | **Superseded** |

## Consequences

- All screen HTML files must include `@media (prefers-color-scheme: dark)` blocks.
- `theme_patched.css` must include dark mode token overrides for all 4 palettes.
- `DESIGN_palette_section.md` must define dark mode tokens alongside light mode tokens.
- The compliance checklist item "Dark mode" changes from "N/A (deferred)" to a required verification item.
- V2 roadmap must remove dark mode from its feature list.

## Related Documents

| Document | Relationship |
|---|---|
| `docs/DESIGN.md` | Contains superseded dark mode deferral language |
| `wip/anvil-ui-build-standard.md` §8 | Contains superseded dark mode deferral language |
| `docs/DESIGN_palette_section.md` | Must be updated with dark mode tokens |
| `docs/theme_patched.css` | Must be updated with dark mode overrides |

---

*End of `dark-mode-v1` ADR*
