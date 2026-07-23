# Anvil Data Tables - Code of Practice for Mybizz

**Version:** 1.0  
**Date:** January 18, 2026  
**Purpose:** Definitive standard for working with Anvil Data Tables in Mybizz  
**Audience:** AI developers building Mybizz on Anvil.works

---

## CORE PRINCIPLES

1. **Anvil manages primary keys automatically** - Every row has a built-in unique ID
2. **Use Row objects, not integer IDs** - Tables link using Row objects directly
3. **Always filter by instance_id** - Enforce multi-tenant data isolation
4. **Use transactions for counters** - If you need sequential IDs, use `@tables.in_transaction`
5. **Trust Anvil's optimization** - No manual index management needed

---

## 1. ROW IDENTIFICATION

### ✅ THE ANVIL WAY (Correct)

Every row automatically has a unique identifier:

```python
# Create a row
contact = app_tables.contacts.add_row(
  instance_id=user,
  first_name='Jane',
  email='jane@example.com'
)

# Get the unique ID
row_id = contact.get_id()  # Returns tuple like [169162, 297786594]

# Use the Row object directly (don't store the ID)
app_tables.bookings.add_row(
  instance_id=user,
  contact=contact,  # Store Row object, not ID
  booking_date=datetime.now()
)
```

### ❌ WHAT NOT TO DO

```python
# ❌ DON'T create contact_id columns expecting auto-increment
app_tables.contacts.add_row(
  contact_id=None,  # This won't auto-generate
  first_name='Jane'
)

# ❌ DON'T try to store integer IDs manually
contact_id = contact.get_id()
app_tables.bookings.add_row(
  contact_id=contact_id  # This stores a tuple, not useful
)
```

### ⚠️ IF YOU NEED HUMAN-READABLE IDs

Only implement this if you need visible IDs like "CONT-0001", "INV-2024-001":

**Step 1: Create counters table**
```
Table: counters
- counter_name (string)
- next_value (number)
```

**Step 2: Add counter rows**
```python
app_tables.counters.add_row(counter_name='contacts', next_value=1)
app_tables.counters.add_row(counter_name='invoices', next_value=1)
```

**Step 3: Implement counter function**
```python
import anvil.tables as tables
from anvil.tables import app_tables

@tables.in_transaction
def get_next_contact_number():
  """Thread-safe counter increment"""
  counter = app_tables.counters.get(counter_name='contacts')
  current = counter['next_value']
  counter['next_value'] += 1
  return f"CONT-{current:05d}"

# Use it
visible_id = get_next_contact_number()  # "CONT-00001"
app_tables.contacts.add_row(
  display_id=visible_id,  # Human-readable string
  instance_id=user,
  first_name='Jane'
)
```

**Mybizz Decision:** For V1.x, we use Anvil's built-in Row IDs. No custom ID columns unless specifically required for invoices/booking numbers.

---

## 2. TABLE LINKING

### ✅ CORRECT LINK PATTERN

```python
# Create parent record
contact = app_tables.contacts.add_row(
  instance_id=user,
  email='john@example.com'
)

# Link child record - store the Row object directly
booking = app_tables.bookings.add_row(
  instance_id=user,
  contact=contact,  # Column type: link_single
  booking_date=datetime.now()
)

# Access linked data
print(booking['contact']['email'])  # 'john@example.com'

# Search by linked record
bookings = app_tables.bookings.search(contact=contact)
```

### ❌ WHAT NOT TO DO

```python
# ❌ DON'T store IDs manually
contact_id = contact.get_id()
booking = app_tables.bookings.add_row(
  contact_id=contact_id  # Wrong approach
)

# ❌ DON'T try to query by ID tuple
app_tables.bookings.search(contact_id=[169162, 297786594])  # Won't work
```

### Column Types for Links

In Data Tables editor:
- **Single link:** Type = "Link to Table" → Choose table → "Single row"
- **Multiple links:** Type = "Link to Table" → Choose table → "List of rows"

In anvil.yaml:
```yaml
bookings:
  columns:
  - name: contact
    target: contacts
    type: link_single  # Single row link
    
events:
  columns:
  - name: attendees
    target: contacts
    type: link_multiple  # Multiple rows link
```

