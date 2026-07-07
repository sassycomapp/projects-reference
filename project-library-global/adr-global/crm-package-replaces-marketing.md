# 06 — Package Name: crm/ Replaces marketing/
Date: 2026-03-18
Status: Accepted
Source: cs-architectural-specification-D.md §13

---

## Context

During development the client-side package containing CRM and marketing forms was
named `marketing/`. As the platform evolved, it became clear that CRM and marketing
are tightly integrated in the C&S vertical — they are not separate concerns. The
package name `marketing/` no longer accurately described its scope. The live repository
confirmed the package had already been renamed.

---

## Decision

**The client-side package for CRM and marketing forms is `client_code/crm/`.**

`marketing/` is a stale name. Any documentation, prompt, or rules file that references
a `marketing/` client package is incorrect and must be updated to `crm/`.

The name reflects the integrated nature of CRM and marketing in the C&S vertical.

### Affected navigation items

The admin navigation group is **"Customers & Marketing"** (display label) but the
underlying package is `crm/`. The display label and the package name are intentionally
different — the label is user-facing, the package name is code-facing.

### server_marketing/ is unaffected

The server-side `server_marketing/` package name is unchanged. Only the client-side
package was renamed. Do not conflate the two.

---

## Consequences

### Rules files affected

| File | Required content |
|---|---|
| `spec_architecture.md` | Package structure — `client_code/crm/` confirmed name |
| `spec_crm.md` | Package reference — `client_code/crm/` |
| `platform_overview.md` | Any package structure references |
| Any file referencing `client_code/marketing/` | Correct to `client_code/crm/` |

---

*End of `crm-package-replaces-marketing` ADR*
