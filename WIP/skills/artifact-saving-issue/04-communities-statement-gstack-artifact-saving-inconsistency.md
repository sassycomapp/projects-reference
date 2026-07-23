# Communities Statement: GStack Artifact Saving Inconsistency

**Date:** 2026-06-26
**Method:** Searched and read GitHub Issues on `garrytan/gstack`, GStack's own
`TODOS.md`, Hacker News threads, and third-party community write-ups/blogs
covering GStack. Every claim below states exactly where it came from.

**Note on the supplied Reddit link
(`reddit.com/r/ClaudeAI/comments/1s7jdof/...`):** direct fetch of this URL
returned `SITE_BLOCKED` (Reddit blocks automated fetches). Broader searches
for that thread's content, and for Reddit discussion of GStack generally,
surfaced commentary about Garry Tan's productivity/LOC claims (the
"5M+ lines of code" debate) — not the artifact-saving/`$SLUG` issue this
investigation is about. **No Reddit-specific discussion of this exact
technical issue was found.** This is stated plainly rather than papered
over — see §5.

---

## Sources Consulted

| # | Source | Type | URL |
|---|---|---|---|
| 1 | Issue #3 | GitHub Issue | `github.com/garrytan/gstack/issues/3` |
| 2 | Issue #648 | GitHub Issue | `github.com/garrytan/gstack/issues/648` |
| 3 | Issue #753 | GitHub Issue | `github.com/garrytan/gstack/issues/753` |
| 4 | Issue #1131 | GitHub Issue | `github.com/garrytan/gstack/issues/1131` |
| 5 | Issue #1201 | GitHub Issue | `github.com/garrytan/gstack/issues/1201` |
| 6 | Issue #1218 (+ PR #1220) | GitHub Issue/PR | `github.com/garrytan/gstack/issues/1218` |
| 7 | `docs/ADDING_A_HOST.md` | Official docs | `github.com/garrytan/gstack/blob/main/docs/ADDING_A_HOST.md` |
| 8 | `TODOS.md` | Maintainer roadmap notes | `github.com/garrytan/gstack/blob/main/TODOS.md` |
| 9 | `plan-eng-review/SKILL.md`, `investigate/SKILL.md` (live upstream) | Source code | `github.com/garrytan/gstack/blob/main/plan-eng-review/SKILL.md`, `.../investigate/SKILL.md` |
| 10 | Hacker News thread | Discussion | `news.ycombinator.com/item?id=47355173` |
| 11 | Hacker News thread | Discussion | `news.ycombinator.com/item?id=47418576` |
| 12 | "GStack Tutorial..." | Third-party blog | `buildthisnow.com/blog/guide/agents/garry-tan-gstack-claude-code` |
| 13 | "What is gstack?" | Third-party blog | `docs.bswen.com/blog/2026-03-31-what-is-gstack-claude-code/` |
| 14 | "GStack: Turn Claude Code Into..." | Third-party blog | `agentconn.com/blog/gstack-claude-code-harness-open-source-2026/` |
| 15 | Augment Code coverage | Third-party blog | `augmentcode.com/learn/garry-tan-open-sources-gstack-claude-code` |

---

## 1. Confirmed Community-Recognized Bug Class: ".claude path leakage"

This is the single most important finding from this research pass, and it
**directly corroborates the leading hypothesis** in
`truth-statement-gstack-artifact-saving-inconsistency.md` §2.

**Source 7** (`docs/ADDING_A_HOST.md`) documents GStack's own host-config
system. It explicitly defines a `pathRewrites` mechanism whose entire job is
converting the literal `~/.claude/skills/gstack` path into the correct
location for every other host, and states plainly: `usesEnvVars: true` is
required for every host *except* Claude — Claude is the **only** host
intended to retain literal `~` paths.

