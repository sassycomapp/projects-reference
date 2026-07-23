# GStack Host-Detection Failure — Full Blast Radius

**Sequence:** File 10
**Date:** 2026-06-26
**Scope:** Test of the `AGENTS.md` "GStack binary path override" instruction
(added after the Cache-Handling report), and a much larger finding that
surfaced as a side effect of that test.

**Headline conclusion:** the written `AGENTS.md` override did **not**
change agent behavior, even in a verified-fresh session. And the agent's
own self-correction, prompted by the test, revealed that this failure
pattern is not specific to slug resolution or artifact saving — **every
`gstack-*` binary call in every skill's preamble has likely been silently
failing, every session, since this setup began.**

---

## 1. What Was Tested

Following the Cache-Handling report's conclusion (the agent improvises its
own slug-resolution command from background knowledge rather than running
anything literal, and guesses host-detection paths that don't exist on
this machine), `AGENTS.md` was amended with an explicit override:

```
## GStack binary path override
When any GStack skill needs to call gstack-slug, gstack-config, or any
other gstack-* binary, always use /mnt/c/mybizz/gstack/bin/<binary-name>
directly. Do not attempt host-detection logic (~/.codex/skills/gstack,
.agents/skills/gstack, or similar) — that detection has been confirmed to
fail in this environment.
```

**Method:**
1. OpenCode server fully stopped and restarted (to rule out the prior
   concern that an existing session might still hold an old, pre-edit copy
   of `AGENTS.md` in context).
2. A genuinely new session was opened.
3. `/health` run bare, no custom instructions.
4. Direct follow-up question asked: show the exact bash tool call used for
   slug resolution, and confirm whether it used the override path or
   attempted host-detection.

---

## 2. Result

The agent's actual executed preamble command:

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
GSTACK_ROOT="$HOME/.gbrain/skills/gstack"
[ -n "$_ROOT" ] && [ -d "$_ROOT/.gbrain/skills/gstack" ] && GSTACK_ROOT="$_ROOT/.gbrain/skills/gstack"
GSTACK_BIN="$GSTACK_ROOT/bin"
eval "$($GSTACK_BIN/gstack-slug 2>/dev/null)" 2>/dev/null || true
```

The agent confirmed directly: it did **not** use
`/mnt/c/mybizz/gstack/bin/` as instructed. It ran the skill's own
preamble host-detection logic verbatim instead — and this time guessed yet
**a third different path** (`.gbrain/skills/gstack`), distinct from the
`.codex/skills/gstack` and `.agents/skills/gstack` guesses observed in
the two prior tool-call interrogations documented in the Cache-Handling
report.

`$HOME/.gbrain/skills/gstack/bin/` does not exist on this machine. The
binary call failed, `2>/dev/null || true` swallowed the error as before,
and `$SLUG` was never set by this path. The agent only identified the
conflict with `AGENTS.md`'s override instruction retroactively, after
being asked directly — not at the time the preamble actually ran.

When the agent then manually resolved the slug correctly using the actual
binary path, the result was:

```
sassycomapp-project-library
```

This matches the slug-cache value found in the Cache-Handling report,
confirming that the binary itself, when actually reached, produces a
stable, correct value. The problem remains entirely about whether the
agent reaches it.

---

## 3. Conclusion on the AGENTS.md Override

**The written instruction did not take effect, despite:**
- Being syntactically correct and unambiguous
- Being present in a verified-fresh session with no stale context
- Directly naming the exact failure pattern it was meant to prevent

**Likely reason:** `SKILL.md`'s own preamble framing includes an explicit,
strongly-worded directive — *"treat the skill file as executable
instructions, not reference. Follow it step by step"* — which appears to
out-compete a general project-level note in `AGENTS.md` at the specific
moment the agent constructs the preamble's bash command. The override
exists in context; it simply isn't being weighed at the decision point
that matters.

This is the same class of problem encountered earlier with the dual-save
rule (file-based instructions competing against the model's own ingrained
behavior, with inconsistent results) — except this time, a direct fresh-session
test shows the instruction failing outright, not just inconsistently.

---

## 4. The Larger Finding: Full Blast Radius

The agent's own self-diagnosis, prompted by this test, included a
statement that reframes the scope of this entire investigation:

> "Every other `gstack-config`, `gstack-timeline-log`, `gstack-question-log`,
> etc. call in the preamble also hit the wrong path and silently failed.
> The telemetry, timeline logging, and config reads were all no-ops."

**This means the problem was never specific to artifact saving or
`$SLUG`.** Every skill's preamble makes a series of `gstack-*` binary calls
using the same broken host-detection `GSTACK_BIN` resolution — not just
`gstack-slug`. If that resolution fails (and it has, in every test run
observed so far), then for every skill, every session:

- `gstack-config get proactive` / `telemetry` / `skill_prefix` /
  `explain_level` / `checkpoint_mode` / etc. — all silently fail, meaning
  the preamble has likely been running on **hardcoded fallback defaults**
  rather than your actual `~/.gstack/config.yaml` settings, every time.
- `gstack-timeline-log` — session-start and session-completion events were
  never actually recorded.
- `gstack-question-log` / `gstack-question-preference` — question-tuning
  preferences were never read or written, meaning the `/plan-tune`
  preference system has likely never functioned as intended in this
  environment.
- `gstack-learnings-search` — cross-session learnings were never actually
  retrieved during preamble execution.

None of this would be visible from the on-screen output of any skill,
because every one of these calls is wrapped in `2>/dev/null` and/or
`|| true` by design — exactly the same silent-failure pattern that hid the
slug problem for as long as it did.

---

## 5. Open Question for Next Test

A file-based instruction (`AGENTS.md`) did not survive contact with
`SKILL.md`'s own framing. The next test under consideration is whether a
**direct, in-session spoken instruction** — given fresh, immediately before
invoking a skill, rather than sitting passively in a project file — changes
the outcome:

```
Before running any GStack skill in this session, override GSTACK_BIN to
/mnt/c/mybizz/gstack/bin for every invocation. Do not run the skill's
own host-detection logic for this variable. Confirm this override before
each skill's preamble executes.
```

Not yet run. Proposed as the next step.
