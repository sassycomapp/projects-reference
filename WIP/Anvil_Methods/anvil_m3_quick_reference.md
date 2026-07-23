# Anvil Material 3 Compliance Code of Practice

**Version**: 1.0  
**Date**: February 2, 2026  
**Purpose**: Enforce full M3 compliance in MyBizz app development  
**Authority**: [Anvil M3 Documentation](https://anvil.works/docs/ui/app-themes/material-3)

---

## 1. M3 Dependency & Setup

### 1.1 Required Configuration
- **Dependency ID**: `4UK6WHQ6UX7AKELK` (Settings → Dependencies → Third Party)
- **Import statement**: Auto-added by theme: `import m3.components as m3`
- **Theme compatibility**: Do NOT mix with legacy Anvil themes (CSS conflicts)

### 1.2 Version Control
- Track M3 theme version in project documentation
- Monitor [M3 changelog](https://github.com/anvil-works/material-3-theme/blob/master/CHANGELOG.md) for breaking changes
- Test after theme version updates

---

## 2. Layouts: NavigationDrawerLayout & NavigationRailLayout

### 2.1 Layout Selection Rules
**NavigationDrawerLayout**: Use when:
- 7+ navigation destinations
- Navigation hierarchy/grouping needed
- Longer destination labels required

**NavigationRailLayout**: Use when:
- 3-7 navigation destinations (per M3 guidelines)
- Compact layout preferred
- Icons can represent destinations clearly

### 2.2 Layout Implementation Standards
```python
# ✅ CORRECT: AdminLayout inherits from M3 layout
class AdminLayout(NavigationDrawerLayoutTemplate):
    def __init__(self, **properties):
        self.init_components(**properties)
        self.build_navigation()
    
    def build_navigation(self):
        # Set navigate_to on NavigationLinks
        self.nav_dashboard.navigate_to = 'DashboardForm'
        self.nav_bookings.navigate_to = 'BookingListForm'
        
        # Feature-based visibility
        self.nav_bookings.visible = self.config['features']['bookings']

# ❌ INCORRECT: Custom sidebar with manual navigation
class AdminLayout(AdminLayoutTemplate):  # Wrong base class
    def nav_link_click(self, **event_args):
        open_form('DashboardForm')  # Manual navigation
```

### 2.3 NavigationLink Properties
- **navigate_to**: Set to Form name (string) or Form instance
- **text**: Destination label
- **icon**: Material icon name
- **visible**: Control with feature flags
- **Do NOT**: Use click handlers when `navigate_to` is set

### 2.4 Responsive Behavior
- NavigationDrawer collapses to modal on mobile (automatic)
- NavigationRail can become bottom bar or modal (configurable via layout properties)
- Test all breakpoints during development

---

## 3. Component Standards

### 3.1 Typography Components

**Text Component** (body text, labels):
```python
# ✅ USE Text for content and labels
self.lbl_description = m3.Text(
    text="Enter customer details",
    style="label",    # 'body' or 'label'
    scale="medium"    # 'small', 'medium', 'large'
)
```

**Heading Component** (titles, headers):
```python
# ✅ USE Heading for page/section titles
self.lbl_page_title = m3.Heading(
    text="Customer Management",
    style="headline",  # 'display', 'headline', 'title'
    scale="large"      # 'small', 'medium', 'large'
)
```

**Typography Role Reference**:
| Use Case | Component | Style | Scale |
|----------|-----------|-------|-------|
| Page title | Heading | headline | large |
| Section header | Heading | headline | small |
| Card title | Heading | title | medium |
| Body text | Text | body | medium |
| Field label | Text | label | medium |
| Caption | Text | body | small |

**❌ NEVER**: Use legacy `Label` component in M3 apps

### 3.2 Form Input Components

**TextBox & TextArea**:
```python
# ✅ ALWAYS use 'outlined' role for form fields
self.txt_name = m3.TextBox(
    role='outlined',
    label='Customer Name',
    placeholder='Enter name'
)

# Error state
if not valid:
    self.txt_email.role = 'outlined-error'
    self.txt_email.placeholder = "Email required"
else:
    self.txt_email.role = 'outlined'
```

**DropdownMenu**:
```python
# ✅ Use outlined role for consistency
self.dd_status = m3.DropdownMenu(
    role='outlined',
    label='Status',
    items=[('active', 'Active'), ('inactive', 'Inactive')]
)
```

**Other Form Components**:
- **Checkbox**: Use for boolean options
- **RadioButton**: Use in RadioGroupPanel for single choice
- **Switch**: Use for feature toggles, settings
- **DatePicker**: Use `outlined` role
- **Slider**: Set min, max, step, value properties
- **FileLoader**: Default styling appropriate

### 3.3 Button Hierarchy

**Visual Hierarchy** (importance descending):
1. **Filled Button**: Primary action (1 per screen recommended)
2. **Outlined Button**: Secondary actions
3. **Text Button**: Tertiary/low-emphasis actions

```python
# ✅ CORRECT button hierarchy in form
self.btn_save = m3.Button(
    text="Save",
    role='filled-button'  # Primary action
)

self.btn_cancel = m3.Button(
    text="Cancel",
    role='outlined'  # Secondary action
)

self.btn_details = m3.Button(
    text="View Details",
    role='text-button'  # Tertiary action
)
```

**IconButton & ToggleIconButton**:
```python
# ✅ Use for icon-only actions
self.btn_edit = m3.IconButton(
    icon='edit',
    tooltip='Edit item'
)

self.btn_favorite = m3.ToggleIconButton(
    icon='favorite_border',
    icon_activated='favorite'
)
```

**ButtonMenu & IconButtonMenu**:
```python
# ✅ Use for action menus
self.menu_actions = m3.IconButtonMenu(
    icon='more_vert'
)
# Add MenuItems in designer or via menu_items property
```

### 3.4 Display Components

**Card**:
```python
# ✅ Use cards for content grouping
self.card_details = m3.Card(
    role='outlined',  # or 'elevated'
    spacing_above='small',
    spacing_below='small'
)
```

**InteractiveCard**:
- Use when entire card should be clickable
- Has click event handler

**Divider**:
- Use for visual separation within containers
- Horizontal or vertical orientation

### 3.5 Feedback Components

**Progress Indicators**:
```python
# LinearProgressIndicator for horizontal progress
self.progress = m3.LinearProgressIndicator(
    indeterminate=False,
    progress=0.5  # 0.0 to 1.0
)

# CircularProgressIndicator for loading states
self.spinner = m3.CircularProgressIndicator(
    indeterminate=True
)
```

---

## 4. Color System

### 4.1 Theme Colors (Required)
**ALWAYS use theme colors** for consistency and theme switching support:

```python
# ✅ CORRECT: Use theme color references
self.card.background_color = 'theme:Surface'
self.card.foreground_color = 'theme:On Surface'
self.btn_primary.background_color = 'theme:Primary'
self.btn_primary.foreground_color = 'theme:On Primary'
```

**Available Theme Colors**:
- `theme:Primary` / `theme:On Primary`
- `theme:Secondary` / `theme:On Secondary`
- `theme:Tertiary` / `theme:On Tertiary`
- `theme:Surface` / `theme:On Surface`
- `theme:Surface Variant` / `theme:On Surface Variant`
- `theme:Error` / `theme:On Error`
- `theme:Background` / `theme:On Background`

**❌ NEVER**: Hardcode color values (`#FFFFFF`, `rgb(255,255,255)`)

### 4.2 Custom Color Schemes
- Use [Google Material Theme Builder](https://material-foundation.github.io/material-theme-builder/) for custom schemes
- Export color scheme and apply in Anvil Color Scheme editor
- Test both light and dark mode variants
- Reference: [Anvil M3 Color Scheme Guide](https://anvil.works/docs/how-to/creating-material-3-colour-scheme)

---

## 5. Container Components

### 5.1 M3-Compatible Containers
**ColumnPanel**: Vertical stacking (primary layout tool)
**LinearPanel**: Horizontal arrangement
**FlowPanel**: Responsive wrapping layout
**GridPanel**: Not M3-specific but compatible
**Card**: Content grouping (M3-specific)

### 5.2 Layout Patterns

**List Form Layout**:
```python
# Structure: ColumnPanel > Header > Filters > DataGrid
col_content (ColumnPanel)
├─ lp_header (LinearPanel)
│  ├─ lbl_title (Heading - headline/large)
│  ├─ spacer
│  └─ btn_new (Button - filled)
├─ lp_filters (LinearPanel)
│  ├─ txt_search (TextBox - outlined)
│  └─ dd_status (DropdownMenu - outlined)
└─ dg_data (DataGrid)
```

**Editor Form Layout**:
```python
# Structure: ColumnPanel > Card(s) > Actions
col_form (ColumnPanel)
├─ card_main (Card - outlined)
│  ├─ lbl_section (Heading - headline/small)
│  ├─ txt_field1 (TextBox - outlined)
│  ├─ txt_field2 (TextBox - outlined)
│  └─ txt_notes (TextArea - outlined)
└─ lp_actions (LinearPanel)
   ├─ btn_save (Button - filled)
   └─ btn_cancel (Button - outlined)
```

### 5.3 Spacing & Alignment
- Use M3 spacing properties: `spacing_above`, `spacing_below`
- Values: `'none'`, `'small'`, `'medium'`, `'large'`
- Set on individual components for consistency

---

## 6. Prohibited Components & Patterns

### 6.1 DO NOT Use Anvil Extras Components
**Prohibited** (M3 alternatives exist):
- ~~Tabs~~ → Use NavigationLink in sidebar
- ~~Pivot~~ → Use custom DataGrid views
- ~~MultiSelectDropDown~~ → Use DropdownMenu + Chips
- ~~Autocomplete~~ → Use DropdownMenu with search
- ~~Quill~~ → Use TextArea
- ~~Switch (Extras)~~ → Use M3 Switch
- ~~Slider (Extras)~~ → Use M3 Slider
- ~~RadioGroup (Extras)~~ → Use M3 RadioGroupPanel
- ~~CheckBoxGroup (Extras)~~ → Use multiple M3 Checkboxes

### 6.2 DO NOT Use Legacy Patterns
**Prohibited**:
- ~~Custom sidebars with Link components~~ → Use NavigationDrawerLayout
- ~~Manual `open_form()` in navigation click handlers~~ → Set `navigate_to` property
- ~~XYPanel for layout~~ → Use ColumnPanel/LinearPanel
- ~~Hardcoded colors~~ → Use `theme:` colors
- ~~Generic Label components~~ → Use Text/Heading with proper roles

---

## 7. Naming Conventions (MyBizz Standard)

### 7.1 Component Prefixes
**MUST follow** nomenclature standard (`05_nomenclature.md`):

| Component Type | Prefix | Example |
|----------------|--------|---------|
| NavigationLink | `nav_` | `nav_dashboard` |
| Button | `btn_` | `btn_save` |
| IconButton | `btn_` | `btn_edit` |
| Text | `lbl_` | `lbl_description` |
| Heading | `lbl_` | `lbl_page_title` |
| TextBox | `txt_` | `txt_customer_name` |
| TextArea | `txt_` | `txt_notes` |
| DropdownMenu | `dd_` | `dd_status` |
| Checkbox | `cb_` | `cb_active` |
| RadioButton | `rb_` | `rb_option1` |
| Switch | `sw_` | `sw_enabled` |
| DatePicker | `dp_` | `dp_booking_date` |
| FileLoader | `fu_` | `fu_upload` |
| Card | `card_` | `card_details` |
| ColumnPanel | `col_` | `col_content` |
| LinearPanel | `lp_` | `lp_header` |
| FlowPanel | `flow_` | `flow_items` |
| DataGrid | `dg_` | `dg_customers` |
| RepeatingPanel | `rp_` | `rp_items` |
| ButtonMenu | `menu_` | `menu_actions` |
| Plot | `plot_` | `plot_revenue` |

---

## 8. Testing & Validation

### 8.1 M3 Compliance Checklist
Before deploying any form:
- [ ] Layout inherits from NavigationDrawerLayout or NavigationRailLayout
- [ ] All NavigationLinks use `navigate_to` property (no manual click handlers)
- [ ] All text uses Text or Heading components (no Label)
- [ ] All form inputs use `outlined` role
- [ ] Button hierarchy correctly applied (filled > outlined > text)
- [ ] All colors use `theme:` references
- [ ] No Anvil Extras components where M3 alternatives exist
- [ ] Component naming follows MyBizz conventions
- [ ] Responsive behavior tested (desktop, tablet, mobile)
- [ ] Error states use M3 error roles (`outlined-error`)

### 8.2 Local Testing Protocol
1. Test via Anvil Uplink (local runtime)
2. Verify all M3 components render correctly
3. Test navigation flow (all NavigationLinks functional)
4. Test responsive breakpoints
5. Verify theme color consistency
6. Test error state handling

### 8.3 Deployment Testing
1. Push to GitHub
2. Test in Anvil.works deployed app
3. Verify no styling conflicts
4. Test on target devices/browsers
5. Remediate if issues found, iterate

---

## 9. Documentation Requirements

### 9.1 Form-Level Documentation
Each form must document:
- Layout type used (NavigationDrawer or NavigationRail)
- Navigation destinations (if layout form)
- Key user flows
- Feature flag dependencies
- M3 component choices and rationale

### 9.2 Component Customization
Document any:
- Custom roles applied
- Theme color overrides
- Non-standard layout patterns
- Workarounds for M3 limitations

---

## 10. Migration from Legacy Themes

### 10.1 Phased Migration Approach
1. **Phase 1**: Add M3 dependency, create new layout templates
2. **Phase 2**: Migrate high-traffic forms first
3. **Phase 3**: Update remaining forms
4. **Phase 4**: Remove legacy theme dependency

### 10.2 Component Mapping
| Legacy Component | M3 Replacement | Notes |
|-----------------|----------------|-------|
| Label | Text or Heading | Set style and scale properties |
| Link | NavigationLink | Use in layout navigation slots |
| Button | Button | Set role: filled/outlined/text |
| TextBox | TextBox | Set role='outlined' |
| TextArea | TextArea | Set role='outlined' |
| DropDown | DropdownMenu | Set role='outlined' |
| DatePicker | DatePicker | Set role='outlined' |
| FileLoader | FileLoader | Default styling sufficient |

---

## 11. Common Pitfalls & Solutions

### 11.1 NavigationLink Not Working
**Problem**: Clicking NavigationLink doesn't navigate
**Solution**: Ensure `navigate_to` property is set correctly; remove any click event handlers

### 11.2 Forms Not Rendering in Layout
**Problem**: Content appears behind/beside navigation
**Solution**: Add content to correct layout slot (content slot, not navigation slot)

### 11.3 Colors Not Updating
**Problem**: Theme colors don't change when scheme updated
**Solution**: Use `theme:` prefix; avoid hardcoded values; ensure M3 theme v1.2.0+

### 11.4 Input Fields Look Wrong
**Problem**: TextBox/DropdownMenu styling incorrect
**Solution**: Set `role='outlined'` on all form input components

### 11.5 Typography Doesn't Match M3
**Problem**: Text appearance inconsistent
**Solution**: Use Text/Heading components with proper style/scale properties, not Label

---

## 12. Resources & References

### 12.1 Official Documentation
- [Anvil M3 Theme Overview](https://anvil.works/docs/ui/app-themes/material-3)
- [M3 Components Reference](https://anvil.works/docs/ui/app-themes/material-3/components)
- [M3 Layouts Reference](https://anvil.works/docs/ui/app-themes/material-3/layouts)
- [Google Material 3 Guidelines](https://m3.material.io/)
- [Material Theme Builder](https://material-foundation.github.io/material-theme-builder/)

### 12.2 Example Apps
- [Virtual Plant Shop](https://anvil.works/blog/introducing-material-3) (NavigationRailLayout)
- [Expense Approval](https://anvil.works/blog/introducing-material-3) (NavigationDrawerLayout)
- [Task Manager](https://anvil.works/blog/introducing-material-3) (ButtonMenu, NavigationDrawer)

### 12.3 Community Support
- [Anvil Community Forum](https://anvil.works/forum)
- [M3 Theme GitHub](https://github.com/anvil-works/material-3-theme)
- [M3 Changelog](https://github.com/anvil-works/material-3-theme/blob/master/CHANGELOG.md)

---

**Compliance Status**: MANDATORY  
**Review Frequency**: Quarterly or on M3 theme major version updates  
**Owner**: MyBizz Development Team