---

## 3. DATA ISOLATION (Multi-Tenant Security)

### ✅ MANDATORY PATTERN

Every table MUST have `instance_id` column linked to `users` table.

**Every query MUST filter by instance_id:**

```python
import anvil.users
from anvil.tables import app_tables

@anvil.server.callable
def get_all_contacts():
  """CORRECT - Filters by current user's instance"""
  user = anvil.users.get_user()
  if not user:
    raise Exception("Not authenticated")
  
  # ALWAYS filter by instance_id
  return app_tables.contacts.search(instance_id=user)

@anvil.server.callable
def get_contact_by_email(email):
  """CORRECT - Filters by instance_id AND email"""
  user = anvil.users.get_user()
  if not user:
    raise Exception("Not authenticated")
  
  return app_tables.contacts.get(
    instance_id=user,
    email=email
  )
```

### ❌ SECURITY VIOLATION

```python
# ❌ NEVER query without instance_id filter
@anvil.server.callable
def get_all_contacts():
  return app_tables.contacts.search()  # EXPOSES ALL INSTANCES' DATA!

# ❌ NEVER trust client-provided instance_id
@anvil.server.callable
def get_contact(instance_id, email):
  return app_tables.contacts.get(
    instance_id=instance_id,  # Client could pass any instance_id!
    email=email
  )
```

### Standard Security Pattern

```python
def get_current_user_or_raise():
  """Helper function - use in all server functions"""
  user = anvil.users.get_user()
  if not user:
    raise Exception("Authentication required")
  return user

@anvil.server.callable
def create_contact(contact_data):
  user = get_current_user_or_raise()
  
  return app_tables.contacts.add_row(
    instance_id=user,  # ALWAYS set from server
    **contact_data
  )
```

---

## 4. CRUD OPERATIONS

### CREATE

```python
@anvil.server.callable
def create_contact(first_name, last_name, email):
  user = anvil.users.get_user()
  
  # Returns the new Row object
  new_contact = app_tables.contacts.add_row(
    instance_id=user,
    first_name=first_name,
    last_name=last_name,
    email=email,
    status='Lead',
    created_at=datetime.now()
  )
  
  return new_contact
```

### READ (Single Record)

```python
@anvil.server.callable
def get_contact_by_email(email):
  user = anvil.users.get_user()
  
  # Returns single Row or None
  contact = app_tables.contacts.get(
    instance_id=user,
    email=email
  )
  
  # Raises exception if multiple matches found
  return contact
```

### READ (Multiple Records)

```python
@anvil.server.callable
def get_all_customers():
  user = anvil.users.get_user()
  
  # Returns SearchIterator (lazy-loaded)
  customers = app_tables.contacts.search(
    instance_id=user,
    status='Customer'
  )
  
  return customers  # Can iterate, slice, or convert to list

@anvil.server.callable
def get_active_contacts():
  user = anvil.users.get_user()
  
  # SearchIterator supports Python operations
  contacts = app_tables.contacts.search(
    instance_id=user,
    status=q.any_of('Lead', 'Customer')
  )
  
  # Convert to list if needed
  return list(contacts)
```

### UPDATE

```python
@anvil.server.callable
def update_contact(contact, updates):
  """
  contact: Row object passed from client
  updates: dict of fields to update
  """
  user = anvil.users.get_user()
  
  # Verify ownership (security check)
  if contact['instance_id'] != user:
    raise Exception("Access denied")
  
  # Update fields
  for key, value in updates.items():
    contact[key] = value
  
  contact['updated_at'] = datetime.now()
  
  # Changes auto-save to database
  return contact

# Alternative: Direct update
@anvil.server.callable
def update_contact_status(email, new_status):
  user = anvil.users.get_user()
  
  contact = app_tables.contacts.get(
    instance_id=user,
    email=email
  )
  
  if contact:
    contact['status'] = new_status
    contact['updated_at'] = datetime.now()
  
  return contact
```

### DELETE

