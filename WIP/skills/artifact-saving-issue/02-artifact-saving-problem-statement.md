# Problem Statement: GStack Artifact Saving Inconsistency

**Project:** project-library (`/mnt/c/mybizz/project-library`)
**Date opened:** 2026-06-25
**Status:** Under active investigation — root cause partially identified, not yet fixed
**Severity:** Moderate — no data loss observed, but artifact discovery between
chained skills is unreliable and undocumented behavior is widespread.

---

## 1. Summary

GStack skills are supposed to save their output artifacts to a predictable
default location (`~/.gstack/projects/$SLUG/...`) so that later skills can
automatically find and reference earlier skills' outputs. In practice, the
actual save location varies between skill runs of the *same project*, the
documentation describing where things save is unreliable, and at least one
skill failed to save anywhere at all under normal use. This document records
every symptom, test, and anomaly found during investigation, so the
investigation does not need to be repeated or re-derived from memory.

---

## 2. Environment / Path Reference

| Item | Path |
|---|---|
| Project repo (WSL) | `/mnt/c/mybizz/project-library` |
| Project repo (Windows) | `C:\mybizz\project-library` |
| Preferred output folder | `/mnt/c/mybizz/project-library/gstack-outputs/` |
| AGENTS.md | `/mnt/c/mybizz/project-library/AGENTS.md` |
| AGENTS.md backup | `/mnt/c/mybizz/project-library/AGENTS.md.backup-20260625` |
| CLAUDE.md | `/mnt/c/mybizz/project-library/CLAUDE.md` |
| GStack install root | `/mnt/c/mybizz/gstack/` |
| GStack bin scripts | `/mnt/c/mybizz/gstack/bin/` |
| **Live/canonical skill source** | `/mnt/c/mybizz/skills/` |
| Skills symlink (what OpenCode actually reads) | `~/.config/opencode/skills/` → symlinked to the line above |
| GStack home | `~/.gstack/` (i.e. `/home/dev-p/.gstack/`) |
| GStack config file | `~/.gstack/config.yaml` |
| `gstack-config` binary | `/mnt/c/mybizz/gstack/bin/gstack-config` |
| `gstack-slug` script | `/mnt/c/mybizz/gstack/bin/gstack-slug` |
| Slug cache directory | `~/.gstack/slug-cache/` |
| Skill audit/spec output folder | `C:\mybizz\project-library\workspace\system-operation-reference\skills\{skill-name}-spec.md` |

**Note on skill source duplication:** `/mnt/c/mybizz/gstack/` also contains
nine duplicate copies of every skill, one per supported AI agent
(`.agents`, `.cursor`, `.factory`, `.gbrain`, `.hermes`, `.kiro`, `.openclaw`,
`.opencode`, `.slate`). These are **not** the live source — they appear to be
build/dev artifacts. The live source confirmed by the user's own
`gstack-reference.md` is `/mnt/c/mybizz/skills/`. This was a significant
false trail during investigation and should not be revisited.

**Note on SKILL.md generation:** Every skill folder contains both `SKILL.md`
and `SKILL.md.tmpl`. Identical-second file timestamps and an explicit
in-file comment (`<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit
directly --> <!-- Regenerate: bun run gen:skill-docs -->`) confirm `SKILL.md`
is generated from `SKILL.md.tmpl` plus shared resolver logic at
`/mnt/c/mybizz/skills/scripts/resolvers/*.ts`. Direct edits to `SKILL.md`
or to gstack's own source were explicitly ruled out as a fix — not our
codebase to rewrite, and edits would be overwritten on regeneration.

---

## 3. Symptoms Discovered

1. **Inconsistent project slug (`$SLUG`).** The same project resolved to at
   least three different slug values depending on invocation context:
   - `dev-p` — when `gstack-slug` was run manually from the home directory
   - `sassycomapp-project-library` — when run manually from inside the
     project directory in a WSL terminal
   - `project-library` — what the live agent actually used when writing a
     `/context-save` checkpoint
   - `unknown` — seen as a fallback literal for `/health`'s save (not
     produced by `gstack-slug` itself; indicates `$SLUG` was unset/empty
     in that execution context)

2. **A skill that asked for save confirmation, then saved nothing when
   declined.** Running `/plan-ceo-review` produced the on-screen question
   *"Shall I save this review to gstack-outputs/ as a formal record?"*
   Answering "no" produced: *"Understood. The CEO review is complete and the
   decisions are recorded in this conversation. No file saved."* No artifact
   was found in either `~/.gstack/projects/.../ceo-plans/` or
   `gstack-outputs/` afterward.

