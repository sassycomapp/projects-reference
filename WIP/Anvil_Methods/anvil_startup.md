## Startup Module, Form
Anvil App Startup Model
- Anvil no longer uses a dedicated Startup Module
- Anvil starts by instantiating a Startup Form, not modules
- The Startup Form (e.g. HomePage) is defined in app settings
- Startup logic is typically placed in the Startup Form and should be designed to run once
- Client-side modules do not self-execute; they only run when explicitly imported and called

---


from ._anvil_designer import HomePageTemplate
from anvil import *
import anvil.server
import anvil.users

# Import your startup logic module (optional, structured apps only)
from .. import startup


class HomePage(HomePageTemplate):
  def __init__(self, **properties):
    user = anvil.users.get_user()
    if user:
      if user['role'] in ['owner', 'manager', 'admin', 'staff']:
        open_form('dashboard.DashboardForm')
        return
      else:
        open_form('customers.ClientPortalForm')
        return

    # If not logged in, continue with HomePage display
    self.init_components(**properties)
