# Matt Pocock Skills Reference

*Last confirmed: 2026-07-13, directly against a real `npx skills@latest add mattpocock/skills --agent opencode --yes` install log on your machine. This supersedes all prior web-research-based versions of this document — an actual install log is stronger evidence than anything fetched from GitHub, since it reflects what's really on your disk right now, not what a README claimed at some point.*

## ✅ RESOLVED: the `qa` situation was misdiagnosed — no overwrite occurred

Correction to the earlier version of this section: GStack does not install through the generic `skills` CLI at all — it has its own installer, and for OpenCode it symlinks each individual skill (including `qa`) as a flat top-level entry directly into `~/.config/opencode/skills/`, pointing at the real source on `/mnt/c/mybizz/skills/`. **Confirmed directly: `~/.config/opencode/skills/qa -> /mnt/c/mybizz/skills/qa/` is intact and was never touched.**

The actual situation: two separate directories both contain something named `qa` — GStack's real one (untouched, at `~/.config/opencode/skills/qa`) and Matt's stray conversational bug-reporting one (at `~/.agents/skills/qa`, installed as part of the 39-skill "General" bundle, never intended for use). No repair needed on GStack's side. The fix is just removing the stray:

```bash
npx skills remove qa
```

This only touches `~/.agents/skills/qa` (the generic tool's own managed directory) — leave `~/.config/opencode/skills/` alone entirely.

**Same treatment recommended for `code-review`** — redundant with GStack's own `review` (confirmed also intact at `~/.config/opencode/skills/review`):

```bash
npx skills remove code-review
```

**Longer-term:** avoid reinstalling Matt's full 39-skill bundle again. Cherry-pick only what PDLF uses:
```bash
npx skills@latest add mattpocock/skills --skill domain-modeling --skill to-tickets --skill triage --skill improve-codebase-architecture --agent opencode --yes
```

**Confirmed by directly listing GStack's installed skill set:** the only exact-name collision between GStack's ~50 skills and Matt's 39 is `qa`. Everything else — including `to-spec` vs. GStack's own `spec`, a different slug — is fine as-is.

**Bonus findings from this same listing, worth knowing about before writing more custom tooling:** GStack already ships `codex` ("multi-AI second opinion") and three named safety skills (`careful`, `freeze`, `guard`) — both directly relevant to earlier discussion in this conversation about gate-audit mechanisms and AI-runaway prevention. Worth looking at before building anything bespoke for either.

---

## Confirmed installed inventory (39 skills total)

**"Mattpocock Skills" category (21) — the skills the vendor itself brands as core:**

| Skill | Status vs. PDLF-New-Project-Mapping.md |
|---|---|
| `domain-modeling` | ✅ **Used** — Step 16 |
| `to-tickets` | ✅ **Used** — Step 19. Confirms the `to-issues`→`to-tickets` rename researched earlier was correct and is now live on your disk. |
| `triage` | ✅ **Used** — Step 39 |
| `improve-codebase-architecture` | ✅ **Used** — Step 40 |
| `to-spec` | ⛔ Present on disk, **deliberately not used**. Confirmed no downstream consumer in the pipeline — this was the correct call, and remains correct. Because it's now installed, double-check it isn't model-invoked into firing unprompted somewhere between Steps 12–18. |
| `prototype` | ⛔ Present on disk, **deliberately not used**. Was removed from Step 25 as optional/non-chained. Same watch-item as `to-spec` — it's now confirmed model-invoked in general (per earlier v1.1.0 research), so confirm it isn't firing where you don't want it. |
| `code-review` | Not currently assigned in PDLF. GStack's `/review` covers Step 37. See collision note above. |
| `ask-matt` | Not currently assigned. Router/entry-point skill — informational value only inside a PDLF pipeline that already dictates its own sequencing. |
| `codebase-design` | Not currently assigned. Reference vocabulary (Module/Interface/Depth/Seam), overlaps conceptually with `improve-codebase-architecture`. |
| `diagnosing-bugs` | Not currently assigned in the mapping, but real and worth considering for ad hoc debugging work outside the formal pipeline. Confirms this name is still current — it was NOT renamed to `diagnose` as one transient web search suggested during earlier research; that appears to have been a stale/incorrect signal. |
| `wayfinder` | Not currently assigned. Genuinely relevant to your **existing-project entry branch** discussion — its whole purpose is planning work too big for one session via a decision map, which has real conceptual overlap with what `step-03-resume-point-audit.md` is doing. Worth a deliberate look before writing that file, not necessarily to use it, but to check whether it does something your custom SOP doesn't. |
| `implement` | Not currently assigned. Per earlier research this is the build-phase umbrella in Matt's own `ask-matt` routing (`idea → to-spec → to-tickets → implement`). PDLF's Step 35 (build loop) is handled natively by OpenCode per the skill-use principle — confirm `implement` doesn't add anything OpenCode's own build loop lacks before considering it. |
| `research` | Not currently assigned. Background-agent, cited-source investigation. Could be a legitimate fit for the "genuinely open questions" that came up repeatedly this session (e.g., whether `/review`/`/qa` persist findings) — worth keeping in mind as a tool for future open questions, not a pipeline step. |
| `tdd` | Not currently assigned as a named pipeline step — Step 35's three-level testing methodology is PDLF's own, older, and already validated against mb-3-cs. Confirm you don't want the two colliding conceptually. |
| `setup-matt-pocock-skills` | Already run once, per your setup — confirmed present for future re-runs if tracker/domain config changes. |
| `grill-with-docs`, `grill-me`, `grilling` | Not currently assigned as pipeline steps. GStack's `/office-hours` and `/plan-*-review` skills are doing PDLF's equivalent interrogation work. |
| `handoff` | Not currently assigned. Worth knowing this exists as a lighter-weight alternative to the custom handoff documents you've been having me write by hand this session. |
| `teach` | Not relevant to PDLF's pipeline. |
| `writing-great-skills` | Not a pipeline step — this is the meta-skill governing how *we* write the stepwise files. Directly relevant to the skill-writing session you're about to start. |

**"General" category (18) — not Matt Pocock's core engineering skillset; a broader bundle that came with the same install:**

| Skill | Relevance to PDLF |
|---|---|
| `qa` | **Naming collision with GStack's own `qa` — GStack's copy confirmed untouched; remove Matt's stray copy. See resolved section at top of document.** |
| `ubiquitous-language` | Conceptually adjacent to `domain-modeling` — don't confuse the two if either ever gets invoked; PDLF uses `domain-modeling` only. |
| `request-refactor-plan` | Conceptually adjacent to `improve-codebase-architecture` — same caution. |
| `resolving-merge-conflicts` | Not currently relevant — no PDLF step involves manual merge conflict resolution as a named procedure. |
| `design-an-interface`, `setup-ts-deep-modules` | TypeScript/interface-design oriented — limited relevance to an Anvil/Python project. |
| `claude-handoff`, `loop-me`, `wizard` | Generic utility skills, no current PDLF assignment. |
| `writing-beats`, `writing-fragments`, `writing-shape`, `edit-article`, `obsidian-vault` | Creative writing / note-taking skills — not software engineering, not relevant to PDLF at all. Their presence tells you this install pulled in the vendor's *entire* multi-purpose skill catalog, not a curated engineering subset. |
| `git-guardrails-claude-code` | Blocks dangerous git commands. Worth considering independent of PDLF, as a safety net given how much git automation the pipeline does — genuinely useful, low-risk to keep. |
| `migrate-to-shoehorn`, `scaffold-exercises`, `setup-pre-commit` | Narrow, tool-specific utilities. `setup-pre-commit` (Husky/lint-staged/Prettier/type-check/tests) may be worth a look given the earlier finding that no CI infrastructure was confirmed for Steps 35A/38A — but it's a Node.js-ecosystem tool, so its fit for a Python/Anvil project needs checking before assuming it applies. |

---

## What this changes for PDLF's skill selection: **nothing in the mapping itself, one real action item**

The four skills PDLF actually uses (`domain-modeling`, `to-tickets`, `triage`, `improve-codebase-architecture`) are all confirmed present, correctly named, matching the mapping exactly. **No change to `PDLF-New-Project-Mapping.md` is needed.**

The one concrete action: **resolve the `qa` collision before Step 38 is ever actually run**, per the instructions at the top of this document. Everything else above is context and future-reference, not an immediate blocker.

---

## The bigger pattern: this repo ships like a live product

Confirmed again by this install: 39 skills, spanning categories ("Mattpocock Skills" vs. "General") that didn't exist in earlier research passes on this same repo. Expect this to keep growing and shifting. The install log is now your best source of truth — better than anything web-researched, including the sections below, which remain useful for *behavioral* detail (what a skill does, how it's invoked) that a bare install log doesn't show.

