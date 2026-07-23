# plan-ceo-review — Skill Spec

**Source:** `/mnt/c/mybizz/skills/plan-ceo-review/SKILL.md`
**Total lines read:** 1434 (lines 1–1434, confirmed complete — file ends at line 1434 with no truncation)
**Date:** 2026-06-26

---

## Q1. Which reports or artifacts does this skill read or depend on?

The skill reads the following artifacts during execution:

**From `~/.gstack/projects/$SLUG/` (gbrain context queries, lines 23–43):**
- Prior CEO plans: `~/.gstack/projects/{repo_slug}/ceo-plans/*.md` (line 25, gbrain query `prior-ceo-plans`)
- Recent design docs: `~/.gstack/projects/{repo_slug}/*-design-*.md` (line 31, gbrain query `recent-design-docs`)
- Recent CEO review activity: timeline entries tagged with `repo:{repo_slug}` containing `plan-ceo-review` (lines 36–43, gbrain query `recent-reviews`)

**From `~/.gstack/projects/$SLUG/` (runtime lookups during skill body):**
- Design doc (from `/office-hours`): lines 910–912 — looks for `~/.gstack/projects/$SLUG/*-$BRANCH-design-*.md`, falls back to `~/.gstack/projects/$SLUG/*-design-*.md`
- Handoff note (from prior CEO review): lines 919–920 — looks for `~/.gstack/projects/$SLUG/*-$BRANCH-ceo-handoff-*.md`
- Checkpoints: line 614 — `find "$_PROJ/ceo-plans" "$_PROJ/checkpoints" -type f -name "*.md"`
- Reviews JSONL: line 615 — `$_PROJ/${_BRANCH}-reviews.jsonl`
- Timeline: line 616 — `$_PROJ/timeline.jsonl`
- Decisions: line 625 — `$_PROJ/decisions.active.json`
- Learnings: line 110 — `${GSTACK_HOME:-$HOME/.gstack}/projects/${SLUG:-unknown}/learnings.jsonl`

**From the repo root (system audit, lines 896–903):**
- `git log --oneline -30`
- `git diff <base> --stat`
- `git stash list`
- `grep -r "TODO|FIXME|HACK|XXX"` across the repo
- Recently touched files (30-day git log)
- `CLAUDE.md` (line 903)
- `TODOS.md` (line 903)
- Any existing architecture docs (line 903)

**From the gstack install directory:**
- `~/.claude/skills/gstack/scripts/jargon-list.json` (line 649)
- `~/.claude/skills/gstack/ETHOS.md` (line 727, line 1049)
- `~/.claude/skills/gstack/office-hours/SKILL.md` (lines 956, 1002 — loaded inline when prerequisite skill is offered)
- `~/.claude/skills/gstack/bin/gstack-brain-cache` — product, goals, recent-decisions, user-profile digests (lines 1116–1122)
- `sections/review-sections.md` (line 1146, line 1393 — **mandatory read before the 11-section deep review**)

---

## Q2. Where exactly will it look for those artifacts (literal path, quoted)?

