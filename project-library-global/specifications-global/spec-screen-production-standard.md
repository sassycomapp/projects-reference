# Mybizz CS — Screen Production Standard

**Date:** 2026-06-03
**Authority:** Approved example — `screens/Example-screen-auth-SignupForm.html`
**Purpose:** Canonical rules for all screen artefacts in the Mybizz CS project

# Note to reader
Hard coated colors must not be present in any document Color palettes are by anvil color tokens system 

---

## Core Principle

A screen is a **visual UI mockup**, not a documentation report. It must look like the intended Anvil form/page when rendered in a browser. A developer or designer opening the screen file should see the form layout, not a component inventory table.

---

## Mandatory Style Rules

### 1. Visual Identity

- **Font:** Roboto via Google Fonts (`https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap`)
- **Design tokens:** M3 CSS custom properties (see token block below)
- **Background:** `var(--surface)` — clean white/off-white
- **No grey documentation backgrounds** — the screen IS the UI

### 2. M3 Token Block (required in every screen)

```css
:root {
  --primary: #2e519e;
  --on-primary: #FFFFFF;
  --primary-container: #d7e0f4;
  --on-primary-container: #101623;
  --secondary: #5d636f;
  --on-secondary: #FFFFFF;
  --secondary-container: #e3e5e8;
  --on-secondary-container: #18191b;
  --surface: #fcfcfd;
  --on-surface: #18191b;
  --surface-variant: #e5e5e6;
  --on-surface-variant: #494b50;
  --surface-container-low: #f4f5f5;
  --outline: #797d86;
  --outline-variant: #cacbce;
  --error: #9e5d2e;
  --on-error: #FFFFFF;
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --font-family: 'Roboto', sans-serif;
  --font-headline-large: 32px/40px 600;
  --font-headline-small: 22px/28px 600;
  --font-title: 18px/24px 500;
  --font-body: 14px/20px 400;
  --font-button: 14px/14px 500;
  --font-label: 12px/16px 500;
  --font-caption: 12px/16px 400;
  --radius-button: 100px;
  --radius-card: 8px;
  --radius-input: 4px;
  --mui-shadow-card: hsla(220,30%,5%,0.07) 0px 4px 16px 0px, hsla(220,25%,10%,0.07) 0px 8px 16px -5px;
  --divider-color: var(--outline-variant);
  /* Status colours — semantic exceptions */
  --status-success: #00b958;
  --status-warning: #ff9469;
  --status-error: #ff000c;
}
```

### 3. Layout Patterns by Form Type

| Form type | Max Width | Layout | Container |
|---|---|---|---|
| Auth forms | 400px | Centred card | `.auth-container > .auth-card` |
| List forms | 1200px | Full-width | `.list-container` with header + filters + grid |
| Dashboard | 1200px | Full-width | `.dashboard-container` with metric cards |
| Editor/Viewer/Forms | 900px | Full-width | `.editor-container` with sectioned panels |
| Public pages | 900px | Full-width | `.page-container` with content sections |
| Settings | 1000px | Full-width | `.settings-container` with tab bar + panels |
| Onboarding | 600px | Centred card | `.onboarding-container` with step indicator |
| Card (grid item) | 320px | Embedded | `.card-container` |
| Modal form | 480px | Overlay | `.modal-container` |
| Modal detail | 640px | Overlay | `.modal-container` |

### 4. Panel Separation

- Use subtle bordered panels (`border: 1px solid var(--surface-variant); border-radius: var(--radius-card)`) to separate functional areas
- Each panel gets `padding: var(--space-6)` and `margin-bottom: var(--space-4)`
- Panel titles use `font: var(--font-title)` or `font: var(--font-headline-small)`

### 5. Form Components

- **Inputs:** Full-width, `border: 1px solid var(--outline)`, `border-radius: var(--radius-input)`, `min-height: 48px`, `padding: var(--space-3) var(--space-4)`
- **Labels:** `font: var(--font-label)`, `color: var(--on-surface-variant)`, above input with `margin-bottom: var(--space-2)`
- **Primary buttons:** `background: var(--primary)`, `color: var(--on-primary)`, `border-radius: var(--radius-button)`, `min-height: 40px`
- **Secondary buttons:** `background: transparent`, `border: 1px solid var(--primary)`, `color: var(--primary)`
- **Tertiary buttons:** Text-only, `color: var(--primary)`, underline on hover

### 6. Data Display

- **Headings:** Use proper hierarchy — `h1` for page title, `h2` for section, `h3` for card/panel title
- **Key-value pairs:** Label in `var(--on-surface-variant)`, value in `var(--on-surface)` with `font-weight: 500`
- **Tags/badges:** `background: var(--secondary-container)`, `color: var(--on-secondary-container)`, `border-radius: 8px`, `padding: 2px 10px`
- **DataGrid mockup:** Styled table with `var(--surface-variant)` header, alternating row backgrounds

### 7. Accessibility

- Semantic HTML: `<main>`, `<article>`, `<header>`, `<section>`, `<nav>`
- `role` attributes where appropriate
- Focus states: `outline: 2px solid var(--primary); outline-offset: 2px`
- `@media (prefers-reduced-motion: reduce)` — disable transitions
- `@media (prefers-color-scheme: dark)` — dark mode token overrides

### 8. Dark Mode (required)

```css
@media (prefers-color-scheme: dark) {
  :root {
    --surface: #1C1B1F;
    --on-surface: #E6E1E5;
    --surface-variant: #49454F;
    --on-surface-variant: #CAC4D0;
    --outline: #938F99;
  }
}
```

---

## What Must Never Appear Again

| Old pattern | Reason |
|---|---|
| `Implementation Screen:` in title | Screen is a mockup, not a report |
| `.meta` block with Source wireframe / Phase / Status / Created | Documentation metadata does not belong in a visual mockup |
| `Screen Purpose` section | Descriptive text — not visual content |
| `Sections (from wireframe)` bullet list | Documentation — not visual content |
| `Component Inventory` table with Stub status | The screen SHOWS the components; it does not list them |
| `Implementation Notes` section | Documentation — not visual content |
| `code { background: #f5f5f5 }` inline annotations | Wireframe annotations are for wireframes, not screens |
| `End of screen file —` footer note | Documentation artifact |
| Arial font family | Must be Roboto |
| Grey `#f5f5f5` body background | Must be `var(--surface)` |

---

## Screen Title Convention

- `<title>Mybizz CS — [Form Name]</title>`
- No "Implementation Screen:" prefix
- No "Anvil Construction Guide" subtitle

---

## File Naming

Unchanged: `screen-{package}-{FormName}.html`

---

*End of screen production standard — 2026-06-03*
