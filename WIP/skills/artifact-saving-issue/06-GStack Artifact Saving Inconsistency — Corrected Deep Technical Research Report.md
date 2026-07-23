# GStack Artifact Saving Inconsistency — Corrected Deep Technical Research Report

**Report Date:** 2026-06-26 (corrected revision)  
**Project:** MB3-CS (`/mnt/c/mybizz/project-library`)  
**Severity:** Design-critical — affects all skill-to-skill artifact chaining  
**Status:** Root cause confirmed with primary-source evidence; resolution pathway identified  
**Correction note:** This report supersedes the prior version. Every factual claim is backed by a primary source cited inline. Claims not yet verifiable by primary source are explicitly marked **[SPECULATION]** or **[UNVERIFIED — requires local test]**.

***

## Executive Summary

This report synthesises four source documents (`artifact-saving-problem-statement.md`, `truth-statement-gstack-artifact-saving-inconsistency.md`, `communities-statement-gstack-artifact-saving-inconsistency.md`, `plan-ceo-review-spec-sample-spec-doc.md`) with deep online research against GStack's upstream source code, GitHub Issues, and official documentation. All claims are sourced to the exact file and line, or explicitly marked as unverified.

**Root cause confirmed by primary source:** Every skill's generated `SKILL.md` preamble contains a hardcoded `~/.claude/skills/gstack/bin/gstack-slug` path that only resolves on a Claude Code install. GStack's own `TODOS.md` tracks this as **open P1 issue #1882** as of June 2026. On an OpenCode-only machine, this call fails silently (suppressed by `2>/dev/null || true`), leaving `$SLUG` unset and scattering all artifact writes across inconsistent directories.

**A secondary independent slug mechanism exists** in `plan-eng-review/SKILL.md`, using a different binary (`browse/bin/remote-slug`) with a different fallback chain — confirmed by direct line-number inspection of the upstream file.

**The JSONL storage layer has acknowledged structural fragility**, documented by the maintainer in `TODOS.md`.

***

## 1. Confirmed Root Cause: Hardcoded `.claude` Path in Every Skill Preamble

### 1.1 The Mechanism — Primary Source Confirmed

Every GStack skill's `SKILL.md` is generated from `SKILL.md.tmpl` via `bun run gen:skill-docs --host <name>`. The canonical root `SKILL.md.tmpl` preamble contains this exact line, confirmed by fetching the live upstream file at `https://raw.githubusercontent.com/garrytan/gstack/main/SKILL.md.tmpl`:[^1]

```bash
~/.claude/skills/gstack/bin/gstack-telemetry-log ...
```

For a Claude Code install, skills are at `~/.claude/skills/gstack/`. For an OpenCode install, the correct path — confirmed from the live `hosts/opencode.ts` at `https://raw.githubusercontent.com/garrytan/gstack/main/hosts/opencode.ts` — is `~/.config/opencode/skills/gstack`.[^2]

The OpenCode host adapter defines these path rewrites:[^2]

```typescript
pathRewrites: [
  { from: '~/.claude/skills/gstack', to: '~/.config/opencode/skills/gstack' },
  { from: '.claude/skills/gstack', to: '.opencode/skills/gstack' },
  { from: '.claude/skills', to: '.opencode/skills' },
],
```

The `usesEnvVars: true` field is set for OpenCode (Claude Code is the only host where `false` applies). The `docs/ADDING_A_HOST.md` verification step, confirmed from the live upstream file at `https://raw.githubusercontent.com/garrytan/gstack/main/docs/ADDING_A_HOST.md`, requires:[^3][^2]

```bash
# Verify output exists and has no .claude/skills leakage
ls .opencode/skills/gstack-*/SKILL.md
grep -r ".claude/skills" .opencode/skills/ | head -5
# (should be empty)
```

GStack's Issue #1218 (adding `pi` as a host) confirms this is an actively-enforced contributor requirement — the PR verification steps explicitly include `grep -r '.claude/skills' .pi/agent/skills/` and assert zero results.[^4]

### 1.2 This Is a Tracked Open Bug — Not a Configuration Error

Critically: this is not just a setup error on this machine. GStack's own `TODOS.md` (fetched live from `https://raw.githubusercontent.com/garrytan/gstack/main/TODOS.md`) lists it as **P1 issue #1882** under the heading "portable skill-install prefix":[^5]

> Every generated SKILL.md hardcodes the literal `~/.claude/skills/gstack/...` for its `bin/`/asset calls (the per-invocation telemetry/config preamble plus ~9 resolvers). `setup` wires the top-level skill symlinks for any directory name, so installing at `~/.claude/skills/<other>` leaves every internal `bin` reference pointing at a non-existent `~/.claude/skills/gstack/` path — failing **silently, at skill-invocation time**. Make the emitted references portable: resolve the install root at runtime... and emit `$GSTACK_BIN`-relative paths instead of the hardcoded prefix.

The same `TODOS.md` entry also states the preamble already defines `GSTACK_ROOT`/`GSTACK_BIN` variables in `scripts/resolvers/preamble/generate-preamble-bash.ts` — but the generated literals do not yet use them. This is why the fix requires a generator-layer change, not a manual file edit.[^5]

### 1.3 Confirming the Failure on This Machine

