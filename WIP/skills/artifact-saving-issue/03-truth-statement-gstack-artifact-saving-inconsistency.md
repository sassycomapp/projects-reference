# Truth Statement: GStack Artifact Saving Inconsistency

**Date:** 2026-06-26
**Method:** Direct fetch of official upstream GStack documentation
(`github.com/garrytan/gstack`), read in full, compared line-by-line against
findings in `artifact-saving-problem-statement.md`. No inference, no
secondary sources. Every claim below states its exact source.

**Scope discipline:** Only GStack's own official docs were consulted. Local
files (`gstack-skills-reference.md`, the blanket audit, the targeted
`plan-ceo-review` spec) are referenced only as *prior claims being checked*,
not as sources of new fact.

---

## Sources Consulted

| # | Source | URL |
|---|---|---|
| 1 | Skill Deep Dives | `https://github.com/garrytan/gstack/blob/main/docs/skills.md` |
| 2 | Repository README | `https://github.com/garrytan/gstack` |
| 3 | Canonical root `SKILL.md` (the `/browse` skill — contains the master preamble template shared by every other skill) | `https://github.com/garrytan/gstack/blob/main/SKILL.md` |

---

## 1. Confirmed Findings

### 1.1 — `/retro`'s save path is `.context/retros/`, not `~/.gstack/projects/$SLUG/retros/`

**Source 1** states plainly that `/retro` "saves a JSON snapshot to
`.context/retros/`."

This **confirms** the direct manual grep performed earlier on the local
`/retro/SKILL.md` (Step 13: "Save Retro History"), which also found
`.context/retros/`.

This **contradicts**, on official authority:
- The local `gstack-skills-reference.md`, which claimed
  `~/.gstack/projects/$SLUG/retros/`
- The automated blanket audit (`default-save-paths.md`), which claimed
  `~/.gstack/projects/$SLUG/retros/*.md`

**Verdict: the direct-grep finding was correct. Both other local documents
were wrong about this specific skill.** This closes Open Question #1 from
the problem statement for the project-scoped retro mode (the *global* mode,
`~/.gstack/retros/global-*.json`, is unaffected and was never in dispute).

### 1.2 — Chaining via `~/.gstack/projects/` is the documented, intended design

**Source 1** confirms, for multiple skills:
- `/office-hours`: design doc "written to `~/.gstack/projects/`" — "feeds
  directly into `/plan-ceo-review` and `/plan-eng-review`"
- `/plan-ceo-review`: visions/decisions "persisted to `~/.gstack/projects/`"
- `/plan-eng-review`: "writes a test plan artifact to `~/.gstack/projects/`.
  When you later run `/qa`, it picks up that test plan automatically"
- `/design-shotgun`: approved variant "saves to
  `~/.gstack/projects/$SLUG/designs/`"
- `/learn`: learnings "accumulate in `~/.gstack/projects/$SLUG/learnings.jsonl`"
  — and explicitly: "other skills... automatically search learnings before
  making recommendations"

**This confirms** the cross-skill-discovery concern raised in §4 of the
problem statement is not a misunderstanding — it is exactly how GStack is
designed to work, end to end, by its own author's account. The chaining
mechanism is real and central to the tool's value proposition, which makes
`$SLUG` instability a design-critical problem, not a cosmetic one.

### 1.3 — Multiple skills save to a *repo-relative* `.gstack/` path, with no tilde — distinct from `~/.gstack/`

**Source 1** states:
- `/design-review` (live audit): "Report with before/after screenshots
  saved to `.gstack/design-reports/`"
- `/qa`: "Full report with screenshots saved to `.gstack/qa-reports/`"

This is a **previously undocumented pattern** in the problem statement: a
third path family, distinct from both `~/.gstack/projects/$SLUG/...`
(home-relative) and `.context/...` (repo-relative, different folder name).
`.gstack/` (no tilde, repo-relative) and `~/.gstack/` (home-relative) are
visually near-identical in skill text and in casual conversation — a single
missing tilde silently redirects a save from the project repo to the user's
home directory, or vice versa. This is a concrete, confirmed naming-collision
risk worth flagging on its own.

### 1.4 — `~/.gstack/` is officially the single unambiguous term for "global state"

**Source 2**'s uninstall instructions describe `~/.gstack/` plainly as
"global state" and direct users to `rm -rf ~/.gstack` to remove it
entirely. This confirms the directory's intended scope and role, consistent
with how it has been used throughout this investigation.

---

## 2. Critical New Finding — Likely Root Cause of `$SLUG` Instability

**Source 3** (the canonical root `SKILL.md`) contains the master preamble
template that every other skill's `SKILL.md` is generated from. Reading it
directly reveals:

```
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" 2>/dev/null || true
```

This line — present in this *exact* literal form — hardcodes a path that
only exists for a **Claude Code** install (`~/.claude/skills/gstack/`).
**Source 2**'s own host table confirms that an OpenCode install places
GStack's binaries at a *different* location entirely:

| Agent | Skills install to |
|---|---|
| OpenCode | `~/.config/opencode/skills/gstack-*/` |
| Claude Code (implicit default) | `~/.claude/skills/gstack/` |

This user's environment is OpenCode-based — confirmed by the user's own
`gstack-reference.md` and by the live symlink at
`~/.config/opencode/skills/` → `/mnt/c/mybizz/skills/`. **No Claude
Code install path has been confirmed to exist on this machine.**

**This exact hardcoded literal path was independently confirmed present, by
direct inspection, in this project's own live skill files** — not merely
in the official upstream copy:

- `/mnt/c/mybizz/skills/autoplan/SKILL.md`, confirmed via a direct
  `diff` against `SKILL.md.tmpl` (problem-statement investigation), shows
  the literal line:
  `eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" 2>/dev/null || true`
