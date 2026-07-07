# Mybizz — Regulatory Compliance Baseline (Specification)

**Scope:** Mybizz-wide. This posture applies because Mybizz-the-company is based in South Africa
and operates a multi-tenant SaaS model — it does not depend on what any individual app does.
App-specific facts (e.g., whether a given app actually processes payments, and therefore whether
PCI DSS applies to it) are recorded in `spec-security-architecture.md`, which
should reference this baseline rather than restate it.

---

## POPIA (South Africa) — Mandatory, Every App

Mybizz is based in South Africa. Registration with the Information Regulator as a Responsible
Party is required after launch, for every Mybizz product. Data subject rights — access,
correction, deletion, objection — must be implemented in every app.

## GDPR (European Union) — Conditional, Per App

Applies to a given app if it serves EU customers. The same data-subject-rights implementation
required for POPIA (export and anonymisation functions) satisfies both regimes — no separate
implementation is needed per regulation, only a determination of whether EU customers are served.

## PCI DSS — Conditional, Per App

Applies to any app that processes payments. The standard Mybizz approach: store only payment
tokens, never raw card numbers — this places a compliant app in SAQ A, the lowest compliance
scope. An annual Self-Assessment Questionnaire is required for any app in scope.

## Standard Data Retention Periods (Mybizz-Wide Defaults)

| Data Type | Retention |
|---|---|
| Contacts | Indefinite, unless deletion requested |
| Transactions and invoices | 7 years (tax compliance) |
| Audit logs | 2 years |
| Email logs | 90 days |
| Completed tasks | Auto-deleted after 90 days |

A given app may need additional data categories beyond this list (recorded in that app's own
security document) but should not override these defaults without a stated reason.

## Required Legal Documents Before Any App Goes Live

Privacy Policy, Terms of Service, Cookie Policy — per app, since these are customer-facing and
must reflect that app's actual data practices, even though the underlying regulatory posture
above is shared.

---

*Regulatory Compliance Baseline v1.0. Extracted from `spec-security-architecture.md` as Mybizz-wide
content, confirmed applicable to every Mybizz product.*