```bash
ls ~/.agents/skills/
cat ~/.agents/.skill-lock.json
```

Re-run and re-diff after every update.

**Source:** `github.com/mattpocock/skills` (MIT) | **Installed:** `~/.agents/skills/<name>/SKILL.md` | **Lock file:** `~/.agents/.skill-lock.json`

**Install/update:** `npx skills@latest add mattpocock/skills --agent opencode --yes`

---

## Behavioral detail on the skills PDLF actually uses

### `domain-modeling` (Step 16)

Active discipline: challenge terms against the glossary, sharpen fuzzy language, discuss concrete scenarios, cross-reference with code. `CONTEXT.md` updates inline the moment a term resolves — never batched. ADRs offered only when hard-to-reverse, surprising, and the result of a genuine trade-off. `CONTEXT.md` must stay "totally devoid of implementation details" — a glossary and nothing else.

### `to-tickets` (Step 19)

Breaks a plan/spec/conversation into tracer-bullet vertical-slice tickets, or — for a wide refactor with a large blast radius — an expand→migrate→contract sequence. Writes to whatever the project's `### Issue tracker` block in `CLAUDE.md`/`AGENTS.md` points at (per your setup, `.scratch/`) — never a hardcoded path.

### `triage` (Step 39)

Five-state role machine: `needs-triage` → `needs-info` / `ready-for-agent` / `ready-for-human` / `wontfix`. Every posted comment must open with the mandatory AI-disclaimer line. Processes whatever's actually in the tracker — has no opinion about how it got there, which is why Steps 37/38's confirmed artifact-persistence requirement matters so much for this step to have real work to do.