```python
@anvil.server.callable
def delete_contact(contact):
  """contact: Row object"""
  user = anvil.users.get_user()
  
  # Verify ownership
  if contact['instance_id'] != user:
    raise Exception("Access denied")
  
  # Delete the row
  contact.delete()
  
  return True
```

---

## 5. SEARCH & QUERY PATTERNS

### Basic Search

```python
from anvil.tables import app_tables
import anvil.tables.query as q

# All rows matching criteria
contacts = app_tables.contacts.search(
  instance_id=user,
  status='Customer'
)

# Get single row (None if not found, exception if multiple)
contact = app_tables.contacts.get(
  instance_id=user,
  email='jane@example.com'
)
```

### Query Operators

```python
# ANY_OF - Match any of these values
leads_or_customers = app_tables.contacts.search(
  instance_id=user,
  status=q.any_of('Lead', 'Customer', 'Active')
)

# NONE_OF - Exclude these values
not_inactive = app_tables.contacts.search(
  instance_id=user,
  status=q.none_of('Inactive', 'Lost')
)

# GREATER_THAN, LESS_THAN
high_value = app_tables.contacts.search(
  instance_id=user,
  total_spent=q.greater_than(5000)
)

recent = app_tables.contacts.search(
  instance_id=user,
  last_contact_date=q.greater_than(datetime.now() - timedelta(days=30))
)

# BETWEEN
mid_range = app_tables.contacts.search(
  instance_id=user,
  total_spent=q.between(1000, 5000)
)

# FULL_TEXT_MATCH - Case-insensitive search
search_results = app_tables.contacts.search(
  instance_id=user,
  q.full_text_match('jane')
)

# LIKE - Pattern matching (use sparingly, slower)
gmail_users = app_tables.contacts.search(
  instance_id=user,
  email=q.like('%@gmail.com')
)
```

### Sorting

```python
import anvil.tables as tables

# Sort ascending (default)
oldest_first = app_tables.contacts.search(
  instance_id=user,
  tables.order_by('created_at')
)

# Sort descending
newest_first = app_tables.contacts.search(
  instance_id=user,
  tables.order_by('created_at', ascending=False)
)

# Multiple sort criteria
sorted_contacts = app_tables.contacts.search(
  instance_id=user,
  tables.order_by('status'),  # First by status
  tables.order_by('last_name')  # Then by last_name
)

# Sort by linked table column
bookings_by_contact_name = app_tables.bookings.search(
  instance_id=user,
  tables.order_by('contact.last_name')
)
```

### Pagination

```python
# Get first 10 results
first_page = app_tables.contacts.search(
  instance_id=user,
  tables.order_by('created_at', ascending=False)
)[:10]

# Get next 10 (page 2)
second_page = app_tables.contacts.search(
  instance_id=user,
  tables.order_by('created_at', ascending=False)
)[10:20]

# Skip first 20, get 10 more (page 3)
third_page = app_tables.contacts.search(
  instance_id=user,
  tables.order_by('created_at', ascending=False)
)[20:30]
```

### Counting

```python
# Count all matches
total_contacts = len(
  app_tables.contacts.search(instance_id=user)
)

# More efficient counting for large tables
total_customers = len(
  app_tables.contacts.search(
    instance_id=user,
    status='Customer'
  )
)
```

### Complex Queries

```python
# Combine multiple conditions
results = app_tables.contacts.search(
  instance_id=user,
  status=q.any_of('Lead', 'Customer'),
  total_spent=q.greater_than(1000),
  last_contact_date=q.greater_than(datetime.now() - timedelta(days=90)),
  tables.order_by('total_spent', ascending=False)
)[:50]  # Top 50 results
```

---

## 6. TRANSACTIONS

### When to Use Transactions

Use `@tables.in_transaction` for:
- Incrementing counters (preventing race conditions)
- Multi-step operations that must succeed together
- Operations requiring consistency

### Transaction Pattern

