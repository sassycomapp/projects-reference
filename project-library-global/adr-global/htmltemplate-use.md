# `htmltemplate-use` ADR: HTMLTemplate Use

## Status

Accepted

## Context

`spec-component-properties.md` documents `HTMLTemplate` with a Designer-only `anvil_slot_repeat` property, implying sanctioned use. The wireframe research report found it in three actual wireframes:

1. `ServicesForm` — wrapping the TimeLapseCarouselComponent, for `position: relative, overflow: hidden`.
2. `AdminLayout` — building an entire Layout shell from scratch.
3. `CustomerLayout` — building an entire Layout shell from scratch.

### Assessment

In all three cases, a native alternative exists:

- The ServicesForm case needs only basic CSS positioning, achievable via a plain ColumnPanel with a role-driven CSS class (per `role-property-assignment-mechanism` ADR, roles are a real, working mechanism). No HTML escape hatch is needed.
- M3 already ships two native, complete Layouts: `NavigationRailLayout` and `NavigationDrawerLayout`. Building a custom HTML-based shell duplicates what these already provide.

## Decision

`HTMLTemplate` is banned. Use stock Anvil M3 components and the two native M3 Layouts in all cases.

## Consequences

- The three wireframes using `HTMLTemplate` require reworking to use native M3 components instead.
- The approved-component list in the UI standards document explicitly excludes `HTMLTemplate`, with this ADR cited as the reason.
- Any future proposal to use `HTMLTemplate` must first demonstrate that no native M3 component or layout can achieve the same result, and must be approved via a new ADR.

## Alternatives Considered

- **Allow `HTMLTemplate` for specific edge cases.** Rejected. Every current use case has a native alternative. Allowing exceptions creates a precedent that erodes the "native M3 only" principle and introduces an HTML escape hatch that bypasses Anvil's component model.
- **Allow `HTMLTemplate` with ADR approval per use.** Rejected as a middle ground. The current use cases do not justify the governance overhead of a per-instance approval process. If a genuinely unachievable case emerges in the future, a new ADR can revisit this ban.

## Related Documents

| Document | Relationship |
|---|---|
| `docs/spec-component-properties.md` | Documents `HTMLTemplate` and its properties; superseded for this component |
| ``adr/adr-global/anvil-extras-exclusion.md`` | Excludes third-party packages; this ADR extends the exclusion principle to `HTMLTemplate` |
| ``adr/adr-global/material-3-theme-component-scope.md`` | Defines permitted M3 components; `HTMLTemplate` is not among them |
| ``adr/adr-global/design-rules.md`` | Wireframe design rules; wireframes must not use `HTMLTemplate` |
| ``adr/adr-global/ui-customization-approach.md`` | Broader UI customization approach; this ADR is a specific component exclusion |
| ``adr/adr-global/role-property-assignment-mechanism.md`` | Role-driven CSS classes provide the native alternative for the ServicesForm case |

---

*End of `htmltemplate-use` ADR*
