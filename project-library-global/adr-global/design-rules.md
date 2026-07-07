# `design-rules` ADR — Wireframe Design Rules

- **Status:** Accepted
- **Date:** 2026-06-10
- **Authority:** This ADR is the single governing document for the creation and review of all wireframe HTML documents in this project. It is binding on every agent (OpenCode, GStack skills) that creates, edits, or reviews a wireframe. A wireframe that violates any rule in this document MUST be rejected and corrected before it is accepted as a deliverable.

---

## Section 0 — Mandatory Pre-Wireframe Reading

**[NEW — added to address recurring compliance failures. Not part of the original 16 rules; flagged separately so nothing original is altered.]**

Before producing or modifying any wireframe, the agent MUST read the following, in this order, and MUST NOT proceed until this is done:

1. This ADR (`design-rules` ADR) in full.
2. `docs/nomenclature.md` — naming conventions for every component type.
3. `docs/m3_component_mapping.md` — which Material 3 components are supported, partially supported, or unsupported.
4. `docs/scaffold-spec.md` — the architectural scaffold the wireframe must conform to.
5. The relevant existing code under `mb-3-cs/client_code/` for the form being wireframed, if it exists — to confirm component names, structure, and naming already in use.
6. Any architecture and design documents in `project-library/docs/`, `project-library/adr/`, and `project-library/implementation/` relevant to the specific form, flow, or screen being wireframed.
7. Any relevant prior outputs in `project-library/gstack-outputs/` (e.g. design review findings, eng review findings) that bear on the form being wireframed.

**A wireframe produced without first completing this reading is non-compliant by definition, regardless of how it looks.**

---

## Section 1 — Document Type and Purpose

1.1. A wireframe is an **instructive document**, not an illustrative one. Its sole purpose is to indicate which components must be placed on the UI and how they must be configured.

1.2. A wireframe MUST NOT be drawn as a "Screen" type document. It does not depict a finished visual design — it depicts construction instructions.

1.3. **Hierarchy of authority:** Architecture determines wireframes. Wireframes determine screens. Screens do NOT determine wireframes. If a screen and a wireframe disagree, the wireframe is correct and the screen must be fixed — never the reverse.

---

## Section 2 — Mandatory Three-Section Document Structure

Every wireframe HTML document MUST contain exactly three sections, in this order:

### 2.1 — Section One: Introduction

This section MUST contain:
- A message to the developer who will use the wireframe document.
- A page title.
- Important notes to the developer, including (at minimum) the following conventions:
  - Solid border = visible on load. Dashed border = hidden on load (`visible = False`).
  - Each component shown in Section Two displays: `component_name (Type) "fixed text if set at design time"`.

### 2.2 — Section Two: The Wireframe Diagram

This section presents only components that are explicitly intended to be created as a Material 3 component or approved custom component. Nothing else may appear here.

Each component shown MUST include: component name, component type, and any intentional display text — and nothing more.

**Required annotation examples (follow this exact format):**

```
ColumnPanel [Container component]: col_analytics (ColumnPanel)
FlowPanel [Container component]: flow_charts_primary (FlowPanel)
Card [Container component]: card_customer_growth (Card)
Plot component: plot_revenue_trend (Plot)
  → A Heading component is required alongside any Plot component
    and must be drawn in the wireframe. Do not include example
    text — the annotation "(Plot Name)" is sufficient.
Heading component: hdg_welcome (Heading) "Welcome Back"
  → Do not include example text beyond what is shown. The
    annotation "Welcome Back" is sufficient for the developer
    to know what text to include.
Link component: nav_sign_up (Link) "Sign up"
  → Do not include example text. The annotation "Sign up" is
    sufficient for the developer to include the display text.
TextBox component: txt_email (TextBox) (W) placeholder="Enter your email"
  → Do not include example text. The annotation is sufficient
    for the developer to include the placeholder text.
```

### 2.3 — Section Three: The Properties Table

This section lists **only** properties that can only be set in the visual Designer — never properties that can be set or overridden programmatically.

**Required format example:**

```
Properties table

[Designer-mandatory properties: writeback (W) must be set in
Designer Data Bindings panel for all TextBox components.
Password masking (hide_text) must be set in Designer.]

Component    | Type    | Designer Property
txt_email    | TextBox | writeback = W
txt_password | TextBox | writeback = W, hide_text = True
```

---

## Section 3 — Component Selection and Usage Rules