```python
import anvil.tables as tables
from anvil.tables import app_tables

@tables.in_transaction
def create_order_with_items(order_data, items):
  """Both order and items created atomically"""
  user = anvil.users.get_user()
  
  # Create order
  order = app_tables.orders.add_row(
    instance_id=user,
    order_number=order_data['order_number'],
    total=order_data['total'],
    status='pending'
  )
  
  # Create all items
  for item in items:
    app_tables.order_items.add_row(
      instance_id=user,
      order=order,  # Link to order
      product=item['product'],
      quantity=item['quantity'],
      price=item['price']
    )
  
  # If any step fails, entire transaction rolls back
  return order

@tables.in_transaction
def transfer_booking(booking, new_contact):
  """Move booking from one contact to another"""
  user = anvil.users.get_user()
  
  # Verify ownership
  if booking['instance_id'] != user:
    raise Exception("Access denied")
  
  old_contact = booking['contact']
  
  # Update booking
  booking['contact'] = new_contact
  
  # Log event for both contacts
  app_tables.contact_events.add_row(
    instance_id=user,
    contact=old_contact,
    event_type='booking_transferred',
    event_date=datetime.now()
  )
  
  app_tables.contact_events.add_row(
    instance_id=user,
    contact=new_contact,
    event_type='booking_received',
    event_date=datetime.now()
  )
  
  return booking
```

### Transaction Behavior

- Fully serializable (prevents conflicts)
- Automatically retries on conflict
- All changes committed together or rolled back
- Deadlock protection built-in

---

## 7. COLUMN TYPES

### Available Types

| Anvil Type | Python Type | Use Case |
|------------|-------------|----------|
| **Text** | `str` | Names, emails, descriptions |
| **Number** | `int`, `float` | Quantities, prices, IDs |
| **True/False** | `bool` | Flags, toggles |
| **Date** | `datetime.date` | Birthdays, deadlines |
| **Date and Time** | `datetime.datetime` | Timestamps, created_at |
| **Simple Object** | `dict`, `list` | JSON data, settings, tags |
| **Media** | Media object | Files, images, PDFs |
| **Link (Single)** | Row object | Foreign key to one row |
| **Link (Multiple)** | List of Rows | Many-to-many relationships |

### Simple Object Usage

Store JSON-like data:

```python
# Store dict
contact = app_tables.contacts.add_row(
  instance_id=user,
  first_name='Jane',
  preferences={
    'newsletter': True,
    'sms_alerts': False,
    'favorite_color': 'blue'
  },
  tags=['VIP', 'Newsletter', 'Repeat Customer']
)

# Access
print(contact['preferences']['newsletter'])  # True
print(contact['tags'])  # ['VIP', 'Newsletter', 'Repeat Customer']

# Update
contact['tags'].append('Birthday Club')
contact['preferences']['sms_alerts'] = True
```

### Media Type Usage

```python
# Store file
file = anvil.media.from_file(file_upload_component.file)

contact = app_tables.contacts.add_row(
  instance_id=user,
  first_name='Jane',
  profile_photo=file  # Media type column
)

# Retrieve
photo = contact['profile_photo']
if photo:
  download_link = photo.url
  file_size = photo.length
  content_type = photo.content_type
```

---

## 8. DATA TABLE PERMISSIONS

### Permission Levels

Set in Data Tables editor for each table:

- **No access** (default) - Client code cannot access
- **Search only** - Client can read, cannot modify
- **Search, edit and delete** - Client has full access

### Security Best Practice

**For Mybizz:** All tables set to "No access" for client code.

**Access via server functions only:**

```python
# ✅ CORRECT - All data access through server functions
@anvil.server.callable
def get_contacts(filters):
  user = anvil.users.get_user()
  # Apply security, filtering, validation here
  return app_tables.contacts.search(instance_id=user, **filters)

# ❌ WRONG - Direct client access bypasses security
# (Don't set table permissions to allow client access)
```

### Exception: Public Forms

Only for truly public data (no instance_id):

```python
# Public newsletter signup - no authentication required
@anvil.server.callable
def subscribe_to_newsletter(email):
  # Validate email
  if not is_valid_email(email):
    raise Exception("Invalid email")
  
  # Check if already subscribed
  existing = app_tables.newsletter_subscribers.get(email=email)
  if existing:
    return "Already subscribed"
  
  app_tables.newsletter_subscribers.add_row(
    email=email,
    subscribed_date=datetime.now()
  )
  
  return "Subscribed successfully"
```