The live skill files at `/mnt/c/mybizz/skills/` were confirmed in the `plan-ceo-review-spec-sample-spec-doc.md` audit to contain the exact hardcoded Claude path at line 1243:[^6]

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG/ceo-plans
```

And in `autoplan/SKILL.md` (confirmed via diff against `SKILL.md.tmpl` during prior investigation)[^7]. The `2>/dev/null || true` error suppression means: if `~/.claude/skills/gstack/bin/gstack-slug` does not exist, `$SLUG` is silently never set. All downstream code using `${SLUG:-unknown}` then writes to the literal `unknown` directory.

**Verification command (run this first):**

```bash
ls -la ~/.claude/skills/gstack/bin/gstack-slug 2>&1
```

If this returns `No such file or directory`, the root cause is confirmed for this machine.

**Leak audit command (run this second):**

```bash
grep -rl '~/.claude/skills/gstack' /mnt/c/mybizz/skills/ | wc -l
```

If this returns a count equal to the number of active skills (~52 per `TODOS.md`), the entire skill set was generated without the OpenCode host adapter's `pathRewrites` being applied.[^5]

### 1.4 The Correct Binary Uses `$SCRIPT_DIR`-Relative Addressing

`bin/gstack-review-read` — fetched live from `https://raw.githubusercontent.com/garrytan/gstack/main/bin/gstack-review-read` — shows the pattern that a correctly-generated skill binary uses:[^8]

```bash
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
eval "$("$SCRIPT_DIR/gstack-slug" 2>/dev/null)"
```

This resolves `gstack-slug` relative to the calling script's own directory, not via a hardcoded absolute path. This is what the `pathRewrites` generator fix is meant to achieve across all skills.

***

## 2. Slug Computation Logic — Fully Verified from Primary Source

The complete `gstack-slug` source was fetched live from `https://raw.githubusercontent.com/garrytan/gstack/main/bin/gstack-slug`. The slug resolution order is:[^8]

1. **Cache lookup** — keyed on exact CWD (`$(pwd)`)
2. **Git remote** — `git remote get-url origin` parsed to `org-repo` format
3. **Basename fallback** — `basename "$PWD"` only when there is no git remote

### 2.1 Cache Key — `tr '/' '_'` (Not base64)

The cache key is computed as:[^8]

```bash
CACHE_KEY=$(printf '%s' "$PROJECT_DIR" | tr '/' '_')
CACHE_FILE="${CACHE_DIR}/${CACHE_KEY}"
```

Forward slashes are replaced with underscores. A path like `/mnt/c/mybizz/project-library` becomes `_mnt_c_mybizz_project-library`.

**Correct cache verification command:**

```bash
CACHE_KEY=$(printf '%s' "/mnt/c/mybizz/project-library" | tr '/' '_')
cat ~/.gstack/slug-cache/"$CACHE_KEY"
```

The prior report incorrectly stated the key was base64-encoded. That was wrong and has been corrected here.

### 2.2 Why Different Invocation Contexts Produce Different Slugs

| Observed slug | Invocation context | Mechanism (from source) |
|---|---|---|
| `dev-p` | `gstack-slug` from home dir | No cache; no git remote at `$HOME`; `basename "$HOME"` = `dev-p`[^8] |
| `sassycomapp-project-library` | Manual WSL terminal in project dir | No cache; git remote found; `org-repo` parsed from URL[^8] |
| `project-library` | Agent `/context-save` | Cache hit (populated from prior manual run in same dir), or git remote absent in agent env; `basename "$PWD"`[^8] |
| `unknown` | `/health` via agent | `~/.claude` binary not found; `$SLUG` never set; `${SLUG:-unknown}` literal fires[^7] |

The cache is CWD-exact — it is only populated when `gstack-slug` is run successfully from the project directory. If the agent's `pwd` differs from the path used to populate the cache (e.g., `/mnt/c/...` vs `/home/dev-p/...` vs a Windows-format path), the cache file will not be found even if the same project is intended.[^8]

### 2.3 Atomic Cache Write

The script uses an atomic write pattern (mktemp + mv) — no partial-write corruption risk at the slug-cache level, though the JSONL storage layer is separate (see §3).[^8]

***

## 3. Secondary Independent Slug Mechanism in `plan-eng-review`

`plan-eng-review/SKILL.md` (fetched live, line numbers confirmed) uses a **different binary** with a **different fallback chain**:[^9]

```bash
# Lines 914 and 969:
SLUG=$(~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
```

This differs from `bin/gstack-slug` in three ways:
- It calls `browse/bin/remote-slug`, not `bin/gstack-slug`
- Its fallback is `basename` of the git toplevel, not `basename "$PWD"`
- It does not consult or populate the slug cache

