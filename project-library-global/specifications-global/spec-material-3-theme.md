Based on the information gathered, here is a strict spec/documentation style rewrite for your project docs:

***

## AM1-XX: Material 3 Theme Implementation Specification

### 1. Overview

This application uses **Anvil's official Material 3 Theme dependency** as the foundational design system. Material 3 (M3) components, layouts, and design tokens are provided by this dependency and customized through roles and CSS overrides. [anvil](https://anvil.works/docs/ui/app-themes/material-3)

**MUI Aesthetic Overlay:** The MUI dashboard aesthetic has been applied on top of M3 via CSS custom properties and Anvil role selectors. See `DESIGN.md` Section "MUI Aesthetic Implementation" for the authoritative MUI variables. See `theme_mui_patch.css` for the reference patch and `theme_patched.css` for the deployment-ready theme.

### 2. Theme Architecture

#### 2.1 Material 3 Dependency (MANDATORY)

- **Dependency ID:** `4UK6WHQ6UX7AKELK`
- **Package Name:** `m3`
- **Version:** v1.2.6
- **Installation Location:** Settings → Dependencies → Third Party
- **Status:** Required for all forms and components

The Material 3 dependency provides:
- M3-compliant component library
- NavigationRailLayout and related layout containers
- Material 3 design tokens (color, typography, elevation)
- Component variants (button appearances, card types, input styles)

#### 2.2 Theme Customization Hierarchy

Theme configuration follows this precedence order:

1. **Material 3 Dependency** (base layer, non-negotiable)
2. **Assets/theme.css** (app-level design tokens and overrides)
3. **Anvil Roles** (reusable component styling patterns)
4. **Component-level CSS** (instance-specific overrides, use sparingly)

### 3. Implementation Rules

#### 3.1 Design Tokens (Assets/theme.css)

All app-specific color, spacing, and typography values MUST be defined as CSS custom properties in `Assets/theme.css`:

```css
/* App-level Material 3 design tokens */
:root {
  /* Primary color palette */
  --app-color-primary: #6750A4;
  --app-color-on-primary: #FFFFFF;
  --app-color-primary-container: #EADDFF;
  
  /* Surface colors */
  --app-color-surface: #FFFBFE;
  --app-color-on-surface: #1C1B1F;
  --app-color-surface-variant: #E7E0EC;
  
  /* Spacing scale (4px base unit) */
  --app-space-1: 4px;
  --app-space-2: 8px;
  --app-space-4: 16px;
  --app-space-6: 24px;
  --app-space-8: 32px;
  
  /* Typography */
  --app-font-body: 'Roboto', sans-serif;
  --app-font-display: 'Roboto', sans-serif;
}
```

**Rule:** Never hardcode hex colors, pixel values, or font names in component styles. Always reference tokens.

#### 3.2 Anvil Roles (Reusable Component Patterns)

Roles provide reusable styling hooks mapped to `.anvil-role-*` CSS classes. Create roles in `theme.css` for all recurring UI patterns:

```css
/* Primary button pattern */
.anvil-role-primary-button {
  background-color: var(--app-color-primary);
  color: var(--app-color-on-primary);
  border-radius: 20px;
  padding: 12px 24px;
}

/* Surface card pattern */
.anvil-role-surface-card {
  background-color: var(--app-color-surface);
  border: 1px solid var(--app-color-surface-variant);
  border-radius: 12px;
  padding: var(--app-space-4);
}

/* Section header pattern */
.anvil-role-section-header {
  font-family: var(--app-font-display);
  font-size: 24px;
  font-weight: 500;
  color: var(--app-color-on-surface);
  margin-bottom: var(--app-space-4);
}
```

**Application (per `role-property-assignment-mechanism` ADR, reversed):**

Roles are set programmatically in code, per the project-wide property-setting rule: if a property can be set programmatically, it must be set programmatically. Anvil's Roles documentation confirms `role` can be set in code.

```python
# Role assignment in code
self.save_button.role = "primary-button"
self.info_card.role = "surface-card"
self.page_title.role = "section-header"
```

Components can have multiple roles (space-separated string or list of strings):

```python
self.card.role = "surface-card elevated"
```

#### 3.3 Material Web Components (EXCLUDED)

**Material Web Components are excluded from the Mybizz CS project.** See `14-anvil-extras-exclusion.md` for the complete component scope policy.

Only components included in the Anvil Material 3 Theme dependency (`4UK6WHQ6UX7AKELK`) are permitted. Any component outside that dependency requires explicit ADR approval.

### 4. What You CANNOT Do

Material 3 in Anvil is **NOT** installed via:

- ❌ Python package management: `pip install material3`
- ❌ npm package management: `npm install @material/web`
- ❌ Gradle dependencies (Android/Flutter pattern)
- ❌ CDN-only approach without Anvil dependency

**Correct approach:** Install Anvil's Material 3 dependency (`4UK6WHQ6UX7AKELK`), then customize via `theme.css` and roles.

### 5. Component-Level Guidelines

#### 5.1 Buttons

- Use `appearance` property (not `role` property for appearance variants)
- Valid appearances: `filled`, `outlined`, `text`
- Assign roles for semantic patterns: `primary-button`, `secondary-button`, `danger-button`

```python
# Material 3 button configuration
self.save_button.appearance = "filled"
# role is set via Designer Properties Panel per `role-property-assignment-mechanism` ADR
```

#### 5.2 TextBox and DropdownMenu

- Use `appearance` property for Material 3 input styling
- Valid appearances: `outlined`, `filled`
- Use `error` property for validation states (not `appearance` variants)

```python
# Material 3 input configuration
self.email_input.appearance = "outlined"
self.email_input.error = "Email is required"
```

#### 5.3 Cards

- Use Material 3 Card component
- Variants: `elevated`, `filled`, `outlined` (via properties, not roles)
- Apply roles for content-specific patterns

### 6. Best Practices

#### 6.1 Centralized Styling

- Define all design tokens in `Assets/theme.css`
- Create roles for recurring UI patterns (buttons, cards, headers, forms)
- Avoid inline styles and component-level CSS unless absolutely necessary

#### 6.2 Consistency

- Use the same role for the same UI pattern across all forms
- If a primary button uses `primary-button` role, ALL primary buttons use that role
- Document custom roles in this specification

#### 6.3 Maintenance

- When updating colors or spacing, change tokens in `theme.css` only
- Role definitions cascade to all components using that role
- Test theme changes across all forms before committing

### 7. Reference

- **Anvil Material 3 Documentation:** [https://anvil.works/docs/ui/app-themes/material-3](https://anvil.works/docs/ui/app-themes/material-3)
- **Anvil Roles Documentation:** [https://anvil.works/docs/ui/custom-styling/roles](https://anvil.works/docs/ui/custom-styling/roles)
- **Material 3 Design System:** [https://m3.material.io/](https://m3.material.io/)

***

**End of Specification**