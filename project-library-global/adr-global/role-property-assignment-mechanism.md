# `role-property-assignment-mechanism` ADR: Role Property Assignment Mechanism (Designer vs. Code)

## Status

**Superseded** — original decision reversed by project-wide property-setting rule.

## Original Decision (superseded)

Set `role` via the **Designer Properties Panel**, not in code.

## Reversal Reason

The project-wide property-setting rule states: **"If a property can be set programmatically, then it must be set programmatically."**

Anvil's official Roles documentation confirms that `role` can be set programmatically:
```python
self.button_1.role = "submit"
```

Since `role` can be set in code, the project rule requires it to be set in code.

## Current Decision

Set `role` in **code** (`self.component.role = "role-name"`), not via the Designer Properties Panel.

## What Anvil's documentation says

Anvil's Roles documentation supports both approaches:
- Designer: "To set the role of a component, select the desired component in the Form Editor and go to the Properties Panel."
- Code: "You can set the role of a component in code: `self.button_1.role = "submit"`"

Components can have multiple roles (space-separated string or list of strings).

## Consequences

- Role assignment is done in Python code, not in the Designer Properties Panel.
- `design-direction.md`, `spec-material-3-theme.md`, and the Standard all need to reflect code-based role assignment.
- The Standard's §6.0 must be updated to show code examples, not Designer instructions.

## Related Documents

| Document | Relationship |
|---|---|
| `docs/design-direction.md` | Source of the original contradiction |
| `docs/spec-material-3-theme.md` | Contains code examples — now correct per this reversal |
| `ui-build-prep/anvil-ui-build-standard.md` | Standard §6.0 must be updated |
| Anvil Roles docs | `https://anvil.works/docs/client/customisation/using-css/roles` |

---

*End of `role-property-assignment-mechanism` ADR (superseded)*