### `improve-codebase-architecture` (Step 40)

Read-only. Explores the codebase, writes an HTML report to your OS temp directory (never the repo), and only drops into a design conversation if you pick a candidate from the report. Does not touch code — any resulting change goes through Step 40B with your explicit approval first.

---

## Setup skill recap (`setup-matt-pocock-skills`)

Explores git remotes, existing `CLAUDE.md`/`AGENTS.md`, `CONTEXT.md`/`CONTEXT-MAP.md`, `docs/adr/`, existing `docs/agents/`. Asks three questions in sequence, not at once: issue tracker choice, triage label mapping, domain doc layout (single- vs. multi-context). Writes an `## Agent skills` block into whichever of `CLAUDE.md`/`AGENTS.md` exists (never creates a duplicate, never creates one if the other exists), plus `docs/agents/issue-tracker.md`, `triage-labels.md`, `domain.md`.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Matt's stray `qa`/`code-review` alongside GStack's real ones | Not an overwrite — GStack's copies are untouched at `~/.config/opencode/skills/`. Remove Matt's: `npx skills remove qa` and `npx skills remove code-review` |
| Skill not in picker | `ls ~/.agents/skills/<name>/SKILL.md` — name may have changed upstream again |
| `to-tickets` or `triage` can't find the tracker | Check `docs/agents/issue-tracker.md` exists and the `### Issue tracker` block is present in `CLAUDE.md`/`AGENTS.md` |
| `domain-modeling` seems to know nothing | `CONTEXT.md` missing/empty — expected until the first term resolves |
| `improve-codebase-architecture` seems to be editing code directly | It shouldn't — it's read-only by design. If you're seeing direct edits, something is invoking a different skill or acting outside the documented procedure; investigate before trusting the output. |
