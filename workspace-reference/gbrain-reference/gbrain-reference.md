# GBrain Reference

## What is GBrain

GBrain is the advisory memory layer. It stores and searches project documents (ADRs, specs, docs) and code (Python, YAML) as searchable vector memories. GBrain does not override current repo files, ADRs, or OpenCode rules — files are always authoritative.

**Engine:** PGLite (local) | **Database:** `~/.gbrain/brain.pglite/` | **Schema pack:** gbrain-everything (code-aware)

---

## Command Reference

### Health & Status

| Command | Description |
|---|---|
| `gbrain doctor` | Full health check (score out of 100) |
| `gbrain doctor --fast` | Quick health check (skips DB probes) |
| `gbrain doctor --json --fast` | Machine-readable health output |
| `gbrain stats` | Page count, chunk count, embed coverage, source breakdown |
| `gbrain --version` | Show installed version |
| `gbrain sources list` | List all registered sources with paths and page counts |
| `gbrain sources list --json` | Machine-readable source list |

### Search & Query

| Command | Description |
|---|---|
| `gbrain search "<terms>"` | Keyword search (full-text) |
| `gbrain query "<question>"` | Hybrid search (vector + keyword + expansion) |
| `gbrain query "<q>" --source <id>` | Search scoped to one source |
| `gbrain query "<q>" --limit 10` | Limit results |
| `gbrain query "<q>" --lang typescript` | Filter by language |
| `gbrain query "<q>" --near-symbol <name>` | Anchor retrieval at a code symbol |
| `gbrain query "<q>" --adaptive-return` | Return tight result set for lookup questions |

### Code Analysis

| Command | Description |
|---|---|
| `gbrain code-def <symbol>` | Find where a symbol is defined |
| `gbrain code-refs <symbol>` | Find all references to a symbol |
| `gbrain code-callers <symbol>` | Find what calls a function |
| `gbrain code-callees <symbol>` | Find what a function calls |
| `gbrain code-blast <symbol>` | Full transitive caller tree (up to depth) |

### Sync & Import

| Command | Description |
|---|---|
| `gbrain sync --repo <path>` | Sync a repo's markdown files into gbrain |
| `gbrain sync --repo <path> --strategy code` | Sync code files (.py, .ts, .js, .yaml) |
| `gbrain sync --all` | Sync all registered sources |
| `gbrain sync --all --parallel 4` | Sync all sources in parallel |
| `gbrain sync --break-lock` | Break a stale sync lock |
| `gbrain sync --no-hard-deadline` | Disable watchdog timeout (for large syncs) |
| `gbrain sync --dry-run` | Preview what would sync, no writes |
| `gbrain embed --stale` | Embed chunks that are missing embeddings |
| `gbrain extract --stale` | Extract links and timelines from stale pages |

### Dream Cycle (Call Graph & Intelligence)

| Command | Description |
|---|---|
| `gbrain dream --source <id>` | Run full dream cycle on a source |
| `gbrain dream --source <id> --no-hard-deadline` | Dream without watchdog timeout |
| `gbrain dream --source <id> --phase extract_atoms --drain` | Run only the extract_atoms phase |
| `gbrain dream --source <id> --phase resolve_symbol_edges` | Run only the symbol resolution phase |

The dream cycle runs these phases in order: lint → backlinks → sync → synthesize → extract → extract_facts → extract_atoms → resolve_symbol_edges → patterns → synthesize_concepts → recompute_emotional_weight → consolidate → propose_takes → grade_takes → calibration_profile → embed → orphans → schema-suggest → purge.

`resolve_symbol_edges` is what populates the call graph (`code-callers`/`code-callees`). It only produces edges if the schema pack extracts code symbols. The `gbrain-everything` pack does this.

### Schema Packs

| Command | Description |
|---|---|
| `gbrain schema active` | Show which pack is active |
| `gbrain schema list` | List bundled + installed packs |
| `gbrain schema use <name>` | Switch active pack |
| `gbrain schema show` | Pretty-print the active pack manifest |
| `gbrain schema stats` | Per-type page counts |
| `gbrain schema explain <type>` | Show resolved settings for a page type |
| `gbrain schema graph` | Show type/primitive graph with link edges |
| `gbrain schema lint` | Lint the active pack |

Installed packs live at `~/.gbrain/schema-packs/<name>/pack.yaml`.

Current active pack: `gbrain-everything` — code-aware meta-pack (extends creator + investor + engineer). Enables `extract_atoms` and `resolve_symbol_edges` for code symbol extraction.

### Source Management

| Command | Description |
|---|---|
| `gbrain sources add <id> --path <dir>` | Register a local source |
| `gbrain sources remove <id> --confirm-destructive` | Remove a source and its data |
| `gbrain sources remove <id> --keep-storage` | Remove source but keep data |
| `gbrain sources status <id>` | Diagnostic for a source |

### Knowledge Graph

