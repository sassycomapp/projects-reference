# `responsive-behaviour-mechanism` ADR: Responsive Behaviour Mechanism

## Status

Accepted

## Context

`design-direction.md` contains two incompatible breakpoint models in the same document:

1. A binary split at 998px.
2. A three-tier system (640px/1024px) with column-count and typography-scaling rules per tier.

Neither model matches how Anvil actually achieves responsive behaviour.

### What Anvil's own documentation and forum say

- Anvil staff (`stucork`), answering this exact question on the forum: responsive behaviour is controlled by a per-container property called **`wrap_on`**, not CSS breakpoints.
- Confirmed in Anvil's official Changelog: *"DataGrids are now responsive: Make your tables wrap easily on mobile devices with the new wrap_on property."*
- `wrap_on` is available on ColumnPanel, FlowPanel, and DataGrid. M3 does not have its own container components; it uses these same classic ones, so `wrap_on` applies identically inside M3 apps.
- Separately and additionally confirmed: NavigationRailLayout and NavigationDrawerLayout auto-collapse to a modal drawer or app bar on small screens. This is genuinely native and automatic, and is a different mechanism from `wrap_on`.

## Decision

Responsive behaviour is achieved via `wrap_on`, set per-container:

- `mobile` where the wireframe shows wrapping or stacking behaviour.
- `never` where the wireframe shows a fixed layout that should not rearrange.

No breakpoint-pixel table is used. Navigation collapse remains separately automatic and requires no instruction.

## Consequences

- `design-direction.md`'s breakpoint tables (both of them) are superseded and require correcting.
- The UI standards document's responsiveness section must be rewritten around `wrap_on`, not breakpoints.
- Developers set a single property per container rather than managing CSS media queries or pixel thresholds.

## Alternatives Considered

- **CSS breakpoint tables (binary or three-tier).** Rejected. Anvil does not expose CSS media query control to the developer in its component model. Breakpoint tables described in `design-direction.md` have no corresponding Anvil mechanism and cannot be implemented as specified.
- **No responsive instruction (rely entirely on native auto-collapse).** Rejected. NavigationRailLayout and NavigationDrawerLayout auto-collapse natively, but content containers (ColumnPanel, FlowPanel, DataGrid) do not rearrange without explicit `wrap_on` instruction. Both mechanisms are needed.

## Related Documents

| Document | Relationship |
|---|---|
| `docs/design-direction.md` | Source of the contradicting breakpoint models this ADR supersedes |
| ``adr/adr-global/design-rules.md`` | Wireframe design rules; wireframes must annotate `wrap_on` where applicable |
| ``adr/adr-global/ui-customization-approach.md`` | Broader UI customization approach; this ADR is a specific responsive instance |

---

*End of `responsive-behaviour-mechanism` ADR*
