# `free-trial-abandoned` ADR — 30-Day Free Trial Abandoned

**Status:** Confirmed  
**Date:** 2026-06-13  
**Authority:** Derived from platform-overview.md, pricing model, Anvil free tier analysis

---

## Context

The original platform design included a 30-day free trial based on Anvil's free tier. The intent was to allow prospective clients to explore the platform before committing to a subscription — modelled loosely on Windows Update-style onboarding.

On examination, this approach has significant problems:

1. **The product requires full configuration before it is useful.** Vault setup, payment gateway credentials, Brevo SMTP, business profile, and service configuration must all be completed before the platform does anything meaningful for a client. A partially configured instance on a restricted free tier is not a useful product demonstration.

2. **Anvil's free tier imposes hard constraints** that would make the trial experience actively misleading:
   - Background tasks restricted to 30-second timeout or unavailable entirely
   - Scheduled tasks unavailable
   - No persistent server
   - Appointment reminders, campaign enrollment, and other core features would not function

3. **The "look around" intent is better served by a live demo** of a fully configured mybizz instance, where all features work correctly and the client sees the real product.

4. **A 30-day money-back guarantee post-signup** provides stronger confidence than a crippled free trial — it commits the client to a configured, working instance, and gives them a full refund path if not satisfied.

---

## Decision

The 30-day free trial is abandoned. It will not be implemented.

**Replaced by:**

| Original | Replacement |
|---|---|
| 30-day free trial on Anvil free tier | Live guided demo on mybizz's own fully configured instance |
| Self-serve trial access | 30-minute guided onboarding session post-signup |
| "Look around" limited functionality | 30-day money-back guarantee on first paid subscription |

All paying clients receive a properly provisioned instance on the Anvil Business plan from day one. No client instance ever runs on the Anvil free tier.

---

## Consequences

- ✅ All client instances properly hosted with full feature access from day one
- ✅ Background tasks, scheduled tasks, and persistent server available to all clients immediately
- ✅ No Anvil free tier constraints affect any client's experience
- ✅ Demo environment (mybizz's own instance) showcases real product, not a restricted version
- ✅ Simplifies provisioning — one process for all clients, no trial-to-paid transition
- ⚠️ Requires a compelling, well-configured demo instance to be maintained by mybizz
- ⚠️ Money-back guarantee terms must be defined and communicated clearly at signup

---

## Related Documents

| Document | Relationship |
|---|---|
| `platform-overview.md` | Pricing and onboarding model |
| `implementation/client-activation-runbook.md` | Provisioning process applies to all clients from day one |

---

*End of `free-trial-abandoned` ADR*