| Command | Description |
|---|---|
| `gbrain think "<question>"` | Multi-hop synthesis across pages + takes + graph |
| `gbrain traverse-graph <slug>` | Walk the link graph from a page |
| `gbrain get-backlinks <slug>` | Find incoming links to a page |
| `gbrain get-links <slug>` | Find outgoing links from a page |
| `gbrain find-orphans` | Pages with no inbound wikilinks |
| `gbrain find-anomalies` | Statistical anomalies in recent activity |

### Facts & Takes

| Command | Description |
|---|---|
| `gbrain recall` | Query per-source hot memory (facts) |
| `gbrain extract-facts "<text>"` | Extract personal-knowledge facts from text |
| `gbrain forget-fact <id>` | Soft-delete a fact |
| `gbrain takes-list` | List takes (claims with weight/confidence) |
| `gbrain takes-search "<query>"` | Keyword search across takes |
| `gbrain think "<question>" --take` | Synthesize + append a take |

### MCP & Server

| Command | Description |
|---|---|
| `gbrain serve` | Start MCP server (holds PGLite lock) |
| `gbrain doctor` | Check MCP server health |

---

## Dream Cycle: How to Run

The dream cycle builds the call graph and runs intelligence phases. Prerequisites: a code-aware schema pack (`gbrain-everything`) and synced code files.

**Step 1: Register code repos**
```bash
gbrain sources add mb-3-cs-code --path /mnt/c/dev/dev-mb-3-cs/mb-3-cs
gbrain sources add mb4ecom-code --path /mnt/c/dev/dev-mb4ecom/mb4ecom
```

**Step 2: Sync code files**
```bash
gbrain sync --repo /mnt/c/dev/dev-mb-3-cs/mb-3-cs --strategy code --no-hard-deadline
```

**Step 3: Run dream**
```bash
gbrain dream --source mb-3-cs-code --no-hard-deadline
```

**Step 4: Verify call graph**
```bash
gbrain code-callers <symbol> --source mb-3-cs-code
```

If `resolve_symbol_edges` reports "resolved 0, unmatched N" — the code files have no function/class definitions (e.g., empty `__init__.py` files in scaffolded Anvil projects). The graph will populate as code is written.

If `extract_atoms` reports "active pack does not declare this phase" — switch to a code-aware pack:
```bash
gbrain schema use gbrain-everything
```

---

## Authority Model

1. User decision in the current session
2. Authoritative repo files and ADRs
3. OpenCode rules and validated outputs
4. GBrain memory (advisory only)

When GBrain recall conflicts with current files, the files win.

---

## OpenCode Integration

GBrain is available in OpenCode via MCP. The agent uses it automatically for semantic questions. The `/sync-gbrain` skill syncs and updates the AGENTS.md guidance block.

| OpenCode Command | Description |
|---|---|
| `/sync-gbrain` | Incremental sync (default) |
| `/sync-gbrain --dream` | Build call graph via dream cycle |
| `/sync-gbrain --full` | Full code reindex (~25-35 min) |
| `/sync-gbrain --code-only` | Only run code stage |
| `/sync-gbrain --dry-run` | Preview, no writes |
| `/setup-gbrain` | One-time setup |

---

## File Locations

| Item | Location |
|---|---|
| GBrain CLI | `~/.bun/bin/gbrain` (on PATH) |
| Config | `~/.gbrain/config.json` |
| Database | `~/.gbrain/brain.pglite/` |
| Schema packs | `~/.gbrain/schema-packs/` |
| GStack bin | `/mnt/c/mybizz/gstack/bin/` |
| GStack home | `~/.gstack/` |

---

## Troubleshooting

**PGLite lock timeout:** `gbrain serve` holds the exclusive lock. Stop it before running CLI commands:
```bash
pkill -f "gbrain serve"
sleep 2
gbrain sync ...
```

**Migration stuck:** Force-retry then apply:
```bash
gbrain apply-migrations --force-retry 0.11.0
gbrain apply-migrations --yes
```

**Missing column (e.g., event_page_id):** Manually add via PGLite:
```bash
cd ~/gbrain && bun -e "
import { PGlite } from '@electric-sql/pglite';
const db = new PGlite('/home/dev-p/.gbrain/brain.pglite');
await db.exec('ALTER TABLE timeline_entries ADD COLUMN IF NOT EXISTS event_page_id INTEGER REFERENCES pages(id) ON DELETE CASCADE');
await db.close();
"
```

**pgvector binary missing:** PGLite limitation — `could not access file "$libdir/vector"`. Does not affect search, sync, or dream. Only blocks the v0.32.2 facts migration.

**Overlapping source paths:** GBrain doesn't allow a child directory to be registered when its parent is already a source (or vice versa). Remove the conflicting source first:
```bash
gbrain sources remove <id> --confirm-destructive
```

**Stale sync lock:**
```bash
gbrain sync --break-lock
```