---

## 9. PERFORMANCE OPTIMIZATION

### Automatic Optimization

Anvil automatically:
- Indexes all link columns
- Caches frequently accessed data
- Optimizes common query patterns
- Manages connection pooling

### Do NOT

- ❌ Try to create custom indexes
- ❌ Manually cache Row objects (Anvil handles this)
- ❌ Optimize prematurely

### DO

- ✅ Use lazy-loading SearchIterators
- ✅ Paginate large result sets
- ✅ Filter early in queries
- ✅ Use `get()` instead of `search()[0]` when retrieving single record

### Efficient Patterns

```python
# ✅ EFFICIENT - Get single record
contact = app_tables.contacts.get(
  instance_id=user,
  email=email
)

# ❌ INEFFICIENT - Load all then take first
contact = app_tables.contacts.search(
  instance_id=user,
  email=email
)[0]

# ✅ EFFICIENT - Lazy loading
for contact in app_tables.contacts.search(instance_id=user):
  process_contact(contact)
  # Only loads records as needed

# ❌ INEFFICIENT - Load everything into memory
all_contacts = list(app_tables.contacts.search(instance_id=user))
for contact in all_contacts:
  process_contact(contact)

# ✅ EFFICIENT - Paginate large datasets
def get_contacts_page(page_size=50, page=0):
  user = anvil.users.get_user()
  start = page * page_size
  end = start + page_size
  
  return app_tables.contacts.search(
    instance_id=user,
    tables.order_by('created_at', ascending=False)
  )[start:end]
```

---

## 10. COMMON PATTERNS FOR Mybizz

### Pattern 1: Event-Driven Timeline

Track all customer interactions:

```python
def log_contact_event(contact, event_type, event_data=None, related_id=None):
  """Standard pattern for logging contact events"""
  user = anvil.users.get_user()
  
  return app_tables.contact_events.add_row(
    instance_id=user,
    contact=contact,
    event_type=event_type,
    event_date=datetime.now(),
    event_data=event_data or {},
    related_id=related_id,
    user_visible=True
  )

# Usage
booking = create_booking(booking_data)
log_contact_event(
  contact=booking['contact'],
  event_type='booking_created',
  event_data={'booking_number': booking['booking_number']},
  related_id=booking.get_id()
)
```

### Pattern 2: Automatic Contact Creation

```python
def get_or_create_contact(email, defaults=None):
  """Get existing contact or create new one"""
  user = anvil.users.get_user()
  
  contact = app_tables.contacts.get(
    instance_id=user,
    email=email
  )
  
  if contact:
    return contact
  
  # Create new contact
  contact_data = defaults or {}
  contact_data['email'] = email
  
  return app_tables.contacts.add_row(
    instance_id=user,
    status='Lead',
    date_added=datetime.now(),
    **contact_data
  )

# Usage in booking creation
@anvil.server.callable
def create_booking(booking_data):
  user = anvil.users.get_user()
  
  # Ensure contact exists
  contact = get_or_create_contact(
    email=booking_data['customer_email'],
    defaults={
      'first_name': booking_data.get('customer_name'),
      'source': 'Booking Widget'
    }
  )
  
  # Create booking with contact link
  booking = app_tables.bookings.add_row(
    instance_id=user,
    contact=contact,  # Link to contact
    **booking_data
  )
  
  # Update contact metrics
  update_contact_metrics(contact)
  
  return booking
```

### Pattern 3: Calculated Fields