| Artifact | Literal path/pattern | Line(s) |
|---|---|---|
| Prior CEO plans | `~/.gstack/projects/{repo_slug}/ceo-plans/*.md` | 25 |
| Recent design docs | `~/.gstack/projects/{repo_slug}/*-design-*.md` | 31 |
| Design doc (branch-specific) | `~/.gstack/projects/$SLUG/*-$BRANCH-design-*.md` | 910 |
| Design doc (branch-agnostic fallback) | `~/.gstack/projects/$SLUG/*-design-*.md` | 911 |
| Handoff note | `~/.gstack/projects/$SLUG/*-$BRANCH-ceo-handoff-*.md` | 919 |
| Checkpoints | `$_PROJ/ceo-plans`, `$_PROJ/checkpoints` | 614 |
| Reviews JSONL | `$_PROJ/${_BRANCH}-reviews.jsonl` | 615 |
| Timeline | `$_PROJ/timeline.jsonl` | 616 |
| Decisions | `$_PROJ/decisions.active.json` | 625 |
| Learnings | `${GSTACK_HOME:-$HOME/.gstack}/projects/${SLUG:-unknown}/learnings.jsonl` | 110 |
| Brain cache digests | `~/.claude/skills/gstack/bin/gstack-brain-cache get {product,goals,recent-decisions,user-profile}` | 1116–1122 |
| Office-hours skill | `~/.claude/skills/gstack/office-hours/SKILL.md` | 956, 1002 |
| Review sections | `~/.claude/skills/gstack/plan-ceo-review/sections/review-sections.md` | 1146, 1393 |
| Jargon list | `~/.claude/skills/gstack/scripts/jargon-list.json` | 649 |
| ETHOS | `~/.claude/skills/gstack/ETHOS.md` | 727 |

Where `$_PROJ` = `${GSTACK_HOME:-$HOME/.gstack}/projects/${SLUG:-unknown}` (line 611).

---

## Q3. Where does this skill save its own output report (exact full literal path)?

**Primary deliverable — CEO plan (EXPANSION and SELECTIVE EXPANSION modes only):**

Literal path from line 1253:
```
~/.gstack/projects/$SLUG/ceo-plans/{date}-{feature-slug}.md
```
Where `{date}` = `YYYY-MM-DD` format (line 1285) and `{feature-slug}` is derived from the plan being reviewed (line 1285).

The directory is created at line 1243:
```
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG/ceo-plans
```

**Stale plan archive directory (line 1249–1250):**
```
~/.gstack/projects/$SLUG/ceo-plans/archive/
```

**Spec review metrics (line 1347):**
```
~/.gstack/analytics/spec-review.jsonl
```

**Review report (written into the plan file itself, not a separate file):**

Lines 784–786 (Plan Status Footer):
> Skills that run plan reviews (`/plan-*-review`, `/codex review`) include the EXIT PLAN MODE GATE blocking checklist at the end of the skill, which verifies the plan file ends with `## GSTACK REVIEW REPORT` before ExitPlanMode is called.

Lines 1406–1434 (EXIT PLAN MODE GATE): The review report is written as a `## GSTACK REVIEW REPORT` section into whatever plan file is currently being reviewed. The file path is not specified in this skill — it depends on which plan file the user has open in plan mode.

**Summary of all save locations:**

| Output | Path | Condition |
|---|---|---|
| CEO plan | `~/.gstack/projects/$SLUG/ceo-plans/{date}-{feature-slug}.md` | EXPANSION or SELECTIVE EXPANSION mode only (line 1240) |
| Stale plan archive | `~/.gstack/projects/$SLUG/ceo-plans/archive/` | Only if stale plans exist (line 1249) |
| Spec review metrics | `~/.gstack/analytics/spec-review.jsonl` | Always (line 1347) |
| Review report | `## GSTACK REVIEW REPORT` section in the plan file | Always (lines 1406–1434) |

---

## Q4. Will this skill save its output in plan mode?

**Yes — for `~/.gstack/` writes.** The generic "Plan Mode Safe Operations" boilerplate (line 156–158) states:

> In plan mode, allowed because they inform the plan: `$B`, `$D`, `codex exec`/`codex review`, writes to `~/.gstack/`, writes to the plan file, and `open` for generated artifacts.

This covers the CEO plan save (`~/.gstack/projects/$SLUG/ceo-plans/...`) and the analytics writes (`~/.gstack/analytics/spec-review.jsonl`).

The telemetry section has an explicit plan-mode exception (lines 759–760):

> **PLAN MODE EXCEPTION — ALWAYS RUN:** This command writes telemetry to `~/.gstack/analytics/`, matching preamble analytics writes.

The review report write (into the plan file) is explicitly allowed by line 786:

