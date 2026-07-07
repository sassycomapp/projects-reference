# PDLF Standards Library — Screen & Wireframe Production Standards

**Location:** `C:\pdlf\standards-library\screen-and-wireframe-production-standards.md`
**Scope:** Project-agnostic production *rules* and the compliance-checklist *method*. Specific
token values (exact hex colors, exact spacing scale) are a project's own DESIGN.md/theme, not a
standard — this file states the *rule that they must exist and be used*, never the values
themselves.
**Source:** Extracted from `mb-3-cs` (screen-production-standard.md, two worked compliance
checklists), generalized.

---

## 1. Core Principle

A screen is a visual UI mockup, not a documentation report. Opening the file should show the
intended rendered layout — not a component-inventory table, a "Screen Purpose" section, or
implementation notes. This is a documented, recurring failure mode: an agent substituting its own
design judgment for the wireframe's and the standard's intention. The fix is not "be more
careful" — it's the mandatory two-stage checklist in Section 3.

## 2. Mandatory Style Rules (Pattern, Not Values)

- **Font:** a single, consistently-loaded web font across the whole app — never mixed families.
- **Colors:** always via the project's design-token system (CSS custom properties, or equivalent)
  — **never hardcoded hex or rgba, anywhere, for any reason.** This must be verified by literal
  grep, not by eye — a single hardcoded value can hide in one line of CSS and pass a visual check.
- **Background:** the token system's own surface color — never a generic grey "documentation"
  background.
- **Layout patterns by form type**: see `m3-design-standards.md` Section 16 for the authoritative
  max-width/container table — do not restate these values here or in any project's own document.
- **Panel separation**: functional areas separated by consistent bordered panels, not ad-hoc
  spacing.
- **Accessibility, always required, not optional:** semantic HTML, visible focus states,
  `prefers-reduced-motion` support, dark mode via `prefers-color-scheme`.

## 3. Mandatory Two-Stage Compliance Check

Screen production is not complete until both checklists pass. This two-stage, itemized,
evidence-based check exists specifically to catch what a general "does this look right" review
misses.

### Stage 1 — Wireframe Fidelity

- Every component the wireframe declares appears in the screen — checked by name/type, not by
  general impression.
- No extra components added.
- Component hierarchy and types match.
- No page chrome not declared by the wireframe (no sidebar/app-bar/footer unless the wireframe
  shows one).
- Sample data matches the project's stated conventions (correct domain/vertical examples, no
  Lorem ipsum, no real production data).

### Stage 2 — Standard Compliance

- Token/color compliance verified by grep, not by eye.
- Only approved components used (no components the project's standard excludes).
- Naming and structural conventions followed.
- Typography, spacing, and radius values match the standard.
- Dark mode and reduced-motion blocks present if required.

**If either checklist fails, the screen is revised and re-checked before proceeding** — document
the specific line and specific violation found, the same way a real audit does, not a general
"some issues were found."

## 4. What Must Never Appear in a Finished Screen

| Pattern | Reason |
|---|---|
| A documentation-style title prefix (e.g. "Implementation Screen:") | The screen is a mockup, not a report |
| A metadata block (source wireframe / phase / status / created date) | Documentation metadata doesn't belong in a visual mockup |
| A "Purpose" or descriptive-text section | Descriptive text is not visual content |
| A "Sections (from wireframe)" bullet list | Documentation, not visual content |
| A component-inventory table with build-status column | The screen *shows* the components; it doesn't list them |
| An "Implementation Notes" section | Documentation, not visual content |
| Inline wireframe-style annotations (e.g. grey code blocks describing intent) | Wireframe annotations are for wireframes, not finished screens |
| A footer note like "End of screen file —" | Documentation artifact |
| A generic system font where the project specifies a particular one | Brand/typography consistency |
| A generic grey documentation background | The screen should use the real surface token |

## 5. The Screen Production Chain

```
Approved wireframe → Screen HTML (this standard applies) → Screen PNG (external render) → Anvil AI UI build
```

The screen HTML is produced by the orchestrator directly, following the *project's own* specific
standard document as the literal spec — a generic "make it look nice" pass will reproduce exactly
the failure mode Section 1 describes.

---

*Screen & Wireframe Production Standards v1.0. The rules and the checklist method are the
reusable part — a project's exact token values and specific banned-pattern history stay in that
project's own screen-production-standard.md.*
