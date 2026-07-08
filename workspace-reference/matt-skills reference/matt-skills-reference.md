# Matt Pocock Skills Reference

## What are Matt Pocock Skills?

Engineering and productivity skills for real software development with AI agents. They enforce domain language, architecture discipline, test-first development, and controlled handoffs. Not vibe coding.

**Source:** `github.com/mattpocock/skills` (MIT) | **Installed:** `~/.agents/skills/<name>/SKILL.md` | **Lock file:** `~/.agents/.skill-lock.json`

**Install/update:** `npx skills@latest add mattpocock/skills`

---

## Skills Catalog (18 installed)

### Engineering — User-invoked (type the command)

| Skill | What it does |
|---|---|
| `/ask-matt` | Router — which skill fits your situation |
| `/grill-with-docs` | Relentless interview + builds domain glossary + ADRs |
| `/triage` | Move issues through state machine (needs-triage → closed) |
| `/improve-codebase-architecture` | Codebase scan + visual HTML report + grill on chosen candidate |
| `/setup-matt-pocock-skills` | Per-repo config scaffold — run once per repo |
| `/to-prd` | Turn conversation into PRD, publish to issue tracker |
| `/to-issues` | Break PRD/spec into independent issues using tracer-bullet slices |
| `/prototype` | Build throwaway prototype to answer a design question |

### Engineering — Model-invoked (agent uses automatically)

| Skill | What it does |
|---|---|
| `/domain-modeling` | Build/sharpen domain glossary + ADRs (called by grill-with-docs) |
| `/tdd` | Red-green-refactor test-driven development |
| `/diagnosing-bugs` | Reproduce → minimise → hypothesise → instrument → fix → regression test |
| `/codebase-design` | Deep module design vocabulary and patterns |

### Productivity — User-invoked

| Skill | What it does |
|---|---|
| `/grill-me` | Relentless interview about a plan (no docs created) |
| `/handoff` | Compact conversation into handoff document for another agent |
| `/teach` | Multi-session teaching workspace with lessons, references, learning records |
| `/writing-great-skills` | Reference for writing and editing skills well |

### Productivity — Model-invoked

| Skill | What it does |
|---|---|
| `/grilling` | The interview loop behind grill-me and grill-with-docs |

### Utility

| Skill | What it does |
|---|---|
| `/find-skills` | Discover and install new skills (from vercel-labs/skills) |

---

## Invocation Rules

- **User-invoked** — only reachable when you type them (e.g. `/grill-me`). They orchestrate.
- **Model-invoked** — reachable when typed OR automatically by the agent when the task fits. They hold reusable discipline.
- A user-invoked skill may call model-invoked skills, but never another user-invoked one.

---

## Where Skills Write Output

### In the project (default behavior)

| Skill | Writes to | When |
|---|---|---|
| `/domain-modeling` | `CONTEXT.md` (glossary) + `adr/` (ADRs) | Creates lazily when terms crystallize |
| `/grill-with-docs` | Same as domain-modeling (calls it internally) | During interview |
| `/to-prd` | `.scratch/<feature>/<prd-slug>.md` | When publishing PRD |
| `/to-issues` | `.scratch/<feature>/<issue-slug>.md` | When breaking into tasks |
| `/triage` | `.scratch/` (updates issue files in-place) | When processing issues |
| `/triage` (rejected) | `.out-of-scope/<topic>.md` | When rejecting issues |
| `/setup-matt-pocock-skills` | `docs/agents/` (3 config files) + `AGENTS.md` | Once per repo |
| `/tdd`, `/diagnosing-bugs`, `/prototype` | In-place in codebase | During implementation |

### Outside the project

| Skill | Writes to | Why |
|---|---|---|
| `/handoff` | `%TEMP%` + copy to `matt-skills-output/` | Temp for agent pickup + permanent copy |
| `/improve-codebase-architecture` | `%TEMP%` + copy to `matt-skills-output/` | HTML report |
| `/teach` | `C:\mybizz\matt-skills-teach` | Shared teaching workspace across all projects |

### Nothing on disk

`/ask-matt`, `/grill-me`, `/grilling`, `/find-skills`, `/writing-great-skills`, `/codebase-design` — conversational or reference only.

---

## The Domain Model System

The most powerful technique in the toolkit. Creates a shared language between you and the agent.

**How it works:**
1. Run `/grill-with-docs` on a new feature
2. Agent asks detailed questions about your design
3. As you answer, it builds a glossary in `CONTEXT.md` and writes ADRs
4. Future sessions read this context — the agent already knows your jargon

**Example:**
- Before: "There's a problem when a lesson inside a section of a course is given a spot in the file system"
- After: "There's a problem with the materialization cascade"

This concision pays off every session. Variables, functions, and files get named consistently. The agent spends fewer tokens thinking because it has concise language.

---

## Workflows

### New Feature (full pipeline)
```
/grill-with-docs → /prototype → /to-prd → /to-issues → /tdd → /handoff
```