3.1. **Approved components only.** All components used MUST be approved Anvil Material 3 Theme components, from the authoritative dependency: **Anvil Material 3 theme and components** (dependency ID `4UK6WHQ6UX7AKELK`). Refer to the official component reference — [Anvil M3 components](https://anvil.works/docs/components/material-3/components) — and to `docs/m3_component_mapping.md`.

3.2. **No legacy Label.** Do not use the legacy `Label` (`lbl_`) component in Material 3 wireframes. Use only approved Material 3 text components. For subtitles specifically, use either `Heading` or `Text` — never `Label`.

3.3. **Correct component-purpose usage.** Follow correct usage standards for every component:
   - `FlowPanel` is for arranging components in horizontal lines with wrapping.
   - `ColumnPanel` is for vertically stacked components.
   - Do not use a `FlowPanel` or `ColumnPanel` to display text, or otherwise use any component outside its intended purpose.

3.4. **Creative, intentional use of available components.** Make creative and innovative use of the M3 components available via the dependency above — do not default to the simplest option when a more appropriate approved component exists.

3.5. **Nomenclature compliance.** Adhere strictly to the naming specification in `docs/nomenclature.md` (for example: `ColumnPanel` is named with the `col_` prefix).

---

## Section 4 — Text and Content Rules

4.1. **Every visible text element must be its own component.** Every user-visible text element shown or implied in a wireframe MUST be represented by its own explicit component. No visible heading, caption, helper text, or section title may exist only as implied text.

4.2. **No sample data.** Wireframes MUST NOT include sample data. Sample data is reserved for screens and is not permitted in wireframes.

4.3. **No behaviour, data, or explanatory text in the wireframe.** All behaviour, data, layout logic, and explanatory or descriptive text belongs exclusively in the specification documents. None of this may appear anywhere in the wireframe — including in notes, captions, or annotations.

---

## Section 5 — Container and Visual Drawing Rules

5.1. **Containers must be drawn as containers.** All containers must be drawn as a container, displaying the components they hold. The container's name is inset in the container's border.

5.2. **No border on non-bordered components.** Do not draw a border around components that do not natively have a border — such as `Heading` components.

---

## Section 6 — Properties Table Rules

6.1. **Designer-only properties only.** The properties table (Section 2.3 above) must list only properties that can *only* be set in the visual Designer. Any property that can be set or overridden programmatically MUST NOT appear in the properties table.

6.2. **Writeback requirement.** `writeback (W)` MUST be set in Designer for `DatePicker` and any other component requiring text input.

---

## Section 7 — Pre-Submission Compliance Checklist

**[NEW — added as a binding self-check. The agent MUST verify every item below before presenting a wireframe as complete.]**

Before submitting any wireframe, confirm ALL of the following are true:

- [ ] Section 0 reading completed (`design-rules` ADR, nomenclature.md, m3_component_mapping.md, scaffold-spec.md, relevant existing client_code, relevant project-library docs/adr/implementation files, relevant gstack-outputs)
- [ ] Document contains exactly three sections, in the correct order (Section 2.1–2.3)
- [ ] Document is NOT formatted as a "Screen" type document
- [ ] No legacy `Label` (`lbl_`) component is used anywhere
- [ ] Every component is an approved M3 component or approved custom component (cross-checked against `m3_component_mapping.md`)
- [ ] Every visible text element has its own explicit component — none implied
- [ ] No sample data appears anywhere in the wireframe
- [ ] No behaviour, data, layout logic, or explanatory text appears in the wireframe diagram or its annotations
- [ ] Containers are drawn as containers with names inset in the border
- [ ] No border drawn around non-bordered component types (e.g. `Heading`)
- [ ] Properties table contains ONLY Designer-only properties — no programmatically-settable properties
- [ ] `writeback (W)` is specified for all `DatePicker` and text-input components in the properties table
- [ ] All component names follow `docs/nomenclature.md` exactly
- [ ] FlowPanel/ColumnPanel and all other containers are used only for their intended structural purpose
- [ ] The wireframe architecturally matches what is defined upstream (it was not invented independently of architecture/spec documents)

**If any box cannot be checked, the wireframe is not complete. Do not present it as a deliverable.**

---

## Consequences

- Wireframes remain strictly limited to UI construction information.
- Specifications remain the authoritative location for behaviour, data rules, and explanatory detail.
- Reviews can reject non-compliant wireframes against a single accepted project decision.
- The mandatory pre-reading requirement (Section 0) and compliance checklist (Section 7) directly target the recurring failure modes of ADR non-compliance, architectural unfamiliarity, and unread specification/scaffold documents.
- Future changes to these rules must be made by updating or superseding this ADR.