> Writing the plan file is the one edit allowed in plan mode.

**There is no skill-specific plan-mode text near the CEO plan save step (line 1253).** The save relies entirely on the generic blanket rule that `~/.gstack/` writes are allowed.

**Important caveat:** The CEO plan is only persisted for EXPANSION and SELECTIVE EXPANSION modes (line 1240). HOLD SCOPE and SCOPE REDUCTION modes do NOT produce a CEO plan file. The review report (written to the plan file) is the only output for those modes.

---

## Q5. Does the skill ask for save confirmation before writing?

**No.** The CEO plan write at line 1253 is presented as an instruction to the agent ("Write to `~/.gstack/projects/$SLUG/ceo-plans/{date}-{feature-slug}.md` using this format:") with no AskUserQuestion gate preceding it. The user has already approved scope decisions via the opt-in/cherry-pick ceremony in Step 0D, but the file save itself is not separately confirmed.

The spec review loop (lines 1289–1349) also writes to disk without a confirmation prompt — it writes first, then reviews, then fixes in-place.

No text in the file asks "save this?" before the CEO plan write. The user can decline scope expansions earlier (Step 0D), which would affect what goes into the plan, but cannot block the save itself.

---

## Q6. Does this skill have more than one save mode?

**Yes — three distinct save paths, two of which are conditional on mode:**

| Mode | Save path | Condition |
|---|---|---|
| CEO plan (primary) | `~/.gstack/projects/$SLUG/ceo-plans/{date}-{feature-slug}.md` | Only for EXPANSION and SELECTIVE EXPANSION (line 1240) |
| Spec review metrics | `~/.gstack/analytics/spec-review.jsonl` | Always (line 1347) |
| Review report | Plan file (whatever the user is reviewing) | Always (lines 1406–1434) |

HOLD SCOPE and SCOPE REDUCTION modes produce no CEO plan file. They only produce the review report in the plan file and the spec review metrics.

Additionally, the common preamble writes telemetry to `~/.gstack/analytics/skill-usage.jsonl` (line 772) regardless of mode.

---

## Q7. Does the save path depend on $SLUG?

**Yes.** The CEO plan save path (line 1253) is:
```
~/.gstack/projects/$SLUG/ceo-plans/{date}-{feature-slug}.md
```

$SLUG is computed by the `gstack-slug` binary at line 1243:
```
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" && mkdir -p ~/.gstack/projects/$SLUG/ceo-plans
```

The design doc lookup (lines 910–911) and handoff note lookup (line 919) also depend on $SLUG.

---

## Q8. Is this skill's output read as a dependency by any other skill?

**Yes — referenced in this file's gbrain context queries (lines 23–43):**

The `prior-ceo-plans` gbrain query (lines 23–28) reads:
```yaml
- id: prior-ceo-plans
  kind: filesystem
  glob: "~/.gstack/projects/{repo_slug}/ceo-plans/*.md"
  sort: mtime_desc
  limit: 5
  render_as: "## Prior CEO plans for this project"
```

This means the skill reads its own prior outputs on subsequent runs. Other skills that share the same gbrain context pattern (e.g., `/autoplan`, which runs `/plan-ceo-review` as a sub-skill) would also see these files.

**Not stated in this file:** Whether `/design-consultation`, `/design-html`, or `/design-shotgun` read CEO plan files. The gbrain queries for those skills (visible in their frontmatter) reference `ceo-plans` globs, suggesting cross-skill dependency, but that is stated in their files, not this one.

**The gstack-reference.md (line 110) confirms:**
> `/plan-ceo-review` → `~/.gstack/projects/$SLUG/ceo-plans/`

---

## Q9. Does this project's AGENTS.md or CLAUDE.md contain anything that could override, redirect, or duplicate this skill's save behavior?

**Yes — AGENTS.md adds a dual-save requirement.**

AGENTS.md lines 39–51:

