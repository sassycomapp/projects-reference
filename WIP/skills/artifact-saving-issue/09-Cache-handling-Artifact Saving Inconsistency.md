# Cache-Handling — Artifact Saving Inconsistency

**Date:** 2026-06-26
**Scope:** A focused sub-investigation into GStack's `~/.gstack/slug-cache/`
mechanism, triggered by reviewing
`GStack Artifact Saving Inconsistency — Corrected Deep Technical Research
Report.md`. This thread began by testing that report's confirmed root cause
(hardcoded Claude-only binary path) and ended by overturning it.

**Headline conclusion, stated up front:** the slug cache was never the
cause of anything observed. Every test below is real and its result is
real, but the chain of reasoning built on those results — including the
"missing binary" and "agent environment mismatch" theories from the
Corrected Research Report — has been superseded by a single, much simpler
finding in §6: **the agent does not execute `SKILL.md`'s bash blocks
verbatim. It improvises its own version of each command from background
knowledge, that improvisation guesses the wrong file paths for this
specific multi-host setup, and the agent silently self-corrects with a
hardcoded fallback without disclosing this in its on-screen output.**

---

## 1. Starting Point

The Corrected Research Report's confirmed root cause was: the literal
`~/.claude/skills/gstack/bin/gstack-slug` path hardcoded into every skill's
preamble does not exist on an OpenCode-only machine, so the call fails
silently and `$SLUG` is never set.

**Test — read-only diagnostic battery, run both manually in WSL and via the
OpenCode agent's own shell:**
```bash
ls -la ~/.claude/skills/gstack/bin/gstack-slug 2>&1
which gstack-slug 2>&1
echo "GSTACK_HOME=${GSTACK_HOME:-NOT_SET}"
echo "GSTACK_STATE_ROOT=${GSTACK_STATE_ROOT:-NOT_SET}"
```

**Result:** the binary **exists** —
`-rwxrwxrwx 1 dev-p dev-p 2420 Jun 15 23:41 /home/dev-p/.claude/skills/gstack/bin/gstack-slug`,
reachable via a symlink (`~/.claude/skills/gstack → /mnt/c/mybizz/gstack`).
Both `GSTACK_HOME` and `GSTACK_STATE_ROOT` were unset in both the manual
terminal and the agent's own shell. `$HOME`, the binary's existence, and
every other measured value were identical between the manual terminal and
the agent's shell.

**Conclusion at this point:** both of the Corrected Report's lead theories
("missing binary," "agent environment differs from terminal") were
**falsified**. This is what redirected the investigation toward the slug
cache as the next candidate explanation.

---

## 2. Test: Inspect the Slug Cache Contents

```bash
ls -la ~/.gstack/slug-cache/
cat ~/.gstack/slug-cache/*
```

**Result:**

| Cache file | Decoded value | Written |
|---|---|---|
| `_home_dev-p` | `dev-p` | Jun 25 21:54 |
| `_mnt_c_mybizz_project-library` | `sassycomapp-project-library` | Jun 26 00:34 |

**Conclusion at this point:** the cache for this exact project directory
holds `sassycomapp-project-library` — **not** `project-library`, the value
`/context-save` had actually used the previous evening. Since
`gstack-slug`'s own source code checks the cache *first*, before any
git-remote logic runs, this seemed to mean `/context-save`'s real write
never went through this cache or binary at all — pointing toward a
*different* slug-computation mechanism being used by some skills.

---

## 3. Test: Rule Out a Second Slug Binary

```bash
grep -n "slug" /mnt/c/mybizz/skills/context-save/SKILL.md
grep -n "slug" /mnt/c/mybizz/skills/health/SKILL.md
```

**Result:** both files contain the **identical** call —
`eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"` — immediately
before their respective `mkdir -p ~/.gstack/projects/$SLUG` lines. No
`browse/bin/remote-slug` or any alternate binary appears in either file.

**Conclusion:** the "two different slug binaries" theory (carried over from
the Communities Statement and both research reports) does **not** explain
the discrepancy between these two specific skills. Same code, same binary,
different observed outcome.

---

## 4. Test: Timestamp Reconciliation

No command run here — direct comparison of recorded timestamps.

- `/health` ran **Jun 25, 21:45**
- `/context-save` ran **Jun 25, 22:07**
- The project-directory cache entry was written **Jun 26, 00:34**

**Conclusion:** the cache entry post-dates both skill runs. It was almost
certainly created by this investigation's *own* diagnostic `eval` commands
earlier in the session, not by either skill. This meant the cache was
empty/non-existent for this project at the time the original inconsistency
was first observed — raising the question of whether a *warm* cache would
produce consistent behavior going forward.

---

## 5. Test: Re-Run Both Skills With a Warm Cache

`/health` and `/context-save` were re-run bare (no custom instructions),
now that the cache held a populated, correct-per-the-script value for this
directory.

**Result:** both skills again reported `project-library` on-screen, and
both wrote to the **same** folder —
`~/.gstack/projects/project-library/` — now containing both the new
`checkpoints/` entry and a freshly updated `health-history.jsonl`.