```python
def update_contact_metrics(contact):
  """Recalculate contact's aggregate metrics"""
  
  # Count transactions
  total_bookings = len(
    app_tables.bookings.search(contact=contact)
  )
  total_orders = len(
    app_tables.orders.search(contact=contact)
  )
  
  # Sum revenue
  total_spent = 0
  for booking in app_tables.bookings.search(contact=contact):
    total_spent += booking['total_amount'] or 0
  for order in app_tables.orders.search(contact=contact):
    total_spent += order['total_amount'] or 0
  
  # Update contact
  total_transactions = total_bookings + total_orders
  contact['total_transactions'] = total_transactions
  contact['total_spent'] = total_spent
  contact['average_order_value'] = (
    total_spent / total_transactions if total_transactions > 0 else 0
  )
  contact['last_contact_date'] = datetime.now()
  contact['updated_at'] = datetime.now()
  
  # Update lifecycle stage
  days_since_contact = (datetime.now() - contact['last_contact_date']).days
  if total_transactions == 0:
    contact['lifecycle_stage'] = 'New'
  elif days_since_contact <= 90:
    contact['lifecycle_stage'] = 'Active'
  elif days_since_contact <= 180:
    contact['lifecycle_stage'] = 'At Risk'
  else:
    contact['lifecycle_stage'] = 'Lost'
```

### Pattern 4: Soft Delete

```python
# Instead of contact.delete(), mark as inactive
def soft_delete_contact(contact):
  """Mark contact as inactive rather than deleting"""
  user = anvil.users.get_user()
  
  if contact['instance_id'] != user:
    raise Exception("Access denied")
  
  contact['status'] = 'Deleted'
  contact['deleted_at'] = datetime.now()
  
  # Log event
  log_contact_event(
    contact=contact,
    event_type='contact_deleted',
    user_visible=False
  )
  
  return contact

# Exclude deleted from normal queries
def get_active_contacts():
  user = anvil.users.get_user()
  
  return app_tables.contacts.search(
    instance_id=user,
    status=q.none_of('Deleted', 'Inactive')
  )
```

---

## 11. COMMON MISTAKES TO AVOID

### ❌ Mistake 1: Trying to Create Primary Keys

```python
# ❌ WRONG - No auto-increment type exists
contact_id = get_next_id()  # Unnecessary!
app_tables.contacts.add_row(
  contact_id=contact_id,
  ...
)

# ✅ RIGHT - Use built-in Row ID
contact = app_tables.contacts.add_row(...)
row_id = contact.get_id()  # Automatic
```

### ❌ Mistake 2: Storing IDs Instead of Row Objects

```python
# ❌ WRONG - Storing ID
contact_id = contact.get_id()
app_tables.bookings.add_row(
  contact_id=contact_id  # Tuple stored, hard to query
)

# ✅ RIGHT - Store Row object
app_tables.bookings.add_row(
  contact=contact  # Queryable, navigable
)
```

### ❌ Mistake 3: Forgetting instance_id Filter

```python
# ❌ WRONG - Security vulnerability
@anvil.server.callable
def get_all_contacts():
  return app_tables.contacts.search()  # ALL INSTANCES!

# ✅ RIGHT - Always filter
@anvil.server.callable
def get_all_contacts():
  user = anvil.users.get_user()
  return app_tables.contacts.search(instance_id=user)
```

### ❌ Mistake 4: Converting SearchIterator Too Early

```python
# ❌ INEFFICIENT - Loads all into memory
contacts = list(app_tables.contacts.search(instance_id=user))
for contact in contacts:
  process(contact)

# ✅ EFFICIENT - Lazy loading
for contact in app_tables.contacts.search(instance_id=user):
  process(contact)
```

### ❌ Mistake 5: Not Checking Row Exists

```python
# ❌ WRONG - Will throw exception if not found
contact = app_tables.contacts.get(instance_id=user, email=email)
contact['status'] = 'Active'  # Exception if contact is None

# ✅ RIGHT - Check first
contact = app_tables.contacts.get(instance_id=user, email=email)
if contact:
  contact['status'] = 'Active'
else:
  print("Contact not found")
```

### ❌ Mistake 6: Modifying Row in Loop

```python
# ❌ WRONG - Can cause issues
for contact in app_tables.contacts.search(instance_id=user):
  contact.delete()  # Modifying while iterating

# ✅ RIGHT - Convert to list first
contacts = list(app_tables.contacts.search(instance_id=user, status='Deleted'))
for contact in contacts:
  contact.delete()
```

---

## 12. TESTING PATTERNS

### Test Data Creation