> When GStack or OpenCode produces output artefacts during a session:
> - For ANY GStack or OpenCode skill that produces a report, plan, review, or artifact
>   during a session → ALWAYS complete the skill's own native default save (to its
>   standard ~/.gstack/projects/ location) without asking for confirmation. ADDITIONALLY
>   save a copy to `gstack-outputs/` for human reference. Never skip the native save,
>   and never ask whether to save — always save both.
>
>   After saving, the on-screen response must confirm both save locations explicitly,
>   e.g.: "Saved to: ~/.gstack/projects/<slug>/<path> AND gstack-outputs/<filename>."

This means the AGENTS.md requires an **additional** save to `gstack-outputs/` that the SKILL.md does not mention. The skill's native save to `~/.gstack/projects/$SLUG/ceo-plans/` is correct per the SKILL.md, but the AGENTS.md demands a duplicate copy in `gstack-outputs/`.

The gstack-reference.md (lines 118–123) further confirms:
> Skills don't do this automatically — you copy artifacts manually.

And line 123: `CEO review reports → gstack-outputs/`

**CLAUDE.md** (lines 33–50) contains skill routing rules but no save-path overrides.

---

## Q10. Does this file's actual content agree with gstack-skills-reference.md's description?

**No — they disagree on what gets saved.**

**gstack-skills-reference.md (lines 261–265):**
> ### /plan-ceo-review
> - **Usage:** CEO/founder-mode plan review. Rethinks the problem, finds the 10-star product, challenges premises. Four modes: Scope Expansion, Selective Expansion, Hold Scope, Scope Reduction.
> - **Prerequisites:** A plan or proposal to review.
> - **Output:** Review findings with scope expansion opportunities, risk analysis, edge cases, diagrams.
> - **Presentation:** Interactive review via AskUserQuestion. Review report written to plan file.

**Actual SKILL.md content:**

The reference doc omits several concrete disk writes that the SKILL.md specifies:
1. **CEO plan file** — `~/.gstack/projects/$SLUG/ceo-plans/{date}-{feature-slug}.md` (line 1253) — not mentioned in the reference
2. **Spec review metrics** — `~/.gstack/analytics/spec-review.jsonl` (line 1347) — not mentioned in the reference
3. **Stale plan archival** — `~/.gstack/projects/$SLUG/ceo-plans/archive/` (line 1249) — not mentioned in the reference
4. **Spec review loop with subagent** — the entire section at lines 1289–1349 — not mentioned in the reference
5. **The `gstack-outputs/` copy requirement from AGENTS.md** — not reflected in the reference doc's Output line

The reference doc says "Review report written to plan file" which is accurate for the `## GSTACK REVIEW REPORT` section. But it omits the CEO plan file, which is a separate, persistent deliverable.

---

## Q11. What side-effect writes happen regardless of the main output?

**Common preamble writes (present in every skill with the full preamble):**

| Write | Path | Line(s) |
|---|---|---|
| Session marker | `~/.gstack/sessions/"$PPID"` | 66 |
| Telemetry (skill usage) | `~/.gstack/analytics/skill-usage.jsonl` | 98, 772 |
| Pending file cleanup | `~/.gstack/analytics/.pending-"$_SESSION_ID"` (rm) | 767 |
| Timeline event (started) | via `gstack-timeline-log` → `~/.gstack/projects/$SLUG/timeline.jsonl` | 120 |
| Timeline event (completed) | via `gstack-timeline-log` → `~/.gstack/projects/$SLUG/timeline.jsonl` | 769 |
| Question log | via `gstack-question-log` → `~/.gstack/projects/$SLUG/${_BRANCH}-reviews.jsonl` | 703 |
| Question preference | via `gstack-question-preference --write` | 712 |
| Learnings log | via `gstack-learnings-log` → `~/.gstack/projects/$SLUG/learnings.jsonl` | 750 |
| Eureka log | `~/.gstack/analytics/eureka.jsonl` | 732 |
| Spec review metrics | `~/.gstack/analytics/spec-review.jsonl` | 1347 |
| Brain sync | `~/.claude/skills/gstack/bin/gstack-brain-sync` (once + discover-new) | 566–567 |