3. **A skill that saved to a location matching none of four documented
   claims.** Running `/retro` produced the on-screen confirmation: *"Retro
   history saved to `memory/retro-2026-06-26.json`."* Verified actual path:
   `C:\mybizz\project-library\memory\retro-2026-06-26.json`. This
   matches **none** of:
   - the user-facing reference doc's claim: `~/.gstack/projects/$SLUG/retros/`
   - a full-repo automated audit's claim: `~/.gstack/projects/$SLUG/retros/*.md`
   - the actual literal text found by direct grep of `/retro`'s own
     `SKILL.md` (Step 13, "Save Retro History"): `.context/retros/`
   - `/retro`'s separate "global" mode path: `~/.gstack/retros/global-*.json`

4. **A skill's own documentation disagreeing with itself.** An automated
   full-repo audit (`default-save-paths.md`) claimed one save path for
   `/retro`; a manual line-by-line grep of the exact same `SKILL.md` file,
   minutes earlier in the same session, found a different path. Both claimed
   to be reading the same file directly.

---

## 4. Concern: Skill-to-Skill Artifact Discovery

Many GStack skills are designed to automatically discover and read each
other's prior outputs via hardcoded shell lookups against
`~/.gstack/projects/$SLUG/...`, for example:

- `/qa` searches for `/plan-eng-review`'s test plan via
  `ls -t ~/.gstack/projects/$SLUG/*-test-plan-*.md`
- `/ship` and `/autoplan` search for design docs via
  `ls -t ~/.gstack/projects/$SLUG/*-design-*.md`
- `/autoplan`'s Phase 4 aggregates task JSONL files written by four separate
  review skills, all keyed to the same `$SLUG`

Because `$SLUG` is not stable across invocation contexts (see Symptom 1), a
skill writing under one slug value and a later skill reading under a
different slug value for the *same logical project* will produce a silent
miss — no error, just "no prior artifact found." This was the original
concern that prompted the entire investigation, and remains only partially
resolved (see §7, Open Questions).

---

## 5. Tests Performed and Outcomes

| # | Skill run | Custom instruction given? | On-screen claim | Actual file found | Path |
|---|---|---|---|---|---|
| 1 | `/health` | None | Diagnostic dashboard only, no save mentioned | Yes, after locating correct slug folder | `~/.gstack/projects/unknown/health-history.jsonl` |
| 2 | `/context-save` | None | "Context saved... File: ..." | Yes | `~/.gstack/projects/project-library/checkpoints/20260625-220726-5-app-system-gbrain-query.md` |
| 3 | `/plan-ceo-review` (after removing the `gstack-outputs` bullet from AGENTS.md) | None | Asked "Shall I save... to gstack-outputs/?" | **No — declined, nothing saved anywhere** | n/a |
| 4 | `/retro` (after AGENTS.md updated with new dual-save rule) | None | "Retro history saved to `memory/retro-2026-06-26.json`" | Yes | `C:\mybizz\project-library\memory\retro-2026-06-26.json` |

**Diagnostic command used to check the default location** (must `cd` into the
project first — running from home directory silently produces a wrong slug):

```bash
cd /mnt/c/mybizz/project-library
eval "$(/mnt/c/mybizz/gstack/bin/gstack-slug 2>/dev/null)"
echo "SLUG: $SLUG"
ls -la ~/.gstack/projects/"$SLUG"/ 2>/dev/null
```

**Note:** `eval "$(...)"` is required — capturing `gstack-slug`'s output as a
plain string (`SLUG=$(gstack-slug)`) does not work, because the script emits
shell assignment lines (`SLUG=...`, `BRANCH=...`) meant to be evaluated, not
a bare value.

---

## 6. Measures Taken So Far

1. **Backed up `AGENTS.md`** to `AGENTS.md.backup-20260625` before any edits.

2. **Removed one bullet** from `AGENTS.md`'s `## GStack artifact governance`
   section: the line directing office-hours/CEO/ENG/planning/task-list/
   handoff-note reports to save to `gstack-outputs/`. This was done to test
   whether removing it would expose the skill's true default behavior.
   Result: it changed behavior from "silently save to gstack-outputs" to
   "ask whether to save to gstack-outputs" — confirming the removed bullet
   had been the active instruction, but also exposing Symptom 2 above.

