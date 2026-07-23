# Anvil Navigation & Routing Code of Practice (M3)

**Version:** 3.0  
**Date:** February 3, 2026  
**Purpose:** Define navigation standards for mybizz using M3 and Routing  
**Authority:** [Anvil M3 Docs](https://anvil.works/docs/ui/app-themes/material-3) | [Routing Docs](https://anvil.works/docs/client/navigation/routing)

---

## Core Architecture

mybizz uses **two navigation systems**:

1. **M3 NavigationLink** - Internal SPA navigation (authenticated areas)
2. **Routing Dependency** - URL-based navigation (public pages, shareable content)

---

## 1. Internal Navigation (M3 NavigationLink)

**Use For:** Authenticated admin/customer interfaces where URLs are not needed

### 1.1 Implementation

```python
# AdminLayout.py
class AdminLayout(NavigationDrawerLayoutTemplate):
    def __init__(self, **properties):
        self.init_components(**properties)
        self.build_navigation()
    
    def build_navigation(self):
        # Set navigate_to property (NO click handlers)
        self.nav_dashboard.navigate_to = 'DashboardForm'
        self.nav_bookings.navigate_to = 'BookingListForm'
        
        # Feature-based visibility
        features = anvil.server.call('get_features')
        self.nav_bookings.visible = features.get('bookings_enabled')
```

### 1.2 NavigationLink Properties

| Property | Required | Purpose |
|----------|----------|---------|
| `navigate_to` | Yes | Form name (string) or Form instance |
| `text` | Yes | Link label |
| `icon` | Recommended | Material icon name |
| `selected` | No | Highlight state (auto-managed) |
| `visible` | No | Feature flag control |

### 1.3 Standards

- **Never** use click handlers when `navigate_to` is set
- **Never** call `open_form()` in NavigationLink click handlers
- Use `nav_` prefix for all NavigationLinks
- Set `navigate_to` in Designer properties panel

**Reference:** [M3 NavigationLink Docs](https://anvil.works/docs/ui/app-themes/material-3/components#navigationlink)

---

## 2. URL Routing (Routing Dependency)

**Use For:** Public pages, shareable content, bookmarkable URLs, browser history

### 2.1 Installation

**Dependency ID:** `3PIDO5P3H4VPEMPL`

Add via Settings → Dependencies → Third Party

**Reference:** [Routing Installation](https://anvil.works/docs/client/navigation/routing#installation)

### 2.2 Basic Route Definition

```python
# In startup module or layout
from routing import router

# Simple route
@router.route("/")
class HomePage(HomePageTemplate):
    def __init__(self, routing_context, **properties):
        self.init_components(**properties)

# Route with parameters
@router.route("/products/:id")
class ProductDetail(ProductDetailTemplate):
    def __init__(self, routing_context, **properties):
        self.product_id = routing_context.params['id']
        self.init_components(**properties)
        self.load_product()
    
    def load_product(self):
        product = anvil.server.call('get_product', self.product_id)
        self.lbl_name.text = product['name']
```

### 2.3 Route Patterns

| Pattern | Example | Use Case |
|---------|---------|----------|
| Static | `/about` | Fixed pages |
| Parameter | `/products/:id` | Dynamic content |
| Query string | `/search?q=yoga` | Filters, search |
| Hash | `/article#comments` | Page sections |

### 2.4 Navigation with Routing

```python
from routing.router import navigate

# Navigate to path
def view_product_click(self, **event_args):
    navigate(path="/products/:id", params={"id": 123})

# Navigate with query
navigate(path="/search", query={"q": "yoga"})

# Replace current URL (no history entry)
navigate(path="/login", replace=True)
```

### 2.5 Navigation Components

```python
from routing.components import NavLink

# In Form designer
self.link_home = NavLink(
    text="Home",
    path="/",
    icon="fa:home"
)

self.link_products = NavLink(
    text="Products",
    path="/products",
    exact_path=False  # Matches /products, /products/123, etc.
)
```

**Reference:** [Routing Navigation Docs](https://routing-docs.anvil.works/navigating/)

---

## 3. Combining M3 + Routing

### 3.1 Architecture Pattern

```python
# Authenticated areas: M3 NavigationLink (no URLs)
AdminLayout (NavigationDrawerLayout)
├─ nav_dashboard → DashboardForm
├─ nav_bookings → BookingListForm
└─ nav_settings → SettingsForm

# Public areas: Routing (with URLs)
@router.route("/")           → HomePage
@router.route("/products")   → ProductCatalog
@router.route("/blog/:slug") → BlogPost
@router.route("/contact")    → ContactForm
```

### 3.2 Mixed Navigation Example

```python
# Public homepage with routing
@router.route("/")
class HomePage(HomePageTemplate):
    def __init__(self, routing_context, **properties):
        self.init_components(**properties)
    
    def btn_login_click(self, **event_args):
        # After auth, switch to M3 internal navigation
        user = anvil.users.login_with_form()
        if user:
            open_form('AdminLayout')  # Now using M3 NavigationLinks

# Admin area uses M3 NavigationLink
class AdminLayout(NavigationDrawerLayoutTemplate):
    def __init__(self, **properties):
        self.init_components(**properties)
        # M3 NavigationLinks handle all internal nav
        self.nav_dashboard.navigate_to = 'DashboardForm'
```

### 3.3 Public Navigation with M3 Components

```python
# Use M3 components in routed Forms
from routing import router
import m3.components as m3

@router.route("/products/:id")
class ProductDetail(ProductDetailTemplate):
    def __init__(self, routing_context, **properties):
        self.init_components(**properties)
        
        # Use M3 components for styling
        self.card_product = m3.Card(role='outlined')
        self.btn_buy = m3.Button(text="Buy Now", role='filled-button')
```

**Reference:** [Routing + Layouts Guide](https://anvil.works/docs/client/navigation/routing/quickstart)

---

## 4. Authentication & Protected Routes

### 4.1 Route Guards

```python
from routing.router import Route, hooks, Redirect

class AuthenticatedRoute(Route):
    @hooks.before_load
    def check_auth(self, **loader_args):
        user = anvil.users.get_user()
        if not user:
            raise Redirect(path="/login")
        return {"user": user}

# Apply to specific routes
class DashboardRoute(AuthenticatedRoute):
    path = "/dashboard"
    form = "DashboardForm"
```

### 4.2 Login Flow

```python
# Public login page (routed)
@router.route("/login")
class LoginForm(LoginFormTemplate):
    def btn_login_click(self, **event_args):
        user = anvil.users.login_with_form()
        if user:
            # Redirect to admin (M3 navigation)
            open_form('AdminLayout')

# Admin layout (M3 NavigationLink)
class AdminLayout(NavigationDrawerLayoutTemplate):
    def __init__(self, **properties):
        self.init_components(**properties)
        self.check_auth()
        self.build_navigation()
    
    def check_auth(self):
        if not anvil.users.get_user():
            navigate(path="/login")
```

**Reference:** [Routing Hooks Docs](https://routing-docs.anvil.works/routes/hooks/)

---

## 5. mybizz Navigation Standards

### 5.1 Admin/Customer Areas

**Use:** M3 NavigationDrawerLayout + NavigationLink

```python
AdminLayout (NavigationDrawerLayout)
├─ Dashboard (always)
├─ Sales & Operations
│  ├─ nav_bookings (if bookings_enabled)
│  ├─ nav_products (if ecommerce_enabled)
│  └─ nav_orders (if ecommerce_enabled)
├─ Customers & Marketing
│  ├─ nav_contacts (always)
│  └─ nav_campaigns (if marketing_enabled)
└─ Settings (always)
```

### 5.2 Public Pages

**Use:** Routing Dependency

```python
Routes:
/                    → HomePage
/products            → ProductCatalog
/products/:id        → ProductDetail
/booking             → BookingForm
/blog                → BlogList
/blog/:slug          → BlogPost
/contact             → ContactForm
```

### 5.3 Naming Conventions

**M3 NavigationLinks:**
- Prefix: `nav_`
- Example: `nav_dashboard`, `nav_bookings`

**Routing NavLinks:**
- Prefix: `link_`
- Example: `link_home`, `link_products`

---

## 6. Parameter Passing

### 6.1 M3 NavigationLink (No Parameters)

```python
# NavigationLinks cannot pass parameters
# For parameterized navigation, use open_form()

def customer_row_click(self, **event_args):
    customer_id = sender.item['id']
    open_form('CustomerDetailForm', customer_id=customer_id)
```

### 6.2 Routing (URL Parameters)

```python
# URL parameters
@router.route("/products/:id")
class ProductDetail(ProductDetailTemplate):
    def __init__(self, routing_context, **properties):
        self.product_id = routing_context.params['id']

# Query parameters
@router.route("/search")
class SearchResults(SearchResultsTemplate):
    def __init__(self, routing_context, **properties):
        self.query = routing_context.query.get('q', '')
```

---

## 7. Browser History & Back Button

### 7.1 M3 NavigationLink

- **No browser history** (SPA navigation)
- **No back button support**
- Acceptable for admin interfaces

### 7.2 Routing

- **Full browser history**
- **Back/forward buttons work**
- **Bookmarkable URLs**
- Required for public pages

---

## 8. Testing Requirements

### 8.1 M3 Navigation Tests

- [ ] NavigationLinks have `navigate_to` property set
- [ ] No click handlers on NavigationLinks
- [ ] Selected state highlights correctly
- [ ] Feature flags hide/show links
- [ ] Mobile responsive (drawer collapses)

### 8.2 Routing Tests

- [ ] All routes resolve correctly
- [ ] URL parameters parsed properly
- [ ] Browser back/forward buttons work
- [ ] Bookmarked URLs load correctly
- [ ] Protected routes redirect to login
- [ ] 404 handling for invalid routes

---

## 9. Common Patterns

### 9.1 Public Homepage → Admin Dashboard

```python
# Public route
@router.route("/")
class HomePage(HomePageTemplate):
    def btn_login_click(self, **event_args):
        user = anvil.users.login_with_form()
        if user:
            open_form('AdminLayout')  # Switch to M3 nav

# Admin uses M3 NavigationLink
class AdminLayout(NavigationDrawerLayoutTemplate):
    def __init__(self, **properties):
        self.init_components(**properties)
        self.nav_dashboard.navigate_to = 'DashboardForm'
```

### 9.2 Shareable Product Link

```python
# Routed product detail (shareable URL)
@router.route("/products/:id")
class ProductDetail(ProductDetailTemplate):
    def __init__(self, routing_context, **properties):
        self.product_id = routing_context.params['id']
        self.init_components(**properties)
    
    def btn_share_click(self, **event_args):
        url = f"https://yourdomain.com/products/{self.product_id}"
        # Copy to clipboard, share via social media, etc.
```

### 9.3 Admin Viewing Public Page

```python
# In AdminLayout (M3 navigation)
class AdminLayout(NavigationDrawerLayoutTemplate):
    def btn_view_site_click(self, **event_args):
        # Navigate to public routed page
        navigate(path="/")
```

---

## 10. Decision Matrix

| Requirement | Solution |
|-------------|----------|
| Internal admin navigation | M3 NavigationLink |
| Internal customer portal nav | M3 NavigationLink |
| Shareable product URLs | Routing |
| Bookmarkable blog posts | Routing |
| Browser back/forward support | Routing |
| Public pages (SEO, sharing) | Routing |
| No URL needed | M3 NavigationLink |
| Feature flag visibility | M3 NavigationLink |

---

## 11. Migration Guide

### 11.1 From Custom Sidebar to M3

```python
# Before (custom sidebar)
class AdminLayout(AdminLayoutTemplate):
    def link_dashboard_click(self, **event_args):
        open_form('DashboardForm')

# After (M3 NavigationLink)
class AdminLayout(NavigationDrawerLayoutTemplate):
    def __init__(self, **properties):
        self.init_components(**properties)
        self.nav_dashboard.navigate_to = 'DashboardForm'
```

### 11.2 Adding Routing to Existing App

1. Install Routing dependency: `3PIDO5P3H4VPEMPL`
2. Migrate to Layouts (required for Routing)
3. Add routes to public pages
4. Keep M3 NavigationLink for admin areas
5. Test browser history functionality

**Reference:** [Porting to Layouts](https://anvil.works/docs/how-to/porting-app-to-new-layouts)

---

## 12. Anti-Patterns

### ❌ Don't Use Routing for Admin Navigation

```python
# ❌ WRONG: Routing in admin area
@router.route("/admin/dashboard")
class DashboardForm...

# ✅ RIGHT: M3 NavigationLink in admin
self.nav_dashboard.navigate_to = 'DashboardForm'
```

### ❌ Don't Use NavigationLink for Public Pages

```python
# ❌ WRONG: No shareable URL
self.nav_products.navigate_to = 'ProductCatalog'

# ✅ RIGHT: Routed public page
@router.route("/products")
class ProductCatalog...
```

### ❌ Don't Mix Navigation Methods

```python
# ❌ WRONG: Redundant navigation
self.nav_dashboard.navigate_to = 'DashboardForm'
def nav_dashboard_click(self, **event_args):
    open_form('DashboardForm')

# ✅ RIGHT: Let navigate_to handle it
self.nav_dashboard.navigate_to = 'DashboardForm'
```

---

## 13. References

### Official Documentation
- [Anvil M3 Theme](https://anvil.works/docs/ui/app-themes/material-3)
- [M3 NavigationLink](https://anvil.works/docs/ui/app-themes/material-3/components#navigationlink)
- [M3 Layouts](https://anvil.works/docs/ui/app-themes/material-3/layouts)
- [Routing Dependency](https://anvil.works/docs/client/navigation/routing)
- [Routing Documentation](https://routing-docs.anvil.works/)
- [Navigation Guide](https://anvil.works/docs/client/navigation)

### GitHub
- [M3 Theme GitHub](https://github.com/anvil-works/material-3-theme)
- [Routing GitHub](https://github.com/anvil-works/routing)

---

**Status:** ✅ MANDATORY  
**Review:** After M3 or Routing major version updates  
**Owner:** mybizz Development Team