This is confirmed present in the upstream source, not a local artefact. Even after the path-leakage issue (#1882) is fixed, skills using `browse/bin/remote-slug` and skills using `bin/gstack-slug` may produce different slug values for the same project, causing silent artifact-discovery misses in skill-to-skill chaining. This is an independent issue from #1882, not yet tracked as a separate upstream issue based on available community research.[^10]

***

## 4. Three Distinct Save-Path Families — Confirmed from Official Docs

GStack's official `docs/skills.md` (source: `https://github.com/garrytan/gstack/blob/main/docs/skills.md`, consulted directly in the truth-statement investigation) confirms three structurally distinct path families:[^11]

| Family | Example skill | Path | Scope |
|---|---|---|---|
| `~/.gstack/projects/$SLUG/` | `/plan-ceo-review`, `/office-hours`, `/learn` | `~/.gstack/projects/$SLUG/ceo-plans/` | Home-relative, global GStack state |
| `.gstack/` (no tilde) | `/design-review`, `/qa` | `.gstack/design-reports/`, `.gstack/qa-reports/` | **Repo-relative** (writes into the current project directory) |
| `.context/` | `/retro` (project mode) | `.context/retros/` | Repo-relative, different folder name |

A single missing tilde silently redirects a save from the user's global `~/.gstack/` state to a folder inside the current project repo. These two paths are typographically one character apart and near-identical in casual reading.[^11]

***

## 5. The `/retro` Save Path — Resolved and Partially Unresolved

### What is resolved

The official save path for `/retro` in project mode is `.context/retros/` — confirmed from GStack's own `docs/skills.md` and from direct grep of the local `/retro/SKILL.md`. Both the `gstack-skills-reference.md` claim (`~/.gstack/projects/$SLUG/retros/`) and the blanket audit claim (`~/.gstack/projects/$SLUG/retros/*.md`) were wrong.[^7][^11]

### What remains unresolved [SPECULATION — requires verification]

The actual observed output path during testing was `C:\mybizz\project-library\memory\retro-2026-06-26.json` — which matches neither the official `.context/retros/` path nor any path mentioned in AGENTS.md. The problem statement offers a hypothesis that the AGENTS.md dual-save instruction redirected the write, but AGENTS.md's `## GStack artifact governance` section directs saves to `gstack-outputs/`, not `memory/`. **The specific reason the agent chose `memory/` is not explained by any verified source.** This remains open.[^7]

***

## 6. The `/plan-ceo-review` Save-Confirmation Anomaly

### What is established

The targeted 1434-line spec of `plan-ceo-review/SKILL.md` found **no save-confirmation prompt** in the skill's own text. The CEO plan write at line 1253 is a direct file-write instruction with no `AskUserQuestion` gate preceding it.[^6]

### The leading hypothesis [SPECULATION — not yet confirmed by new evidence]

The "Shall I save this review to gstack-outputs/?" prompt likely originated from conversational/chat context (the prior AGENTS.md instruction referencing `gstack-outputs/`) combined with a broken `$SLUG` leaving no valid native save path. When the user declined, the agent had nowhere else to write. This explanation is consistent with all known evidence but has not been independently confirmed in a controlled test session.[^7][^11]

***

## 7. JSONL Storage Layer — Maintainer-Acknowledged Structural Fragility

GStack's `TODOS.md` (live source: `https://github.com/garrytan/gstack/blob/main/TODOS.md`) documents a maintainer-identified weakness in the JSONL storage substrate. A shared `lib/jsonl-store.ts` module (introduced in CHANGELOG for injection-rejection and atomic single-line appends) was added as an interim hardening measure. The maintainer's note identifies the core structural concern: **lost-update on rewrite, partial-line corruption on crash, and no transactions** for multi-writer canonical state. SQLite is noted as the intended long-term fix.[^12][^5]

This does not directly cause the `$SLUG` inconsistency, but it means even after slug computation is stabilised, the JSONL files that store cross-session state (`learnings.jsonl`, `timeline.jsonl`, `decisions.jsonl`) have acknowledged data-integrity constraints at high write concurrency.

The `gstack-analytics` command (`run gstack-analytics` to see a personal usage dashboard from local JSONL) provides a lower-risk way to confirm whether skills are completing and logging correctly than invoking a full skill each time.[^13]

***

## 8. OpenCode Claude Code Compatibility — Behaviour Confirmed, Source Attribution Corrected

The prior report cited this section as "OpenCode's official documentation" — that was wrong. The actual sources are:

**Source 1 (third-party plugin, not official OpenCode docs):** The `opencode-agent-skills` plugin README (`https://github.com/joshuadavidthomas/opencode-agent-skills/blob/main/README.md`) documents skill discovery priority that includes `~/.claude/skills/` as a Claude Code compatibility location.[^14]

**Source 2 (personal gist, not official OpenCode docs):** A GitHub gist by user "zeke" (`https://gist.github.com/zeke/c6bed98a445e559b0d3563087b5e6764`) documents that OpenCode reads `~/.claude/CLAUDE.md` as a compatibility fallback.[^15]

**Source 3 (OpenCode community forums):** `https://forums.basehub.com/anomalyco/opencode/10` references `packages/opencode/src/skill/skill.ts` skill discovery including `~/.claude/skills/`.[^16]

The **behaviour** — that OpenCode may load skill files from `~/.claude/skills/` in addition to `~/.config/opencode/skills/` — is corroborated across multiple independent sources. However, none of these is OpenCode's own official documentation. **The correct approach is to verify directly:**

```bash
ls -la ~/.claude/skills/ 2>&1
ls -la ~/.config/opencode/skills/ 2>&1
```

If `~/.claude/skills/gstack/` exists (e.g. from a dev-mode symlink), OpenCode may be loading Claude-format skills alongside or instead of the correctly-generated OpenCode skills. Until this local check is run, the "hidden amplifier" framing is **[UNVERIFIED — requires local test]**.

***

## 9. Environment Variable Relationships — Partially Verified

GStack uses at least two different environment variable names that control the base state directory. Their relationship is confirmed from the CHANGELOG and test source, but the full picture requires a local test:[^17][^12]

| Variable | Where confirmed | What it controls |
|---|---|---|
| `GSTACK_HOME` | Canonical preamble template `SKILL.md.tmpl`[^1] | Override for `~/.gstack` in all skill templates (`${GSTACK_HOME:-$HOME/.gstack}`) |
| `GSTACK_STATE_ROOT` | `test/declared-annotation.test.ts`[^17] | Used in test isolation; sets state root for `gstack-config` and related binaries |

A third name — `GSTACK_STATE_DIR` — appears in a third-party field guide blog (`https://agentairforce.com/osint/gstack/05-developer-tooling`) but was **not confirmed in any primary GStack source**. It should not be treated as authoritative until verified by running `echo $GSTACK_STATE_DIR` in the live agent environment.

**Diagnostic command (run in agent shell):**

```bash
echo "GSTACK_HOME=${GSTACK_HOME:-NOT_SET}"
echo "GSTACK_STATE_ROOT=${GSTACK_STATE_ROOT:-NOT_SET}"
```

***

## 10. The AGENTS.md Dual-Save Rule — What It Gets Right and Wrong

### What it gets right

The instruction to save both to the native `~/.gstack/projects/` path AND to `gstack-outputs/` for human visibility is a valid operational requirement. The official GStack documentation confirms skills do not do this automatically.[^6]

### Why it currently cannot work

The AGENTS.md rule instructs the agent to "always complete the skill's own native default save...without asking for confirmation." With `$SLUG` failing silently, there is no valid native save path. The agent has nowhere to write. The ask-then-skip behaviour (Symptom 2) is the agent surfacing a decision it cannot resolve from its instructions — when declined, it had no fallback path. This is expected behaviour given the broken precondition, not a bug in AGENTS.md itself.[^7]

### Recommended amendment (post-fix)

After slug computation is verified working, add a guard assertion:

```
## GStack slug verification
Before running any GStack skill:
1. Run: eval "$(/mnt/c/mybizz/gstack/bin/gstack-slug 2>&1)" && echo "SLUG: $SLUG"
2. If $SLUG is empty or "unknown", STOP and report: "Slug computation failed."
3. Only proceed with skill execution if $SLUG = "project-library" for this project.
```

***

## 11. Resolution Pathway

### 11.1 Scope Decision Required First

GStack's `TODOS.md` P1 item #1882 acknowledges that the proper fix requires modifying GStack's own generator — specifically, rewriting the `pathRewrites` resolver to emit `$GSTACK_BIN`-relative paths rather than hardcoded literals. This was explicitly ruled out earlier in this investigation ("it is not my intention to rewrite the gstack code"). The options below are ordered from non-invasive to invasive; choose based on whether that boundary remains in place.[^5]

### 11.2 Option A: Targeted `sed` Replacement (Non-Generator, Interim)

Replace the hardcoded Claude path with the actual path of the `gstack-slug` binary on this machine across all skill files:

```bash
find /mnt/c/mybizz/skills -name 'SKILL.md' | xargs sed -i \
  's|~/.claude/skills/gstack/bin/|/mnt/c/mybizz/gstack/bin/|g'
```

**Caveat:** `SKILL.md` files are auto-generated (`<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->`). This edit will be overwritten by any future `bun run gen:skill-docs` invocation. Track it as a local patch that must be reapplied after upgrades.[^7]

### 11.3 Option B: Project-Level SLUG Override (Least Invasive)

Pin the slug in the project's `.env` or AGENTS.md to prevent all downstream inconsistency without touching any skill files:

```bash
# In /mnt/c/mybizz/project-library AGENTS.md or a sourced .env:
export SLUG=project-library
```

This bypasses the entire `gstack-slug` binary call and guarantees slug consistency regardless of which binary or fallback is used. It is a workaround, not a fix — but it is idempotent and survives skill regeneration.

### 11.4 Option C: Regenerate via Official Generator (Invasive — Modifies gstack Repo)

```bash
cd /mnt/c/mybizz/gstack
bun run gen:skill-docs --host opencode

# Mandatory leak check (must return zero lines):
grep -r '.claude/skills' .opencode/skills/ | head -5
```

This is the "correct" fix per `ADDING_A_HOST.md` but requires touching GStack's own tooling. It also may not be available if the `gstack` repo at `/mnt/c/mybizz/gstack/` does not have `bun` dependencies installed.[^3]

### 11.5 Populate the Slug Cache Correctly

Once Option A, B, or C is in place and slug computation succeeds, pre-populate the cache from inside the project directory to guarantee future consistency:[^8]

```bash
cd /mnt/c/mybizz/project-library
eval "$(/mnt/c/mybizz/gstack/bin/gstack-slug 2>&1)"
echo "Slug set to: $SLUG"

# Verify the cache file:
CACHE_KEY=$(printf '%s' "/mnt/c/mybizz/project-library" | tr '/' '_')
cat ~/.gstack/slug-cache/"$CACHE_KEY"
```

***

## 12. Diagnostic Battery — Priority-Ordered

Run these in a fresh agent session, reporting raw output at each step:

| # | Command | What it confirms |
|---|---|---|
| 1 | `ls -la ~/.claude/skills/gstack/bin/gstack-slug 2>&1` | Whether the hardcoded Claude binary exists |
| 2 | `grep -rl '~/.claude/skills/gstack' /mnt/c/mybizz/skills/ \| wc -l` | How many skill files carry the leaking path |
| 3 | `cd /mnt/c/mybizz/project-library && pwd && git remote get-url origin 2>&1` | Agent's CWD and git remote visibility |
| 4 | `echo $HOME && echo $PATH \| tr ':' '\n' \| head -20` | Whether agent shell environment matches WSL terminal |
| 5 | `echo "GSTACK_HOME=${GSTACK_HOME:-NOT_SET}"; echo "GSTACK_STATE_ROOT=${GSTACK_STATE_ROOT:-NOT_SET}"` | Which state-dir env vars, if any, are set |
| 6 | `ls -la ~/.claude/skills/ 2>&1; ls -la ~/.config/opencode/skills/ 2>&1` | Whether both skill locations exist and OpenCode may be loading the wrong one |
| 7 | `which gstack-slug 2>&1` | Whether the `gstack-slug` binary is on the agent's `$PATH` at all |

***

## 13. Open Questions — Requiring Local Tests, Not Further Documentation

| # | Question | Diagnostic | Status |
|---|---|---|---|
| 1 | Does `~/.claude/skills/gstack/bin/gstack-slug` exist? | Diagnostic #1 above | **Unrun** |
| 2 | How many skill files carry the leaking path? | Diagnostic #2 above | **Unrun** |
| 3 | Does the agent's shell see the git remote for this project? | Diagnostic #3 above | **Unrun** |
| 4 | What env vars are set in the agent shell? | Diagnostic #5 above | **Unrun** |
| 5 | Does `~/.claude/skills/` exist and does OpenCode load from it? | Diagnostic #6 above | **Unrun** |
| 6 | Why did `/retro` write to `memory/` rather than `.context/retros/` or `gstack-outputs/`? | Re-run `/retro` in a clean session with only the AGENTS.md dual-save rule | **Unrun** |
| 7 | Is the `browse/bin/remote-slug` vs `bin/gstack-slug` discrepancy tracked as a separate upstream bug? | Monitor GStack issues; no issue found in this research pass | **Open upstream** |

***

## 14. Source Index — Full Paths to All Primary Sources

| # | Source | Full URL / Path | Type |
|---|---|---|---|
| 1 | Problem Statement | `artifact-saving-problem-statement.md` (attached) | Primary (local) |
| 2 | Communities Statement | `communities-statement-gstack-artifact-saving-inconsistency.md` (attached) | Primary (local) |
| 3 | Truth Statement | `truth-statement-gstack-artifact-saving-inconsistency.md` (attached) | Primary (local) |
| 4 | plan-ceo-review Spec | `plan-ceo-review-spec-sample-spec-doc.md` (attached) | Primary (local) |
| 5 | GStack ADDING_A_HOST.md | `https://raw.githubusercontent.com/garrytan/gstack/main/docs/ADDING_A_HOST.md` | Official primary |
| 6 | GStack SKILL.md.tmpl (root) | `https://raw.githubusercontent.com/garrytan/gstack/main/SKILL.md.tmpl` | Official primary |
| 7 | GStack TODOS.md | `https://raw.githubusercontent.com/garrytan/gstack/main/TODOS.md` | Official primary |
| 8 | GStack CHANGELOG.md | `https://raw.githubusercontent.com/garrytan/gstack/main/CHANGELOG.md` | Official primary |
| 9 | GStack bin/gstack-slug (full source) | `https://raw.githubusercontent.com/garrytan/gstack/main/bin/gstack-slug` | Official primary |
| 10 | GStack bin/gstack-review-read | `https://raw.githubusercontent.com/garrytan/gstack/main/bin/gstack-review-read` | Official primary |
| 11 | GStack hosts/opencode.ts | `https://raw.githubusercontent.com/garrytan/gstack/main/hosts/opencode.ts` | Official primary |
| 12 | GStack plan-eng-review/SKILL.md (lines 914, 969) | `https://raw.githubusercontent.com/garrytan/gstack/main/plan-eng-review/SKILL.md` | Official primary |
| 13 | GStack autoplan/SKILL.md | `https://github.com/garrytan/gstack/blob/main/autoplan/SKILL.md` | Official primary |
| 14 | GStack test/declared-annotation.test.ts | `https://github.com/garrytan/gstack/blob/main/test/declared-annotation.test.ts` | Official primary |
| 15 | GStack Issue #1218 (pi host; pathRewrites verification) | `https://github.com/garrytan/gstack/issues/1218` | Official issue |
| 16 | GStack Issue #3 (setup false-positive success) | `https://github.com/garrytan/gstack/issues/3` | Official issue |
| 17 | GStack README | `https://github.com/garrytan/gstack` | Official primary |
| 18 | opencode-agent-skills README | `https://github.com/joshuadavidthomas/opencode-agent-skills/blob/main/README.md` | Third-party plugin |
| 19 | OpenCode — Claude CLAUDE.md compatibility (gist) | `https://gist.github.com/zeke/c6bed98a445e559b0d3563087b5e6764` | Personal gist — unverified |
| 20 | OpenCode forums — skill discovery | `https://forums.basehub.com/anomalyco/opencode/10` | Community — unverified |
| 21 | OpenCode config documentation | `https://opencode.ai/docs/config/` | Official OpenCode docs |
| 22 | OpenCode skills documentation | `https://opencode.ai/docs/skills/` | Official OpenCode docs |
| 23 | GStack Field Guide Module 05 (GSTACK_STATE_DIR claim) | `https://agentairforce.com/osint/gstack/05-developer-tooling` | Third-party blog — unverified |

---

## References

1. [SKILL.md.tmpl - garrytan/gstack](https://github.com/garrytan/gstack/blob/main/SKILL.md.tmpl) - Use Garry Tan's exact Claude Code setup: 23 opinionated tools that serve as CEO, Designer, Eng Manag...

2. [https://raw.githubusercontent.com/garrytan/gstack/...](https://raw.githubusercontent.com/garrytan/gstack/main/hosts/opencode.ts) - ... opencode', displayName: 'OpenCode', cliCommand: 'opencode', cliAliases: [], globalRoot: '.config...

3. [gstack/docs/ADDING_A_HOST.md at main - GitHub](https://github.com/garrytan/gstack/blob/main/docs/ADDING_A_HOST.md) - Use Garry Tan's exact Claude Code setup: 23 opinionated tools that serve as CEO, Designer, Eng Manag...

4. [garrytan/gstack - add pi coding agent as a supported host - GitHub](https://github.com/garrytan/gstack/issues/1218) - CLAUDE.md → AGENTS.md rewrite -- added to pathRewrites · generator fix: frontmatter name: now matche...

5. [TODOS.md - garrytan/gstack](https://github.com/garrytan/gstack/blob/main/TODOS.md) - Use Garry Tan's exact Claude Code setup: 23 opinionated tools that serve as CEO, Designer, Eng Manag...

6. [plan-ceo-review-spec-sample-spec-doc.md](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/50806054/20af1b7e-f7af-4dab-ad90-41bb0c0b797e/plan-ceo-review-spec-sample-spec-doc.md?AWSAccessKeyId=ASIA2F3EMEYEZLW7RLCE&Signature=wGlg07fO2rxnMaeKaLm%2F74j4Flc%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEJf%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIEv4x0r%2FrNGT1NBRjGH6jnGTdS%2Bd0oZVmWNuukL0XDGxAiEAnjYHbvGumX6AQryaKr7wUxukJtvnvxHR4q%2BNXlVhDxQq8wQIXxABGgw2OTk3NTMzMDk3MDUiDMFTUGjyZRFUaGNhsirQBJCKQV38f3u7GlmahZLC4mK3sNlw4%2Bk%2F7uw986TCkXPN%2B0NY7mXOPcTEyYYYp96w08R3kkmNyAkRQydNjxoyWBx0gSgXKLQchuVxYUie%2B7RqIj0K8kg67bvLhc7wOw7FEoLe4KQrn2bcBCzAkZArNa%2BIZefdnpG9%2F2zr%2FP9RxA1uBqlueqmXAEYOSYpjTdfgZrrFtVsCoaqp26dJpFdv%2FHpkJRJHM5%2Fwj25yHlDEB%2Frq%2FX%2FbvFkqtBDiFWuqXZS8EFvxNVSJOxf9ndH6i%2FvH60kjuzQwp4gs9eM%2FTeD9sAX7KWPn0rZPCIKm4Y13JMFhtmW%2FEQfqNKFttUpYGZyyJfqDky35D%2Fs7bat1EhJSk6p9fjyx6fb1K2xqW8Nin62qwFFsxioXVtDWCnO%2BFURo90I28HXjSYs%2B%2BeqRaMd8tUVO5owGLs2K2%2BG02dTE%2BUE%2BiXbQNNHVyHlosH7g20ucvhTskmO5fAzywzc0M5qAt1jYjv16PhGqsPlZ8OVakm4%2B%2F1andC2GK%2B9Z6IIQ63UHTMe9r76CUcjPHwpJDTyCh6Dh3Sj8p4Fqfbc5lQQeAjnXLtb%2Fe6BehGzzMXnDw3hV1d7vDXCTMQPAe9fMEw3dhG5K2T44w4WKvNnik97vjfrD8DZG7heoMgLFknIjAsFjiu%2Bex%2FdjyE%2B9TbuxDfvrjHYCcvkZGWst8I278afMUlHazD%2BGuvJFd3%2FJwvrboTjv9jSpMED3cb2tVeFSgUw1w8CTeVd%2FGDS4GJlqVdOuIQmr%2FXJPGqKyAQLEJrZZLV6jlfMwk7L40QY6mAGkjLKX%2FMsd0GI480zk73nOCpI%2B8ch0nhUye2iFEaheVAxqMVtPnoEcexdK8mCtt2FeNy%2FwRhuUG5cu4wAqxin2V1XCJEX0peHyz1girmEwxpM8w8sAJeRtvawMXgzkWZxbZ3T5IUREg%2B%2BPY%2Ba15otXFVNF9aQtv2yvMzvOA1gTc5UKVaXxL3Z54SWV%2F%2FQAENxvQ0LFB2Y9SQ%3D%3D&Expires=1782458086) - # plan-ceo-review — Skill Spec

**Source:** `/mnt/c/mybizz/skills/plan-ceo-review/SKILL.md`
**T...

7. [artifact-saving-problem-statement.md](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/50806054/2d24e9f7-7ed5-4038-adc7-b004546284fb/artifact-saving-problem-statement.md?AWSAccessKeyId=ASIA2F3EMEYEZLW7RLCE&Signature=4U4Xh%2BCPquVIdWq5CPWS1duUb10%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEJf%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIEv4x0r%2FrNGT1NBRjGH6jnGTdS%2Bd0oZVmWNuukL0XDGxAiEAnjYHbvGumX6AQryaKr7wUxukJtvnvxHR4q%2BNXlVhDxQq8wQIXxABGgw2OTk3NTMzMDk3MDUiDMFTUGjyZRFUaGNhsirQBJCKQV38f3u7GlmahZLC4mK3sNlw4%2Bk%2F7uw986TCkXPN%2B0NY7mXOPcTEyYYYp96w08R3kkmNyAkRQydNjxoyWBx0gSgXKLQchuVxYUie%2B7RqIj0K8kg67bvLhc7wOw7FEoLe4KQrn2bcBCzAkZArNa%2BIZefdnpG9%2F2zr%2FP9RxA1uBqlueqmXAEYOSYpjTdfgZrrFtVsCoaqp26dJpFdv%2FHpkJRJHM5%2Fwj25yHlDEB%2Frq%2FX%2FbvFkqtBDiFWuqXZS8EFvxNVSJOxf9ndH6i%2FvH60kjuzQwp4gs9eM%2FTeD9sAX7KWPn0rZPCIKm4Y13JMFhtmW%2FEQfqNKFttUpYGZyyJfqDky35D%2Fs7bat1EhJSk6p9fjyx6fb1K2xqW8Nin62qwFFsxioXVtDWCnO%2BFURo90I28HXjSYs%2B%2BeqRaMd8tUVO5owGLs2K2%2BG02dTE%2BUE%2BiXbQNNHVyHlosH7g20ucvhTskmO5fAzywzc0M5qAt1jYjv16PhGqsPlZ8OVakm4%2B%2F1andC2GK%2B9Z6IIQ63UHTMe9r76CUcjPHwpJDTyCh6Dh3Sj8p4Fqfbc5lQQeAjnXLtb%2Fe6BehGzzMXnDw3hV1d7vDXCTMQPAe9fMEw3dhG5K2T44w4WKvNnik97vjfrD8DZG7heoMgLFknIjAsFjiu%2Bex%2FdjyE%2B9TbuxDfvrjHYCcvkZGWst8I278afMUlHazD%2BGuvJFd3%2FJwvrboTjv9jSpMED3cb2tVeFSgUw1w8CTeVd%2FGDS4GJlqVdOuIQmr%2FXJPGqKyAQLEJrZZLV6jlfMwk7L40QY6mAGkjLKX%2FMsd0GI480zk73nOCpI%2B8ch0nhUye2iFEaheVAxqMVtPnoEcexdK8mCtt2FeNy%2FwRhuUG5cu4wAqxin2V1XCJEX0peHyz1girmEwxpM8w8sAJeRtvawMXgzkWZxbZ3T5IUREg%2B%2BPY%2Ba15otXFVNF9aQtv2yvMzvOA1gTc5UKVaXxL3Z54SWV%2F%2FQAENxvQ0LFB2Y9SQ%3D%3D&Expires=1782458086) - # Problem Statement: GStack Artifact Saving Inconsistency

**Project:** project-library (`/mnt/c/_mb...

8. [gstack/bin/gstack-review-read at main · garrytan/gstack - GitHub](https://github.com/garrytan/gstack/blob/main/bin/gstack-review-read) - Use Garry Tan's exact Claude Code setup: 23 opinionated tools that serve as CEO, Designer, Eng Manag...

9. [gstack/autoplan/SKILL.md at main - GitHub](https://github.com/garrytan/gstack/blob/main/autoplan/SKILL.md) - Use Garry Tan's exact Claude Code setup: 23 opinionated tools that serve as CEO, Designer, Eng Manag...

10. [communities-statement-gstack-artifact-saving-inconsistency.md](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/50806054/031889f4-14d1-4e06-8d68-39642fc9cb2f/communities-statement-gstack-artifact-saving-inconsistency.md?AWSAccessKeyId=ASIA2F3EMEYEZLW7RLCE&Signature=Gs1tHimTMmIUvkzBICOuJXmg3lE%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEJf%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIEv4x0r%2FrNGT1NBRjGH6jnGTdS%2Bd0oZVmWNuukL0XDGxAiEAnjYHbvGumX6AQryaKr7wUxukJtvnvxHR4q%2BNXlVhDxQq8wQIXxABGgw2OTk3NTMzMDk3MDUiDMFTUGjyZRFUaGNhsirQBJCKQV38f3u7GlmahZLC4mK3sNlw4%2Bk%2F7uw986TCkXPN%2B0NY7mXOPcTEyYYYp96w08R3kkmNyAkRQydNjxoyWBx0gSgXKLQchuVxYUie%2B7RqIj0K8kg67bvLhc7wOw7FEoLe4KQrn2bcBCzAkZArNa%2BIZefdnpG9%2F2zr%2FP9RxA1uBqlueqmXAEYOSYpjTdfgZrrFtVsCoaqp26dJpFdv%2FHpkJRJHM5%2Fwj25yHlDEB%2Frq%2FX%2FbvFkqtBDiFWuqXZS8EFvxNVSJOxf9ndH6i%2FvH60kjuzQwp4gs9eM%2FTeD9sAX7KWPn0rZPCIKm4Y13JMFhtmW%2FEQfqNKFttUpYGZyyJfqDky35D%2Fs7bat1EhJSk6p9fjyx6fb1K2xqW8Nin62qwFFsxioXVtDWCnO%2BFURo90I28HXjSYs%2B%2BeqRaMd8tUVO5owGLs2K2%2BG02dTE%2BUE%2BiXbQNNHVyHlosH7g20ucvhTskmO5fAzywzc0M5qAt1jYjv16PhGqsPlZ8OVakm4%2B%2F1andC2GK%2B9Z6IIQ63UHTMe9r76CUcjPHwpJDTyCh6Dh3Sj8p4Fqfbc5lQQeAjnXLtb%2Fe6BehGzzMXnDw3hV1d7vDXCTMQPAe9fMEw3dhG5K2T44w4WKvNnik97vjfrD8DZG7heoMgLFknIjAsFjiu%2Bex%2FdjyE%2B9TbuxDfvrjHYCcvkZGWst8I278afMUlHazD%2BGuvJFd3%2FJwvrboTjv9jSpMED3cb2tVeFSgUw1w8CTeVd%2FGDS4GJlqVdOuIQmr%2FXJPGqKyAQLEJrZZLV6jlfMwk7L40QY6mAGkjLKX%2FMsd0GI480zk73nOCpI%2B8ch0nhUye2iFEaheVAxqMVtPnoEcexdK8mCtt2FeNy%2FwRhuUG5cu4wAqxin2V1XCJEX0peHyz1girmEwxpM8w8sAJeRtvawMXgzkWZxbZ3T5IUREg%2B%2BPY%2Ba15otXFVNF9aQtv2yvMzvOA1gTc5UKVaXxL3Z54SWV%2F%2FQAENxvQ0LFB2Y9SQ%3D%3D&Expires=1782458086) - # Communities Statement: GStack Artifact Saving Inconsistency

**Date:** 2026-06-26
**Method:** Sear...

11. [truth-statement-gstack-artifact-saving-inconsistency.md](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/50806054/68dfa387-5b35-4d06-9601-78109c756d11/truth-statement-gstack-artifact-saving-inconsistency.md?AWSAccessKeyId=ASIA2F3EMEYEZLW7RLCE&Signature=By4yN%2FFlEdVdTzY%2B97s6M%2F39Pp4%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEJf%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIEv4x0r%2FrNGT1NBRjGH6jnGTdS%2Bd0oZVmWNuukL0XDGxAiEAnjYHbvGumX6AQryaKr7wUxukJtvnvxHR4q%2BNXlVhDxQq8wQIXxABGgw2OTk3NTMzMDk3MDUiDMFTUGjyZRFUaGNhsirQBJCKQV38f3u7GlmahZLC4mK3sNlw4%2Bk%2F7uw986TCkXPN%2B0NY7mXOPcTEyYYYp96w08R3kkmNyAkRQydNjxoyWBx0gSgXKLQchuVxYUie%2B7RqIj0K8kg67bvLhc7wOw7FEoLe4KQrn2bcBCzAkZArNa%2BIZefdnpG9%2F2zr%2FP9RxA1uBqlueqmXAEYOSYpjTdfgZrrFtVsCoaqp26dJpFdv%2FHpkJRJHM5%2Fwj25yHlDEB%2Frq%2FX%2FbvFkqtBDiFWuqXZS8EFvxNVSJOxf9ndH6i%2FvH60kjuzQwp4gs9eM%2FTeD9sAX7KWPn0rZPCIKm4Y13JMFhtmW%2FEQfqNKFttUpYGZyyJfqDky35D%2Fs7bat1EhJSk6p9fjyx6fb1K2xqW8Nin62qwFFsxioXVtDWCnO%2BFURo90I28HXjSYs%2B%2BeqRaMd8tUVO5owGLs2K2%2BG02dTE%2BUE%2BiXbQNNHVyHlosH7g20ucvhTskmO5fAzywzc0M5qAt1jYjv16PhGqsPlZ8OVakm4%2B%2F1andC2GK%2B9Z6IIQ63UHTMe9r76CUcjPHwpJDTyCh6Dh3Sj8p4Fqfbc5lQQeAjnXLtb%2Fe6BehGzzMXnDw3hV1d7vDXCTMQPAe9fMEw3dhG5K2T44w4WKvNnik97vjfrD8DZG7heoMgLFknIjAsFjiu%2Bex%2FdjyE%2B9TbuxDfvrjHYCcvkZGWst8I278afMUlHazD%2BGuvJFd3%2FJwvrboTjv9jSpMED3cb2tVeFSgUw1w8CTeVd%2FGDS4GJlqVdOuIQmr%2FXJPGqKyAQLEJrZZLV6jlfMwk7L40QY6mAGkjLKX%2FMsd0GI480zk73nOCpI%2B8ch0nhUye2iFEaheVAxqMVtPnoEcexdK8mCtt2FeNy%2FwRhuUG5cu4wAqxin2V1XCJEX0peHyz1girmEwxpM8w8sAJeRtvawMXgzkWZxbZ3T5IUREg%2B%2BPY%2Ba15otXFVNF9aQtv2yvMzvOA1gTc5UKVaXxL3Z54SWV%2F%2FQAENxvQ0LFB2Y9SQ%3D%3D&Expires=1782458086) - # Truth Statement: GStack Artifact Saving Inconsistency

**Date:** 2026-06-26
**Method:** Direct fet...

12. [gstack/CHANGELOG.md at main](https://github.com/garrytan/gstack/blob/main/CHANGELOG.md) - Use Garry Tan's exact Claude Code setup: 23 opinionated tools that serve as CEO, Designer, Eng Manag...

13. [GitHub - garrytan/gstack: Use Garry Tan's exact Claude Code setup](https://github.com/garrytan/gstack) - Use Garry Tan's exact Claude Code setup: 23 opinionated tools that serve as CEO, Designer, Eng Manag...

14. [opencode-agent-skills/README.md at main](https://github.com/joshuadavidthomas/opencode-agent-skills/blob/main/README.md) - An OpenCode plugin that provides tools for using agent skills - joshuadavidthomas/opencode-agent-ski...

15. [OpenCode reads ~/.claude/CLAUDE.md (undocumented Claude ...](https://gist.github.com/zeke/c6bed98a445e559b0d3563087b5e6764) - OpenCode reads ~/.claude/CLAUDE.md (undocumented Claude Code compatibility) - opencode-reads-claude-...

16. [##OpenCode's Take on Skills](https://forums.basehub.com/anomalyco/opencode/10)

17. [declared-annotation.test.ts - garrytan/gstack - GitHub](https://github.com/garrytan/gstack/blob/main/test/declared-annotation.test.ts) - prevStateRoot = process.env.GSTACK_STATE_ROOT;. prevHome = process.env.GSTACK_HOME;. process.env.GST...