3. **Added a new instruction** to `AGENTS.md`'s `## GStack artifact
   governance` section (current live text):

   ```
   - For ANY GStack or OpenCode skill that produces a report, plan, review, or artifact
     during a session → ALWAYS complete the skill's own native default save (to its
     standard ~/.gstack/projects/ location) without asking for confirmation. ADDITIONALLY
     save a copy to `gstack-outputs/` for human reference. Never skip the native save,
     and never ask whether to save — always save both.

     After saving, the on-screen response must confirm both save locations explicitly,
     e.g.: "Saved to: ~/.gstack/projects/<slug>/<path> AND gstack-outputs/<filename>."
     If either save fails, state which one failed and why — do not silently report
     success for only the save that worked.
   ```

   **Not yet verified** whether this instruction reliably prevents the
   ask-then-skip behavior seen in Test 3, since that test was run *before*
   this instruction was added, and Test 4 (`/retro`, run after) saved to a
   location ( `memory/...`) this instruction doesn't even mention.

4. **Began per-skill targeted audits.** A full-repo blanket audit
   (`default-save-paths.md`, all 59 skills in one pass) was attempted first,
   but found to contain at least one factual error when cross-checked
   manually (the `/retro` path — see Symptom 4). Switched to one-skill-at-
   a-time targeted audits using a fixed 12-question template, saved to
   `workspace/system-operation-reference/skills/{skill-name}-spec.md`. Only
   one skill (`plan-ceo-review`) has been completed under this method so
   far.

---

## 7. Anomalies Catalogued

| # | Anomaly | Sources in conflict |
|---|---|---|
| 1 | `/retro`'s save path has four different claimed/observed values | reference doc, blanket audit, direct SKILL.md grep, actual runtime result |
| 2 | `/plan-ceo-review`'s targeted audit states no save-confirmation prompt exists in its `SKILL.md` text, yet one was directly observed on-screen | targeted spec (Q5, Q10) vs. direct observation (Test 3) |
| 3 | The user-facing `gstack-skills-reference.md` omits real disk writes confirmed present in the actual `SKILL.md` (CEO plan file, spec review metrics, stale plan archival, the entire spec-review-subagent section) | reference doc vs. targeted spec (Q10) |
| 4 | `$SLUG` resolves to ≥3 different values for one project depending on invocation context | see Symptom 1 |
| 5 | `gstack-config resolve-user-slug` and `gbrain doctor` both hang indefinitely when run directly in WSL | unresolved, not root-caused |
| 6 | Nine duplicate skill-source folders exist inside the gstack repo; only one (`/mnt/c/mybizz/skills/`) is actually live | confirmed via user's own `gstack-reference.md`; false trail consumed significant investigation time |
| 7 | No `gstack-config` key exists for output location at all | confirmed via `gstack-config list` / `gstack-config defaults` — no path-related setting among ~14 existing keys |

---

## 8. Leading Hypothesis on Root Cause (Unconfirmed)

`/mnt/c/mybizz/gstack/bin/gstack-slug` computes `$SLUG` in this order:

1. Check a cache file keyed by the **exact absolute working directory** the
   script was run from (`~/.gstack/slug-cache/<encoded-cwd>`)
2. If no cache, compute from `git remote get-url origin`, parsed to
   `org-repo` format
3. **Only if there is no git remote at all**, fall back to
   `basename "$PWD"`

This fully explains the three observed slug values (home dir → no remote →
username fallback; WSL terminal in project dir → remote found → full
`org-repo` slug; agent's own session → fell back to plain basename,
implying `git remote get-url origin` did **not** succeed in the agent's
execution context despite succeeding in the manual WSL terminal in the same
apparent directory).

**This points to the agent's shell execution environment differing from the
user's manual WSL terminal session** — possibly working directory, possibly
visibility of the git remote, possibly `$HOME` itself. This has not yet been
directly confirmed.

**Proposed diagnostic (not yet run):** ask the agent directly, in a live
session, to run and report the raw output of:
```
pwd
git remote get-url origin
echo $HOME
```
and compare against the same three commands run manually in WSL.

---

## 9. Open Questions / Next Steps

1. Run the diagnostic in §8 to confirm or rule out the environment-mismatch
   hypothesis as the root cause of `$SLUG` instability.
2. Determine whether the new `AGENTS.md` dual-save rule (§6.3) actually
   prevents the ask-then-skip failure mode, by re-running a governed skill
   in a fresh session.
3. Determine whether the `/plan-ceo-review` "shall I save to gstack-outputs"
   prompt (Symptom 2) originated from conversational/chat context rather
   than from any file-based instruction, by testing in a session with no
   prior chat history mentioning `gstack-outputs`.
4. Continue per-skill targeted audits (§6.4) for the remaining ~58 skills,
   prioritizing skills actually in active use.
5. Investigate where Matt's-skills artifacts save to — not yet started.
6. Decide on the shape of the proposed "devlog" — a chronological index of
   completed tasks across gstack, gbrain, Matt's-skills, and direct
   OpenCode/developer tasks, pointing at wherever each artifact actually
   landed, rather than duplicating content — discussed but not yet built.
