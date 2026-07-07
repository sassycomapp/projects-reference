# `data-access-patterns` ADR — Data Access Patterns and Query Limitations

**Status:** Approved  
**Date:** 2026-06-14  
**Authority:** Derived from platform-overview.md, `anvil-platform-constraints` ADR, authoritative-schema.md

---

## Context

Anvil Data Tables are backed by PostgreSQL but expose a simplified query API. The query model does not support:

- SQL JOINs across tables
- Complex aggregations (GROUP BY, HAVING, window functions)
- Subqueries
- Raw SQL execution

All data access goes through Anvil's `app_tables` API, which provides `.search()`, `.get()`, `.get_by_id()`, and `.limit_to()` methods. These are adequate for simple lookups but insufficient for complex reporting and analytics.

---

## Decision

### Data Access Rules

1. **All data access is server-side.** Client code never reads Data Tables directly. All queries go through server functions (`client-instance-architecture` ADR).

2. **Use `.search()` with filters, not iteration.** Never iterate over all rows. Always use `.search()` with keyword arguments to filter, or `.limit_to()` to bound the result set.

3. **Denormalize where needed.** Since JOINs are not available, frequently accessed related data is denormalized into the primary table. For example, `bookings` stores `contact_name` alongside `contact_id` to avoid a second lookup.

4. **Pre-compute aggregations.** Complex aggregations (revenue by service, booking completion rates) are computed by background tasks and stored in summary tables, not computed on-the-fly.

5. **Server-side filtering for lists.** When a form needs to display a filtered list (e.g., bookings for a specific date range), the filter is applied server-side in a server function, not client-side after loading all rows.

### Query Patterns

**Pattern 1: Simple lookup**
```python
# Server module
def get_contact(contact_id):
    return app_tables.contacts.get_by_id(contact_id)
```

**Pattern 2: Filtered search**
```python
# Server module
def get_bookings_for_date(date_str):
    return app_tables.bookings.search(
        booking_date=date_str,
        order_by='-created_at'
    )
```

**Pattern 3: Denormalized read**
```python
# Server module — bookings table has contact_name stored directly
def get_booking_list():
    bookings = app_tables.bookings.search(order_by='-booking_date')
    return [
        {
            'id': b.get_id(),
            'contact_name': b.contact_name,  # Denormalized — no second lookup
            'service': b.service_name,        # Denormalized
            'date': b.booking_date,
            'amount': b.total_amount,
            'status': b.status
        }
        for b in bookings
    ]
```

**Pattern 4: Pre-computed aggregation**
```python
# Background task — runs daily
@anvil.server.background_task
def compute_daily_revenue():
    today = date.today()
    bookings = app_tables.bookings.search(booking_date=today.isoformat())
    
    total = sum(b.total_amount for b in bookings if b.status == 'confirmed')
    by_service = {}
    for b in bookings:
        if b.status == 'confirmed':
            svc = b.service_name
            by_service[svc] = by_service.get(svc, 0) + b.total_amount
    
    app_tables.daily_summary.add_row(
        date=today.isoformat(),
        total_revenue=total,
        revenue_by_service=json.dumps(by_service),
        booking_count=len([b for b in bookings if b.status == 'confirmed'])
    )
```

**Pattern 5: Bounded pagination**
```python
# Server module
def get_contacts_page(cursor=None, page_size=50):
    if cursor:
        return app_tables.contacts.search(
            order_by='last_name',
            limit_to_page_size=page_size,
            starting_after=cursor
        )
    else:
        return app_tables.contacts.search(
            order_by='last_name',
            limit_to_page_size=page_size
        )
```

### Summary Tables

For reporting and analytics, pre-computed summary tables store aggregated data:

| Summary Table | Purpose | Computed By |
|---|---|---|
| `daily_summary` | Revenue, booking count, new contacts per day | Daily background task |
| `service_performance` | Revenue and booking count per service | Daily background task |
| `provider_performance` | Revenue and booking count per provider | Daily background task |
| `campaign_metrics` | Open rates, click rates, conversions per campaign | Hourly background task |

These tables are populated by background tasks (`real-time-and-background-tasks` ADR) and read directly by dashboard and reporting forms.

---

## Consequences

- ✅ All data access follows a consistent, server-side pattern
- ✅ Denormalization avoids the JOIN limitation for frequently accessed data
- ✅ Pre-computed aggregations enable fast dashboard loading
- ✅ Bounded pagination prevents timeout on large result sets
- ⚠️ Denormalization introduces data consistency risk — denormalized fields must be updated when source data changes
- ⚠️ Summary tables require background task scheduling — adds to per-client task load (`real-time-and-background-tasks` ADR)
- ⚠️ Complex ad-hoc queries are not possible — reporting is limited to pre-computed summaries

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/anvil-platform-constraints.md`` | Data Table limitations and lazy-load behavior |
| ``adr/adr-global/client-instance-architecture.md`` | All data access is server-side |
| ``adr/adr-global/real-time-and-background-tasks.md`` | Background tasks compute summary tables |
| `docs/authoritative-schema.md` | 36-table schema definition |
| `docs/platform-overview.md` | Section 13: Data Architecture |

---

*End of `data-access-patterns` ADR*
