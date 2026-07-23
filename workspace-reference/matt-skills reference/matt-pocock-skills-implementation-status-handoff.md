# Matt Pocock Skills — Implementation Status Report
**Handoff document. Prepared for a separate technical conversation.**
**Status as of this report: 4 OpenCode instructions reported executed successfully by the
developer. Execution output itself was not reviewed by the assistant that wrote this report —
treat "implemented" items below as developer-reported, not independently verified.**

---

## 1. System context

- **PDLF** (Project/Program Design Lifecycle Framework): a stage-gated, ledger-backed
  orchestration system running on **OpenCode**. Not Claude Code — no Claude Code tooling,
  plugins, or `CLAUDE.md` conventions apply anywhere in this system.
- Three capability layers: GStack (workflow), Matt Pocock skills (engineering discipline),
  PostgreSQL (`pdlf` database, single source of truth ledger), GBrain (persistent memory).
- Repo topology per project: `<pdlf-home>` (`dev-pdlf` in dev, `C:\pdlf\` in production) is
  the framework itself. Each managed project has two repos: `{slug}` (code, Anvil-connected)
  and `{slug}-project-library` (docs). Confirmed example: `mb5pdlf` (code) +
  `mb5pdlf-project-library` (docs), both under `C:\dev\dev-mb5pdlf\`.

## 2. Matt Pocock skills — what's in use

Source: `mattpocock/skills`, v1.1.0, MIT licensed. Installed via the `skills.sh` copy method
(not the Claude Code plugin route — confirmed inapplicable since this system doesn't use
Claude Code). Files are plain, editable copies inside the relevant repos, not a live-synced
bundle — upstream updates do not propagate automatically.

**Five skills in scope, all others deliberately excluded:**

| Skill | Type | Pipeline position | Role |
|---|---|---|---|
| `setup-matt-pocock-skills` | User-invoked, config generator | Run once per required repo before any other engineering skill | Writes `docs/agents/issue-tracker.md`, `docs/agents/domain.md`, `docs/agents/triage-labels.md` (if `triage` present), plus a "Matt Pocock skills artifact governance" section in that repo's `AGENTS.md` |
| `domain-modeling` | Model-invoked, fat/self-contained | Step 16, gate `domain-locked` | Builds/stress-tests domain glossary and ADRs |
| `to-tickets` | User-invoked, fat/self-contained | Step 19 | Breaks locked plan into tracer-bullet vertical-slice tickets |
| `triage` | User-invoked, fat | Step 39, gate `build-verified` | 5-role classification (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`) of findings from Steps 36–38 |
| `improve-codebase-architecture` | User-invoked, fat, optional | Step 40 | Read-only architecture health scan, HTML report |

**Explicitly excluded, with reasoning:**
- `to-spec`/`to-prd`, `prototype` — removed per PDLF's own "skill earns its place only if it
  does something OpenCode can't" principle.
- `grilling` (and wrappers `grill-me`, `grill-with-docs`) — considered and rejected. `triage`
  and `improve-codebase-architecture` both have internal "grill if needed" branches, but PDLF's
  existing deviation-request mechanism (a step that can't complete presents three options to
  the developer, never decides unilaterally) already covers the same need. Resolution: those
  branches are routed to the deviation-request flow, not to any Matt Pocock skill.
- Full framework alternatives (BMAD-METHOD, GitHub Spec Kit, OpenSpec, GSD, Superpowers)
  evaluated and rejected — they are complete orchestration systems that would compete with
  PDLF's own orchestrator, not narrow skills that slot inside it. Decision: keep current
  5-skill selection, no framework switch.

## 3. Dependency audit — explicit verdict

Every skill in §2 was individually checked for whether it's a thin wrapper requiring a
companion skill to also be present (Matt Pocock's repo mixes both types — some skills are a
few lines that just delegate to another named skill; others are fully self-contained).

