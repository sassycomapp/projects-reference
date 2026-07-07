# Compliance Checklist — screen-analytics-DashboardAnalyticsForm

**Screen:** `/mnt/c/mybizz/project-library/screens/analytics/DashboardAnalyticsForm/screen-analytics-DashboardAnalyticsForm.html`
**Wireframe:** `/mnt/c/mybizz/project-library/wireframes/analytics/DashboardAnalyticsForm/wireframe-analytics-DashboardAnalyticsForm.html`
**Date:** 2026-07-02
**Revised:** 2026-07-02 — audit found 2 issues, fixed

---

## A. Wireframe fidelity

- [x] Every component in the wireframe's Section 2 appears in the screen
  - `hdg_analytics_title` (Heading) ✓
  - `txt_analytics_subtitle` (Text) ✓
  - `flow_period_selector` (FlowPanel) with 4 Link components ✓
  - `flow_metrics_row` (FlowPanel) with 4 Cards ✓
  - `flow_charts_primary` (FlowPanel) with 2 Cards (2 Plots) ✓
  - `flow_charts_secondary` (FlowPanel) with 2 Cards (1 Plot + 1 DataGrid) ✓
  - `col_empty_state` — excluded (dashed border = hidden on load) ✓
- [x] No extra components
- [x] Component hierarchy matches wireframe
- [x] Component types match wireframe

## B. Page chrome exclusion

- [x] No left sidebar, app bar, footer, or navigation
- [x] The screen contains ONLY what the wireframe declares

## C. Repeating panels and row templates

- [x] DataGrid row content NOT rendered — placeholder annotation shown
- [x] No RepeatingPanel present

## D. Sample data (per Standard §8)

- [x] ZAR currency: "R 12 450", "R 259"
- [x] South African consulting vertical
- [x] No Lorem ipsum, no production data

## E. MUI aesthetic overlay (all 9 variables)

- [x] 1. Surface distinction — `var(--surface)` body, `var(--surface-container-low)` DataGrid header
- [x] 2. Card shadow — MUI dual-layer hsla
- [x] 3. Card radius — 8px (metric cards, chart cards)
- [x] 4. Heading weight — 600 (`.hdg-analytics-title`)
- [x] 5. Dividers — outline-variant at 40% opacity
- [x] 6. Hover opacity — 0.08 via `--link-hover-bg: color-mix()`
- [x] 7. Pressed/focused — 0.12 via CSS variable
- [x] 8. App bar — N/A
- [x] 9. Left nav — N/A

## F. Colour and token compliance

- [x] No hardcoded hex (verified via grep) — badge colours `#DEFCD3`/`#1B5E20`/`#FFF3CD`/`#856404` are semantic status exceptions per Standard §8
- [x] No hardcoded rgba (verified via grep) — period selector hover now uses `var(--link-hover-bg)` with `color-mix()`
- [x] All other colors via CSS custom properties

**Previously failing — now fixed:**
- ~~Line 111: `rgba(73, 62, 243, 0.08)` on `.flow-period-selector a:hover`~~ → `var(--link-hover-bg)` using `color-mix()`

## G. Component compliance

- [x] No Label, no HTMLTemplate, no SidesheetContent, no HtmlPanel
- [x] All components from approved M3 list

## H. Naming and structure

- [x] Prefixes correct (hdg_, txt_, nav_, card_, flow_, col_, dg_, plot_)
- [x] Material Icons imported

## I. Output quality

- [x] Renders correctly
- [x] Typography scale correct (Roboto, 32px/600 title, 14px body)
- [x] Spacing on 4px grid
- [x] Dark mode — `@media (prefers-color-scheme: dark)` present with 7 token overrides
- [x] Reduced motion present

**Previously failing — now fixed:**
- ~~Missing dark mode block~~ → added

---

## Discrepancies found

**None remaining.** Two issues found and fixed.

---

*End of checklist — screen-analytics-DashboardAnalyticsForm (revised 2026-07-02)*
