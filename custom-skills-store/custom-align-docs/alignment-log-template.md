# ALIGNMENT-LOG.md template

Persists across runs — append to it, never overwrite. One entry per `/align-docs` run.

```markdown
## <date> — Alignment run

**Documents in scope:** <list>

**Resolved:**
- <drift map entry> → <fix applied, which document(s) changed>

**Deferred:**
- <drift map entry> → <why it was left open>

**Notes:** <anything worth flagging for the next run — recurring drift patterns, documents that keep needing fixes, etc.>