### Bug Fix
```
/triage → /diagnosing-bugs
```

### Architecture Health Check
```
/improve-codebase-architecture → /handoff
```

### Learning
```
/teach
```

---

## Per-Repo Setup

`/setup-matt-pocock-skills` creates three config files in `docs/agents/`:

| File | Purpose |
|---|---|
| `docs/agents/issue-tracker.md` | Where issues live (GitHub, GitLab, or local `.scratch/`) |
| `docs/agents/triage-labels.md` | Label vocabulary for `/triage` (5 canonical labels) |
| `docs/agents/domain.md` | How skills consume domain docs (CONTEXT.md, ADRs) |

**Current setup:** All projects use local markdown issue tracker (`.scratch/`) with default triage labels.

| Project | CONTEXT.md | docs/agents | .scratch | .out-of-scope | matt-skills-output |
|---|---|---|---|---|---|
| mb-3-cs | Bootstrapped | 3 files | Yes | Yes | Yes |
| mb4ecom | Lazy create | 3 files | Yes | Yes | Yes |
| mb5pdlf | Lazy create | 3 files | Yes | Yes | Yes |
| dev-pdlf | Lazy create | 3 files | Yes | Yes | Yes |

---

## Integration with GStack and PDLF

Matt Pocock skills complement GStack and PDLF — they don't replace them.

| Layer | Tool | Role |
|---|---|---|
| Orchestration | PDLF | Full project lifecycle, stage-gated |
| Workflow | GStack | Planning reviews, QA, review, ship |
| Engineering discipline | Matt Pocock | Domain modeling, TDD, architecture, debugging, handoffs |

PDLF integration points:
- PDLF Step 2 calls `/to-prd` to create the PRD
- PDLF Step 5 calls `/domain-modeling` to lock the domain model
- PDLF Step 7 calls `/to-issues` to break into tasks
- PDLF Step 16 uses `/tdd` for test-first builds
- PDLF Step 21 optionally calls `/improve-codebase-architecture`

---

## Artifact Governance (per AGENTS.md)

These overrides are applied in each project's AGENTS.md:

| Skill | Default destination | Override |
|---|---|---|
| `domain-modeling`, `grill-with-docs` (ADRs) | `docs/adr/` | → `adr/` |
| `domain-modeling`, `grill-with-docs` (context) | `CONTEXT.md` | No change |
| `to-prd`, `to-issues`, `triage` | `.scratch/` | No change |
| `handoff`, `improve-codebase-architecture` | `%TEMP%` only | + copy to `matt-skills-output/` |
| `teach` | Invoking directory | → `C:\mybizz\matt-skills-teach` |
| Others | — | Ask before saving |

---

## File Locations

| Item | Location |
|---|---|
| Skills (installed) | `~/.agents/skills/<name>/SKILL.md` |
| Lock file | `~/.agents/.skill-lock.json` |
| Teaching workspace | `C:\mybizz\matt-skills-teach` |
| Per-project config | `<project>/docs/agents/` |
| Per-project issues | `<project>/.scratch/` |
| Per-project glossary | `<project>/CONTEXT.md` |
| Per-project outputs | `<project>/matt-skills-output/` |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Skill not in picker | Verify `~/.agents/skills/<name>/SKILL.md` exists |
| Skill can't find issue tracker | Check `docs/agents/issue-tracker.md` exists |
| Agent doesn't know domain terms | `CONTEXT.md` missing or empty — run `/grill-with-docs` |
| `/triage` errors about .out-of-scope/ | Create directory: `mkdir .out-of-scope` |
| Handoff not saved to matt-skills-output/ | Check directory exists and AGENTS.md has governance rule |
| `/teach` writes to wrong directory | Check AGENTS.md redirects to `C:\mybizz\matt-skills-teach` |

---

## Workflow Pipelines

### New Feature (full pipeline)
```
/grill-with-docs → /prototype → /to-prd → /to-issues → /tdd → /handoff
```
Grill to align on design and build domain model, prototype to test the design, publish PRD, break into issues, build test-first, hand off.

### Bug Fix
```
/triage → /diagnosing-bugs
```
Process bug through triage state machine, then disciplined diagnosis loop.

### Architecture Health Check
```
/improve-codebase-architecture → /handoff
```
Scan codebase for deepening opportunities, generate HTML report, grill on chosen candidate, hand off for implementation.

### Domain Model Building
```
/grill-with-docs → /domain-modeling
```
Grill-with-docs calls domain-modeling internally. Run standalone `/domain-modeling` when you want to sharpen terms without a full grilling session.

### Issue Processing
```
/triage → /to-issues → /tdd
```
Triage incoming issues, break ready ones into tasks, build test-first.

### Learning
```
/teach
```
Creates a multi-session teaching workspace with lessons, references, and learning records.

### Quick Decision: Which Skill?

Not sure which skill to use? Start with:
```
/ask-matt
```
It routes you to the right skill based on your situation.