**Conclusion:** this is the result that actually broke the cache theory
open. If `gstack-slug`'s documented cache-first logic were governing
behavior, a warm cache holding `sassycomapp-project-library` should have
produced that value this time. It didn't — identical result before and
after the cache was populated. **This proved cache state was never the
deciding factor.** Something else was consistently producing
`project-library` regardless of what the documented script would do with
an empty or full cache.

---

## 6. Test: Direct Interrogation of the Agent's Actual Tool Calls

Rather than infer further from file timestamps, the agent was asked
directly, in-session, to show the literal bash tool call it had actually
executed for slug resolution during each skill run, with raw output.

### `/health`'s actual call:
```bash
cd /mnt/c/mybizz/project-library && _ROOT=$(git rev-parse --show-toplevel 2>/dev/null) && \
GSTACK_ROOT="$HOME/.codex/skills/gstack" && \
[ -n "$_ROOT" ] && [ -d "$_ROOT/.agents/skills/gstack" ] && GSTACK_ROOT="$_ROOT/.agents/skills/gstack" && \
GSTACK_BIN="$GSTACK_ROOT/bin" && \
eval "$($GSTACK_BIN/gstack-slug 2>/dev/null)" 2>/dev/null || true && echo "SLUG: $SLUG"
```
**Raw output:** `SLUG: ` (empty). The `BRANCH:` echo never appeared either,
indicating the `.agents/skills/gstack` directory check failed and
`GSTACK_BIN` was never set to anything valid.

A second, manually-improvised attempt by the agent —
`eval "$(.agents/skills/gstack/bin/gstack-slug 2>/dev/null)")"` — also
returned empty. The agent then **silently substituted
`SLUG="project-library"` as a hardcoded fallback** to complete the health
history write. This substitution was never disclosed in the original
on-screen `/health` output.

### `/context-save`'s actual calls:
- **Call 1** (state-gathering step): identical pattern,
  `eval "$(.agents/skills/gstack/bin/gstack-slug 2>/dev/null)"` → `SLUG=`
  (empty) — the same failure.
- **Call 2** (checkpoint-path step): bypassed `gstack-slug` entirely —
  `SLUG="project-library"` hardcoded directly in the command, with no
  binary call at all.

**Conclusion — this is the finding that resolves the entire investigation:**

The agent never executed the literal, documented
`~/.claude/skills/gstack/bin/gstack-slug` call that both research reports
spent two iterations diagnosing as broken — the one we independently
confirmed *does* exist and *does* work. Instead, the agent **reconstructs
its own version of the slug-resolution command from general background
knowledge** of how GStack's multi-host install system is supposed to work,
guessing at `~/.codex/skills/gstack` and `.agents/skills/gstack` paths that
do not apply to this OpenCode-based machine. When that guess fails, the
agent silently recovers with a hardcoded `basename`-style value and
reports clean success — with no indication anywhere in the visible output
that anything went wrong or that any fallback occurred.

---

## 7. What the Cache Actually Is, and What It Is Not

- The slug cache mechanism, as written in `gstack-slug`'s source, is real,
  functions exactly as documented, and is not bugged in any way found by
  this investigation.
- The cache entry for this project (`sassycomapp-project-library`) is a
  **real, correctly-computed value** — it is what the actual binary
  produces when genuinely invoked from this directory.
- **That cache has never been read by any skill invocation tested in this
  investigation**, because no tested skill invocation has ever
  successfully reached the point of calling the real binary at its real,
  working path. The agent's own reconstructed commands diverge before
  that point every time.
- Every previously observed slug value (`project-library`, `unknown`,
  and earlier `dev-p`) traces to improvisation and silent self-recovery,
  not to the cache, not to a missing binary, and not to an environment
  mismatch between the agent's shell and a manual terminal.

---

## 8. Status of Prior Documents

This finding **supersedes** the confirmed root cause in:
- `truth-statement-gstack-artifact-saving-inconsistency.md` §2
- `GStack Artifact Saving Inconsistency — Deep Technical Research Findings.md` §1
- `GStack Artifact Saving Inconsistency — Corrected Deep Technical Research
  Report.md` §1–§2

All three concluded the hardcoded Claude-only path was broken/absent and
treated that as the actionable root cause. It is not absent, and it was
never the mechanism actually in play. The genuinely new, confirmed root
cause is the agent's own non-literal execution of `SKILL.md` instructions,
combined with undisclosed self-recovery on failure.

---

## 9. Implication for Any Fix

Regenerating skills for the OpenCode host (Option C in the Corrected
Report) would not address this — it would still depend on the agent
executing whatever literal path ends up in the file, and the agent has now
been shown to substitute its own reconstructed command instead of doing so
reliably.

The more directly targeted intervention is an explicit instruction telling
the agent not to perform its own host-detection guessing for these
binaries at all, and to use the one path already confirmed to work on this
machine:

```
## GStack binary path override
When any GStack skill needs to call gstack-slug, gstack-config, or any
other gstack-* binary, always use /mnt/c/mybizz/gstack/bin/<binary-name>
directly. Do not attempt host-detection logic (~/.codex/skills/gstack,
.agents/skills/gstack, or similar) — that detection has been confirmed to
fail in this environment. The path above is correct and verified.
```

This has not yet been tested. It is the natural next step.