**Feature discovery / one-time markers (conditional):**

| Write | Path | Line(s) |
|---|---|---|
| Writing style prompted | `~/.gstack/.writing-style-prompted` | 192 |
| Writing style pending (rm) | `~/.gstack/.writing-style-prompt-pending` | 191 |
| Completeness intro seen | `~/.gstack/.completeness-intro-seen` | 201 |
| Telemetry prompted | `~/.gstack/.telemetry-prompted` | 229 |
| Proactive prompted | `~/.gstack/.proactive-prompted` | 247 |
| Vendoring warned | `~/.gstack/.vendoring-warned-${SLUG:-unknown}` | 314 |
| Feature prompted markers | `~/.claude/skills/gstack/.feature-prompted-continuous-checkpoint`, `~/.claude/skills/gstack/.feature-prompted-model-overlay` | 173–174 |

**Brain context preflight (transient, line 1123–1125):**

| Write | Path | Line(s) |
|---|---|---|
| Brain context temp file | `/tmp/.gstack-brain-context-$$.md` (written then immediately deleted) | 1123, 1125 |

**Config writes via `gstack-config set` (conditional on user choices):**

| Write | Path | Condition |
|---|---|---|
| Telemetry setting | `~/.gstack/config.yaml` | User accepts telemetry (lines 214–225) |
| Proactive setting | `~/.gstack/config.yaml` | User chooses proactive on/off (lines 242–243) |
| Routing declined | `~/.gstack/config.yaml` | User declines routing (line 289) |
| Explain level | `~/.gstack/config.yaml` | User chooses terse (line 187) |
| Cross-project learnings | `~/.gstack/config.yaml` | User enables/disables (lines 1088–1089) |

**CLAUDE.md write (conditional, lines 263–287):**

| Write | Path | Condition |
|---|---|---|
| Skill routing rules | `CLAUDE.md` | User accepts (line 263), then `git add CLAUDE.md && git commit` (line 287) |

---

## Q12. If a dependency from Q1 is missing, what does the skill do?

The skill has explicit graceful degradation for several missing dependencies:

**Design doc missing (lines 932–987):**

The skill offers the `/office-hours` prerequisite skill via AskUserQuestion:
> "No design doc found for this branch. `/office-hours` produces a structured problem statement, premise challenge, and explored alternatives — it gives this review much sharper input to work with. Takes about 10 minutes."

Options:
- A) Run /office-hours now (we'll pick up the review right after)
- B) Skip — proceed with standard review

If skipped: "No worries — standard review. If you ever want sharper input, try /office-hours first next time." Then proceeds normally. (Line 948–949)

If `/office-hours` is run but produces no design doc: "If none was produced (user may have cancelled), proceed with standard review." (Line 987)

**Handoff note missing (lines 919–921):**

No error. The skill simply continues without prior session context:
```
[ -n "$HANDOFF" ] && echo "HANDOFF_FOUND: $HANDOFF" || echo "NO_HANDOFF"
```

**Office-hours skill file unreadable (lines 958, 1004):**

> "Could not load /office-hours — skipping." and continue.

**Brain cache digests missing (lines 1128–1133):**

> If a digest is `(no X digest available yet)`, treat that section as cold; ask the user.

**WebSearch unavailable (lines 1054):**

> "Search unavailable — proceeding with in-distribution knowledge only."

**Learnings file missing (line 117–118):**

```
echo "LEARNINGS: 0"
```

**Spec review subagent fails (lines 1328–1330):**

> "Spec review unavailable — presenting unreviewed doc." The document is already written to disk; the review is a quality bonus, not a gate.