**Result: 4 of the 5 deployed skills (`setup-matt-pocock-skills`, `domain-modeling`,
`to-tickets`, `improve-codebase-architecture`'s core logic) are self-contained — no companion
skill required, nothing to co-distribute.**

**The one dependency that was found** — `triage` and `improve-codebase-architecture` both
have an internal branch that calls into a `grilling` interview skill — was deliberately **not**
resolved by importing `grilling` alongside them. It was resolved by routing that branch to
PDLF's own existing deviation-request mechanism instead (see §2, exclusions). So the correct
final state is: **no additional skill needs to be distributed alongside the 5 in use.** If a
future Claude or developer sees a "grill if needed" instruction inside `triage` or
`improve-codebase-architecture`'s own file text, that is expected and already accounted for —
it should not be read as a missing dependency.

## 4. Confirmed repo/path facts

| Fact | Value | Confirmed by |
|---|---|---|
| `<pdlf-home>` Matt Pocock config path | `docs/agents/` | Live `dev-pdlf/AGENTS.md` |
| `.scratch/` exists | `C:\dev\dev-pdlf\.scratch`, `C:\dev\dev-mb5pdlf\mb5pdlf-project-library\.scratch` | Direct filesystem check |
| `.scratch/` absent | `C:\dev\dev-mb5pdlf\mb5pdlf` (code repo) | Direct filesystem check |
| Repos requiring `setup-matt-pocock-skills` | `<pdlf-home>` + each `{slug}-project-library` | Derived from `.scratch/` locations above; code repo does not need it |
| `{slug}-project-library`'s own `docs/agents/` path | **Not independently confirmed** | Should not be assumed identical to `<pdlf-home>`'s — Layer 3 project instances use a different folder convention (`adr-local/` vs. `docs/adr/`) per `spec-architecture.md` |

## 5. Postgres ledger

- Connection: `localhost:5432`, database `pdlf`, role `pdlf_agent`, password in
  `C:\dev\dev-pdlf\.env` (`PGPASSWORD`).
- PostgreSQL runs on Windows; OpenCode runs in WSL. Direct `psql` from WSL does not work
  (binfmt does not pass env vars) — must bridge via
  `powershell.exe -Command '$env:PGPASSWORD="..."; & "C:\Program Files\PostgreSQL\15\bin\psql.exe" ...'`.
- Six original tables: `projects`, `steps`, `gates`, `deviations`, `bft_records`, `isc`.
- **`steps` table — full confirmed schema:**
  `id` (serial PK) · `project_slug` (FK → `projects.slug`) · `step_number` (text) ·
  `sort_order` (numeric) · `phase` · `step_type` (`skill`/`manual`/`optional`/`gate`) ·
  `skill_family` (`gstack`/`matt-skills`/`opencode`/`manual`/null) · `skill_name` ·
  `inputs_used` (jsonb) · `artifact_path` · `synthesized` (bool) ·
  `status` (`pending`/`in_progress`/`complete`/`blocked`/`skipped`) · `next_action` ·
  `started_at` · `completed_at` · `blockers` · `notes_for_next_session` · `gbrain_query` ·
  `gbrain_results`. Unique constraint: `(project_slug, step_number)`.
  `bft_records`, `deviations`, `gates` all reference `steps.id` via their own `step_id` column
  — this is the established FK pattern. A feature-scoped loop (Steps 35–39 repeat once per
  feature) does not need a separate scoping column: each feature's execution gets its own
  `steps` row, so `steps.id` alone identifies "this step, for this feature, this run."

## 6. Changes implemented (developer-reported successful; verify on entry to new conversation)

Four OpenCode instructions were run, in this order:

1. **Stale reference correction** — in `dev-pdlf/AGENTS.md`'s "Matt Pocock skills artifact
   governance" section: `to-prd, to-issues, triage → .scratch/` corrected to
   `to-tickets, triage → .scratch/`; `setup-matt-pocock-skills → docs/agents/ + CLAUDE.md`
   corrected to `... + AGENTS.md`. Repo-wide search for the same stale strings also run.
2. **Repo setup verification** — confirmed config presence in `<pdlf-home>` and
   `{slug}-project-library`; explicitly did not assume or force the code repo needs it.
3. **Grilling-branch routing** — `stepwise/step-39-triage.md` and
   `stepwise/step-40-improve-codebase-architecture.md` edited so any "grill if needed"
   condition routes to PDLF's deviation-request flow instead of any grilling-family skill.
4. **`triage_findings` ledger table** — new table created:
   ```sql
   CREATE TABLE triage_findings (
     id              SERIAL PRIMARY KEY,
     step_id         INTEGER NOT NULL REFERENCES steps(id),
     finding_id      TEXT NOT NULL,
     source_step     TEXT NOT NULL CHECK (source_step IN ('36', '37', '38')),
     triage_label    TEXT NOT NULL CHECK (triage_label IN
                       ('needs-triage','needs-info','ready-for-agent','ready-for-human','wontfix')),
     resolution_note TEXT,
     resolved_by     TEXT,
     resolved_at     TIMESTAMPTZ,
     created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
   );
   ```
   `stepwise/step-39-triage.md` updated to upsert `triage` skill output into this table
   (treating the `.scratch/` label file as transient input only, not persisted state — mirrors
   how Step 38A already consolidates CI results into the ledger). `build-verified` gate logic
   updated to require every `triage_findings` row for the current feature's `steps.id` to be
   `wontfix` or have a non-null `resolved_at`, joined through the `gates.step_id` FK.

## 7. Outstanding items (not yet done)

1. **Confirm `{slug}-project-library`'s actual `docs/agents/` path** — not yet independently
   verified against that repo's own `AGENTS.md`; do not assume it matches `<pdlf-home>`'s.
2. **MIT license credit** — a one-line attribution for `mattpocock/skills` has not yet been
   added anywhere in PDLF's own docs.
3. **No periodic-diff policy in place yet** — copied skill files will silently drift from
   `mattpocock/skills` upstream `main`; no cadence or process has been established to check.
4. **Verify the four executed changes above** against the actual repo state — this report
   relies on developer-reported success, not a reviewed diff/output.

---
*End of handoff report.*