```python
def create_test_contact(user, **overrides):
  """Helper for creating test data"""
  defaults = {
    'first_name': 'Test',
    'last_name': 'Contact',
    'email': f'test{datetime.now().timestamp()}@example.com',
    'status': 'Lead',
    'source': 'Test',
    'date_added': datetime.now()
  }
  defaults.update(overrides)
  
  return app_tables.contacts.add_row(
    instance_id=user,
    **defaults
  )

# Usage in tests
test_contact = create_test_contact(
  user=current_user,
  first_name='Jane',
  status='Customer'
)
```

### Cleanup

```python
def delete_test_data(user):
  """Clean up test records"""
  test_contacts = app_tables.contacts.search(
    instance_id=user,
    source='Test'
  )
  
  for contact in list(test_contacts):
    # Delete related records first
    for event in app_tables.contact_events.search(contact=contact):
      event.delete()
    
    # Then delete contact
    contact.delete()
```

---

## 13. Mybizz-SPECIFIC STANDARDS

### Mandatory Columns

Every table MUST have:
- `instance_id` (link to users)
- `created_at` (datetime)
- `updated_at` (datetime) - if records are mutable

### Naming Conventions

- **Tables:** Lowercase, underscores: `contacts`, `order_items`
- **Columns:** Lowercase, underscores: `first_name`, `total_spent`
- **Link columns:** Singular name matching target table: `contact`, `order`, `product`
- **Status fields:** Lowercase strings: `'active'`, `'pending'`, `'completed'`

### Standard Status Values

```python
# Contacts
STATUS_LEAD = 'Lead'
STATUS_CUSTOMER = 'Customer'
STATUS_INACTIVE = 'Inactive'

# Lifecycle
LIFECYCLE_NEW = 'New'
LIFECYCLE_ACTIVE = 'Active'
LIFECYCLE_AT_RISK = 'At Risk'
LIFECYCLE_LOST = 'Lost'

# Orders
ORDER_STATUS_PENDING = 'pending'
ORDER_STATUS_PAID = 'paid'
ORDER_STATUS_SHIPPED = 'shipped'
ORDER_STATUS_COMPLETED = 'completed'
ORDER_STATUS_CANCELLED = 'cancelled'
```

### Error Handling

```python
@anvil.server.callable
def create_contact(contact_data):
  try:
    user = get_current_user_or_raise()
    
    # Validate
    if not contact_data.get('email'):
      return {'success': False, 'error': 'Email required'}
    
    # Check duplicate
    existing = app_tables.contacts.get(
      instance_id=user,
      email=contact_data['email']
    )
    if existing:
      return {'success': False, 'error': 'Contact already exists'}
    
    # Create
    contact = app_tables.contacts.add_row(
      instance_id=user,
      created_at=datetime.now(),
      **contact_data
    )
    
    return {'success': True, 'contact': contact}
    
  except Exception as e:
    print(f"Error creating contact: {str(e)}")
    return {'success': False, 'error': 'Failed to create contact'}
```

---

## QUICK REFERENCE CHECKLIST

When creating a new table:
- [ ] Add `instance_id` column (link to users)
- [ ] Add `created_at` column (datetime)
- [ ] Add `updated_at` column if mutable (datetime)
- [ ] Set permissions to "No access" for client code
- [ ] Use Row objects for relationships (not integer IDs)
- [ ] No custom primary key columns (use row.get_id())

When writing server functions:
- [ ] Get user with `anvil.users.get_user()`
- [ ] Check authentication (raise exception if None)
- [ ] ALWAYS filter by `instance_id=user`
- [ ] Validate input data
- [ ] Return structured responses (`{'success': bool, 'data': ...}`)
- [ ] Wrap in try/except with proper error handling

When querying:
- [ ] Filter by `instance_id` FIRST
- [ ] Use `get()` for single record (returns None if not found)
- [ ] Use `search()` for multiple records (returns SearchIterator)
- [ ] Apply sorting with `tables.order_by()`
- [ ] Paginate large results with slicing `[start:end]`

---

## VERSION HISTORY

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Jan 18, 2026 | Initial Code of Practice based on Anvil documentation reality check |

---

**This document is the definitive standard for Mybizz Data Tables work.**  
**All code must conform to these patterns.**  
**When in doubt, refer to this document first.**
