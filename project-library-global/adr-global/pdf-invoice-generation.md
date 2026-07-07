# `pdf-invoice-generation` ADR — PDF Invoice Generation

**Status:** Approved  
**Date:** 2026-06-14  
**Authority:** Derived from platform-overview.md, integration-architecture.md

---

## Context

The platform generates invoices automatically on booking completion and supports manual invoice creation. Invoices must be:

- Generated as PDF files
- Stored in the client instance
- Emailed to customers via Brevo SMTP
- Downloadable from the admin panel

Anvil has no native PDF library. PDF generation must happen server-side (CPython), as Skulpt does not support the required libraries.

---

## Decision

### PDF Generation: Server-Side Python Library

PDF generation uses a Python PDF library running server-side. The library must:

- Run on CPython (server modules)
- Generate PDFs from structured data (not HTML rendering)
- Support text, tables, images, and basic styling
- Be installable as an Anvil server dependency or bundled with the server module

### Recommended Library: `reportlab`

`reportlab` is the most widely used Python PDF generation library. It runs on CPython, has no external dependencies, and provides fine-grained control over PDF layout.

**Why reportlab over alternatives:**

| Alternative | Reason not chosen |
|---|---|
| `weasyprint` | Requires system-level dependencies (Cairo, Pango) — not available on Anvil |
| `pdfkit` (wkhtmltopdf) | Requires external binary — not available on Anvil |
| `fpdf2` | Simpler than reportlab but less mature; fewer layout options |
| HTML-to-PDF services | External dependency; adds latency and cost |

### Invoice Template Design

The invoice PDF is generated from a server function that takes invoice data and returns a `Media` object:

```python
from reportlab.lib.pagesizes import A4
from reportlab.lib.units import mm
from reportlab.pdfgen import canvas
from io import BytesIO
import anvil.media

@anvil.server.callable
def generate_invoice_pdf(invoice_id):
    invoice = app_tables.invoices.get_by_id(invoice_id)
    business = app_tables.business_profile.get()
    contact = app_tables.contacts.get_by_id(invoice.contact_id)
    
    buffer = BytesIO()
    c = canvas.Canvas(buffer, pagesize=A4)
    
    # Header: business logo, name, address
    # Invoice details: number, date, due date
    # Line items: service, quantity, unit price, total
    # Payment terms
    # Footer: thank you note, contact info
    
    c.save()
    buffer.seek(0)
    
    filename = f"invoice_{invoice.invoice_number}.pdf"
    return anvil.media.from_file(buffer, media_type='application/pdf', name=filename)
```

### Storage and Delivery

| Step | Mechanism |
|------|-----------|
| Generation | Server function returns `Media` object |
| Storage | Stored in `invoices` table as `BlobMedia` column |
| Email delivery | Sent via Brevo SMTP as email attachment |
| Admin download | Downloaded from admin panel via server call |

**Note:** `BlobMedia` has row size limits (`anvil-platform-constraints` ADR). For invoices with many line items, consider storing the PDF externally (S3 or similar) and keeping only the URL in the Data Table. This is a V2 consideration if invoice complexity grows.

---

## Consequences

- ✅ Professional PDF invoices generated entirely server-side
- ✅ No external service dependency for PDF generation
- ✅ Full control over invoice layout and branding
- ⚠️ `reportlab` must be verified as installable on Anvil's server environment
- ⚠️ PDF generation adds server load — batch generation recommended for bulk invoices
- ⚠️ `BlobMedia` storage has limits — monitor invoice file sizes

---

## Related Documents

| Document | Relationship |
|---|---|
| ``adr/adr-global/anvil-platform-constraints.md`` | Server-side only constraint; BlobMedia limits |
| `integration-architecture.md` | Section 1: Payment Gateway Architecture (invoice trigger) |
| `docs/user-flows.md` | §3: Admin Daily Operations (invoice viewing) |
| `docs/platform-overview.md` | Section 5.5: Payments & Invoicing |

---

*End of `pdf-invoice-generation` ADR*