**Source 6** (Issue #1218, adding `pi` as a supported host) confirms this is
an actively-tested, named concern in GStack's own contributor practice — the
PR's own verification steps include:
- `bun run gen:skill-docs --host pi → 43 skills generated, no .claude path
  leakage`
- `grep -r '.claude/skills' .pi/skills/ → zero results`

The same issue states directly, describing how existing hosts work: *"gstack's
host adapters for cursor, kiro, opencode are mostly string replacement --
rename tools, rewrite paths, done."*

**Implication for this investigation:** GStack's own contributors treat
"a non-Claude host's generated skill files still contain hardcoded
`~/.claude/skills/gstack/...` paths" as a real, known, specifically-tested-for
failure mode — not a hypothetical. This strengthens, on community/maintainer
authority, the theory that this project's live skill files
(`/mnt/c/mybizz/skills/`) may never have been correctly generated for
the OpenCode host, and instead carry the Claude-only literal paths that the
official tooling is specifically designed to rewrite.

---

## 2. A Second, Independent Slug-Computation Mechanism Found

**Source 9** (`plan-eng-review/SKILL.md`, live upstream) contains, in its
office-hours-recovery flow:

```
SLUG=$(~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
```

This calls a **different binary** (`browse/bin/remote-slug`) than the one
examined directly in the prior Truth Statement (`bin/gstack-slug`), with a
**different fallback chain** (straight to `basename` of the git toplevel,
no git-remote-based slug attempt at all).

**This is a new finding, not previously documented**: GStack does not have
one single slug-computation function used consistently everywhere. At least
two different mechanisms exist (`bin/gstack-slug` and
`browse/bin/remote-slug`), invoked from different skills, with different
fallback logic. This alone could produce inconsistent project identifiers
even in an environment where every path resolves correctly and every binary
exists — independent of the `.claude` path-leakage issue in §1.

---

## 3. A Parallel, Previously-Reported Bug of the Same Shape

**Source 1** (Issue #3, March 2026) reports that a project-local install can
silently break the `/browse` skill because the compiled binary falls back to
a **hardcoded path** (`~/.claude/skills/gstack/browse/src/server.ts`) when no
user-level install exists at that exact location — and, separately, that
`./setup` could **report success even when the actual binary was never
built**.

This is the same *shape* of failure mode hypothesized for `$SLUG`: a
hardcoded path silently failing, with no error surfaced to the user, and a
tool that reports things are fine when they are not. This is not the same
bug, but it establishes that this failure pattern — hardcoded path,
silent failure, false-positive success reporting — has occurred at least
once before in this exact codebase, was reported, and was treated as a real
defect rather than expected behavior.

---

## 4. Maintainer-Acknowledged Data-Integrity Weakness in the Storage Layer

**Source 8** (`TODOS.md`) contains the maintainer's own roadmap note,
attributing the concern to an internal review process ("Codex outside-voice
T5"): the JSONL files used for `learnings.jsonl`, `timeline.jsonl`, and
related cross-session state are explicitly called *"the wrong primitive"*
for multi-writer canonical state, citing **lost-update on rewrite,
partial-line corruption on crash, and no transactions**. A migration to
SQLite is noted as the intended long-term fix, with v1.8.0.0 noted as having
added file-locking (`flock` + `O_APPEND`) as an interim hardening measure.

This does not directly explain the `$SLUG`/save-location inconsistency, but
it is independent, maintainer-sourced confirmation that the artifact storage
layer underlying GStack's cross-skill memory has known structural fragility,
acknowledged by the project itself rather than inferred from outside.

---

## 5. What Was *Not* Found

- **No Reddit discussion of this specific issue.** The supplied thread could
  not be fetched directly (blocked), and broader Reddit searches surfaced
  only the unrelated productivity-claims debate. If Reddit discussion of
  this specific bug exists, it was not found by this search pass — this is
  a gap, not a confirmed absence.
- **No GitHub Issue was found that reports the exact symptom** (artifacts
  scattered across inconsistent project-slug folders, or a skill asking to
  save and then saving nothing when declined). The issues found (§1, §3) are
  adjacent and structurally similar, not exact matches. It is possible this
  specific combination has not yet been reported upstream, or exists in an
  issue this search did not surface (GitHub's own issue search is not fully
  indexed by general web search).
- **Hacker News discussion (Sources 10–11)** centers entirely on the
  productivity/LOC-claims debate ("is 10K lines/week meaningful," "is this
  just prompts in a folder"). No technical discussion of artifact storage or
  multi-host path correctness was found there.

---

## 6. What the Community/Maintainer Record Suggests Works

- **The `pathRewrites` + `usesEnvVars` host-config mechanism (§1)** is the
  *intended* fix for exactly this class of problem, and is treated as a
  solved, tested pattern for officially-supported hosts (Cursor, Kiro,
  OpenCode, Factory, Slate) per Issue #1218's own description.
- **Explicit grep-for-leakage testing** (`grep -r '.claude/skills' <host>/skills/`)
  is the verification method GStack's own contributors use to confirm a host
  adapter is clean. This is a directly reusable diagnostic for this project's
  own situation — a more general version of the single-binary check proposed
  in the Truth Statement.

## 7. What Does Not Appear to Work, Per the Issue Record

- **Relying on `./setup`'s own success message** is not fully trustworthy
  per Issue #3 — it has reported success in the past when the actual
  artifact was missing.
- **Some host adapters are incomplete in practice**, not just in theory: per
  Issue #1201, `setup --host hermes` was, at the time of that report, only a
  help banner rather than a real installer, despite the host being listed as
  "supported." This establishes that host coverage breadth (the README's
  host table) does not guarantee that every listed host's install path is
  fully implemented end-to-end at any given time.
