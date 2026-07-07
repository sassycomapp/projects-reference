# ADR 42: Mandatory README in Every Client Instance Documenting the Five-App System

## Status

Accepted

## Context

Mybizz runs on a five-app architecture within a single Anvil account:

1. **`mb-3-cs`** — the development workspace. All feature work happens here. Never a dependency for any client instance.
2. **`master_template`** — the sole source of all server-side logic and UI forms for every client instance. Tracks the `stable` GitHub branch. Never given to clients directly; client instances depend on it.
3. **`blank_client_template`** — the provisioning clone source. Contains the 36-table schema, one startup module, and `master_template` as a dependency. Zero server modules, zero forms.
4. **`[client-name]-cs`** — one per paying client. Contains the schema, the startup module, and an active dependency on `master_template`. Nothing else.
5. **`Mybizz_management`** — internal platform-operator tooling (deferred, post-launch).

Each client instance therefore appears, on inspection, almost entirely empty: no forms, no server modules — only a startup module and a dependency reference. All server-side logic and UI a client instance displays is inherited live from `master_template`. There is no copy to fall out of date and no "deploy" step; the instance simply reflects whatever `master_template` currently contains.

This emptiness is intentional, but Anvil's dependency-precedence behavior means that if a server module or form is ever added directly to a client instance app, Anvil will execute or render that local copy instead of the inherited one. This permanently and silently breaks update propagation for that single client instance — the client stops receiving any future change to `master_template` with no error, warning, or other obvious signal that anything has gone wrong.

A human developer with architectural context, on opening a client instance and finding it conspicuously empty, will typically pause and investigate before adding code — the absence of any forms or modules functions as an implicit stop sign. However, this is a learned instinct, not a guarantee. It does not reliably transfer to:

- An AI coding agent given a task scoped to "fix something for client X," which may treat an empty directory as an invitation to create the missing file rather than as a structural signal to investigate further.
- A new team member, contractor, or maintainer who has not yet internalized the five-app model.
- Anyone working under time pressure who skips the investigation step.

No client instance currently contains any documentation that makes this constraint explicit at the point where a developer or agent would actually encounter it.

## Decision

Every client instance app (App 4) must contain a standardized README, present from the moment of provisioning, that:

1. **Leads with the constraint, not the explanation.** The first visible line must state plainly that the directory is intentionally empty except for the startup module, and that no forms or server modules are to be added here.
2. **Explains the mechanism.** A brief explanation that this app depends on `master_template` for all server logic and UI, and that adding a form or module here does not extend that behavior — it silently and permanently overrides the dependency for this instance only.
3. **Redirects to the correct location.** A pointer to where the requested change actually belongs: the `mb-3-cs` development workspace, tested against its own private client instance, then merged `develop` → `stable` into `master_template`, after which every client instance — including this one — reflects the change automatically.

This README is added to `blank_client_template` (App 3), so that every newly provisioned client instance inherits it automatically at clone time with no manual step required.

## Consequences

**Positive**

- Removes reliance on tacit human intuition as the sole safeguard against an action that is easy to take, hard to notice, and effectively irreversible without manual remediation.
- Gives AI coding agents and unfamiliar maintainers the necessary context inline, at the exact point where the risk exists, rather than in a separate architecture document they may never consult before acting.
- Converts a non-obvious platform-specific trap into an explicit, documented constraint.

**Negative / Trade-offs**

- This is a procedural control, not a technical one. It reduces the likelihood of the mistake but does not make it impossible — nothing in Anvil itself prevents a form or module from being added to a client instance.
- Depends on the README being read before action is taken. It does not help if a developer or agent edits the instance without first opening or being given the README's contents.
- Requires the README to be kept current if the five-app structure changes; since it is propagated only at provisioning time via `blank_client_template`, any future revision will not retroactively reach already-provisioned client instances without a separate update pass.

## Alternatives Considered

- **Rely on developer training and tacit convention alone.** Rejected — does not transfer to AI agents, new hires, or anyone acting under time pressure, and was the status quo that prompted this ADR.
- **Document the rule only in a central/external architecture document.** Rejected — context not present at the point of action is context that can be skipped; the constraint needs to live where the risk is, not only where the explanation is.
- **Build a platform-level technical guard** (e.g., a CI or audit step that flags any client instance containing local forms or server modules). Not adopted at this time due to current tooling constraints, but noted as a stronger, detective-control alternative worth revisiting in a future ADR if provisioning scale or team size increases the practical risk.
