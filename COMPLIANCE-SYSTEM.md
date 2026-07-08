# Compliance System — OpenCode + AGENTS.md

## Overview

This document describes the compliance system that governs agent behavior across all Anvil.works projects. The system has three layers: programmatic enforcement (deterministic), behavioral rules (probabilistic), and reference documents (on-demand).

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│              opencode.json (per project)          │
│  Layer 1: Programmatic enforcement (deterministic)│
│  - Permission denials (anvil.yaml, destructive    │
│    bash, Plan can't write)                        │
│  - Instructions array (loads README.md, indexes)  │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│              AGENTS.md (global + project)         │
│  Layer 2: Behavioral rules (probabilistic)        │
│  - Rules, definitions, precedence                 │
│  - Project-specific constraints                   │
│  - Loaded into LLM context at session start       │
└────────────────────┬────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────┐
│         Global reference documents (on-demand)    │
│  Layer 3: Binding documents (read when needed)    │
│  - ADRs, policies, specifications, SOPs           │
│  - INDEX.md files loaded into context             │
│  - Full documents read via Read tool on demand    │
└─────────────────────────────────────────────────┘
```

---

## Layer 1: Programmatic enforcement (opencode.json)

Rules enforced by the `permission` config in `opencode.json`. The agent cannot bypass these regardless of what AGENTS.md says.

### What's enforced

| Rule | opencode.json config | Enforcement |
|---|---|---|
| anvil.yaml protection | `"edit": {"**/anvil.yaml": "deny"}` | Hard block |
| Plan agent can't write | `"agent.plan.permission.edit": "deny"` | Hard block |
| No destructive rm | `"bash": {"rm -rf*": "deny"}` | Hard block |
| No git reset --hard | `"bash": {"git reset --hard*": "deny"}` | Hard block |
| No git push --force | `"bash": {"git push --force*": "deny"}` | Hard block |

### How it works

When the agent attempts a denied action, OpenCode blocks it before execution. The agent receives feedback explaining the denial. This is deterministic — not dependent on the LLM reading or following a rule.

### Limitations

Permissions can only block tool calls. They cannot enforce behavioral rules like "read the README" or "don't add scope" or "ask before batching."

---

## Layer 2: Behavioral rules (AGENTS.md)

Rules the agent reads at session start and is expected to follow. Not programmatically enforced.

### File locations

| File | Location | Scope |
|---|---|---|
| Global AGENTS.md | `~/.config/opencode/AGENTS.md` | All sessions, all projects |
| Project AGENTS.md | `{project-root}/AGENTS.md` | That project only |

### What's covered

- Plan/Build role definitions
- Session boundary rules (rule file re-read, mandatory README reading)
- Global behavioral principles (fact verification, default-to-ask, zero gate-jumping)
- Safety rules (no file operations without ask, no credential exposure)
- Project-specific rules (two-repo separation, GBrain discipline, pdlf workflow)
- Artifact governance (routing, promotion, confirmation)
- Task completion reporting

### Line count targets

All AGENTS.md files are kept under 200 lines to maximize adherence. Longer files reduce the probability that the agent follows any individual rule.

### Current line counts

| File | Lines |
|---|---|
| Global AGENTS.md | 132 |
| projects/AGENTS.md | 116 |
| mb-3-cs/project-library/AGENTS.md | 138 |
| mb4ecom-project-library/AGENTS.md | 119 |
| mb5pdlf-project-library/AGENTS.md | 122 |
| dev-pdlf/AGENTS.md | 133 |
| dev-project-template/AGENTS.md | 120 |

---

## Layer 3: Reference documents (on-demand)

Binding documents that the agent reads when a task requires them. Not loaded into every conversation — only when relevant.

### Global reference directories

| Directory | Contents | Index file | When to check |
|---|---|---|---|
| `project-library-global/adr-global/` | 40 architectural decisions | `INDEX.md` | Before architecture choices |
| `project-library-global/specifications-global/` | 37 specifications | `INDEX.md` | Before implementing features |
| `project-library-global/policy-global/` | Policies (work in progress) | `INDEX.md` | Before governance actions |
| `project-library-global/standard-operating-procedures-global/` | SOPs (work in progress) | `INDEX.md` | Before standard tasks |

### How it works

1. The `instructions` array in the global `opencode.json` loads the four `INDEX.md` files into context at session start. This gives the agent awareness of what documents exist.
2. The AGENTS.md "Global reference documents" section tells the agent when to check each directory.
3. When a task is relevant, the agent uses the Read tool to load the specific document it needs.

### Why not load all documents

Loading all 77+ global documents into every conversation would consume the entire context window. The index-on-demand pattern gives the agent awareness of what exists at minimal token cost (~200 tokens for all four indexes), then loads specific documents only when needed.

---

## How the layers interact

### Example: Agent starts a task in mb-3-cs

1. OpenCode loads `~/.config/opencode/AGENTS.md` (global) into context
2. OpenCode loads `mb-3-cs/project-library/AGENTS.md` (project) into context
3. OpenCode loads the four global INDEX.md files into context (via `instructions`)
4. OpenCode loads `mb-3-cs/project-library/README.md` into context (via `instructions`)
5. The agent reads the rules and indexes
6. If the task involves architecture, the agent reads the relevant ADR from `adr-global/`
7. If the task involves implementation, the agent reads the relevant spec from `specifications-global/`
8. If the agent tries to edit `anvil.yaml`, opencode.json blocks it (Layer 1)
9. If the agent tries to `rm -rf`, opencode.json blocks it (Layer 1)
10. If the agent tries to write (as Plan), opencode.json blocks it (Layer 1)
11. If the agent deviates from the task, AGENTS.md rules apply (Layer 2 — probabilistic)

### Example: Agent context compaction

When context compaction occurs:
1. The global AGENTS.md is re-read (loaded from disk)
2. The project AGENTS.md is re-read (loaded from disk)
3. The INDEX.md files are re-read (loaded via `instructions`)
4. The README.md is re-read (loaded via `instructions`)
5. The agent re-orientates to the rules and context

---

## What's NOT covered

The following cannot be enforced in OpenCode (no hooks system):

- "Read the README before every task" — probabilistic (AGENTS.md rule)
- "Don't add scope" — behavioral (AGENTS.md rule)
- "No time-based references" — content-level (AGENTS.md rule)
- "No hardcoded colors" — content-level (AGENTS.md rule)
- "Check ADRs before starting" — procedural (AGENTS.md rule + index awareness)
- "Ask before batching" — judgment (AGENTS.md rule)
- Post-compaction context re-injection — no hook available; relies on OpenCode's built-in re-read

---

## Files in this system

### Configuration files
- `~/.config/opencode/opencode.jsonc` — global config (permissions, instructions, agent restrictions)
- `{project}/opencode.json` — project config (project-specific permissions and instructions)

### Rule files
- `~/.config/opencode/AGENTS.md` — global behavioral rules
- `{project}/AGENTS.md` — project-specific behavioral rules

### Reference indexes
- `project-library-global/adr-global/INDEX.md`
- `project-library-global/specifications-global/INDEX.md`
- `project-library-global/policy-global/INDEX.md`
- `project-library-global/standard-operating-procedures-global/INDEX.md`

### Project README files
- `{project}/README.md` — project-specific context (loaded via `instructions`)

---

## Maintenance

- **AGENTS.md files:** Keep under 200 lines. Review when rules change.
- **opencode.json files:** Update when new enforceable rules are identified.
- **INDEX.md files:** Update when new documents are added to global reference directories.
- **README.md files:** Each project maintains its own. Content is the project owner's responsibility.
- **This document:** Update when the compliance system architecture changes.