- `/mnt/c/mybizz/skills/plan-ceo-review/SKILL.md`, line 1243, per the
  targeted skill-spec audit, shows:
  `eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG/ceo-plans`

**If `~/.claude/skills/gstack/bin/gstack-slug` does not exist on this
machine, every one of these calls fails silently** — note the trailing
`2>/dev/null` (suppresses the error message) and `|| true` (suppresses the
non-zero exit code). No diagnostic of any kind reaches the user or the
agent. `$SLUG` would simply remain unset for the remainder of that skill
run, and any later code using `${SLUG:-unknown}` would fall through to the
literal string `unknown` — **the exact behavior observed when `/health` was
tested**, and a plausible explanation for the inconsistent slugs observed
across other skill runs as well.

**This is the strongest lead produced by this entire investigation.** It is
stated here as a well-evidenced hypothesis, not yet a proven fact, pending
the direct check below.

**Recommended next diagnostic (not yet run):**
```bash
ls -la ~/.claude/skills/gstack/bin/gstack-slug 2>&1
```
If this returns "No such file or directory," the hypothesis is confirmed:
this project's live skill files were never correctly adapted for the
OpenCode host, and still carry Claude-Code-only paths inherited unmodified
from the upstream template.

---

## 3. Other New Findings From Official Docs

### 3.1 — The official preamble itself defines the `unknown` fallback

Confirmed directly in **Source 3**:
```
_LEARN_FILE="${GSTACK_HOME:-$HOME/.gstack}/projects/${SLUG:-unknown}/learnings.jsonl"
```
The literal string `unknown` as a fallback for an empty `$SLUG` is not a
bug introduced anywhere downstream — it is baked into GStack's own official,
canonical template. This confirms Symptom 1 in the problem statement
(the `unknown` folder seen for `/health`) traces directly to this exact
official line, not to anything specific to this project's configuration.

### 3.2 — Two different environment-variable names appear to control the same thing

- **Source 3** uses `GSTACK_HOME` throughout the canonical preamble as the
  override for the base state directory (defaulting to `$HOME/.gstack`).
- **Source 2**, in the `/spec` skill's one-line description, instead
  references `$GSTACK_STATE_ROOT` for the same kind of purpose ("archive to
  `$GSTACK_STATE_ROOT/projects/$SLUG/specs/`").

These were not cross-checked against each other within the official docs
themselves, and no source explains the relationship between the two names.
This may be a documentation inconsistency on GStack's own side, a newer
skill using a renamed variable, or two genuinely separate mechanisms. Not
resolved by this fact-finding pass — flagged for direct testing
(`echo $GSTACK_HOME; echo $GSTACK_STATE_ROOT` in the actual agent
environment) rather than further documentation review.

### 3.3 — The nine duplicate per-host skill folders are official and intentional, not a build artifact

The problem statement (§2, "Note on skill source duplication") had
characterized the nine duplicate skill-source folders found inside
`/mnt/c/mybizz/gstack/` (`.cursor`, `.factory`, `.gbrain`, `.hermes`,
`.kiro`, `.openclaw`, `.opencode`, `.slate`, plus `.agents`) as "likely
build/dev artifacts" and a "false trail."

**Source 2** corrects this directly — its own host-support table lists
exactly these per-host install targets as the official, by-design
multi-agent installation mechanism (the same table quoted in §2 above).
**This characterization should be corrected**: these folders are real and
intentional; they were simply not the live source for *this* project's
OpenCode-driven setup, which instead reads from a separately-maintained
`/mnt/c/mybizz/skills/` folder whose relationship to GStack's own
official per-host generation process is not yet established (see §4 below).

### 3.4 — `gstack-analytics` exists as a standalone diagnostic binary, untried so far

**Source 2** documents a `gstack-analytics` command ("see your personal
usage dashboard from the local JSONL file"). This has not been run in this
investigation and may offer an independent, lower-risk way to confirm
whether skills are completing and logging correctly, without needing to
invoke a full skill each time.

### 3.5 — `/ship` has its own independent "ask once, cache forever" pattern — structurally similar to, but distinct from, the save-confirmation issue

**Source 1** describes the Review Readiness Dashboard gate: if the Eng
Review is missing, `/ship` "asks — but won't block you. Decisions are
saved per-branch so you're never re-asked." This is a different feature
(a review-completion gate, not an artifact save), but it establishes that
GStack has a known, designed pattern of asking a yes/no question once and
caching the answer per-branch. This is *not* confirmed to be the mechanism
behind the `/plan-ceo-review` "shall I save to gstack-outputs" prompt
(Symptom 2 in the problem statement) — that prompt's origin remains
unexplained by anything found in the official docs (see §4).

---

## 4. Still Unresolved — Not Addressed by Official Docs

1. **The exact origin of `/plan-ceo-review`'s "Shall I save this review to
   gstack-outputs/?" prompt.** Official docs make no mention of any GStack
   skill ever asking about `gstack-outputs/` — that folder name is entirely
   local to this project (`AGENTS.md`), never an upstream concept. The
   prompt's likely origin (conversational/chat-context influence rather
   than a file-based instruction) remains unconfirmed either way.

2. **Whether `/mnt/c/mybizz/skills/` was ever generated through
   GStack's official `./setup --host opencode` process**, or assembled by
   some other means. If it was generated through the official process, the
   hardcoded Claude-only paths found in §2 above would represent a genuine
   upstream bug worth reporting. If it was assembled differently, the
   explanation may sit entirely within this project's own setup history,
   not GStack's codebase.

3. **The relationship between `GSTACK_HOME` and `GSTACK_STATE_ROOT`** (§3.2).

None of these can be resolved by further documentation reading — each
requires a direct check against this machine's actual files or environment.
