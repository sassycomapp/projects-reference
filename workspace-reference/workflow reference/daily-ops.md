# Daily Operations — Quick Reference

---

## START OPENCODE

```bash
cd /mnt/c/dev/dev-mb-3-cs/mb-3-cs-project-library
opencode web --hostname 0.0.0.0
```
Then open browser: `http://localhost:3000`

---

## STOP / RESTART OPENCODE

```bash
pkill -f "opencode web"
```
Then run Start sequence above.

---

## AFTER DOING WORK — run every time

```bash
cd /mnt/c/dev/dev-mb-3-cs/mb-3-cs-project-library
git add -A
git commit -m "your message here"
git push origin main
gbrain sync
gbrain embed --stale
```

---

## RUN MB-ALIGN-DOCS AFTER A DOCS MANAGER RUN THAT DID STRUCTURAL CHANGES

If Docs Manager renamed, moved, or retired files within a project-library
(or project-library-global), run mb-align-docs against that same project
next — before considering the documentation environment stable:

```
/mb-align-docs (project-name)
```

This catches register or front-matter drift that Docs Manager's structural
changes may have introduced. Skip if the Docs Manager run was text-only
corrections (no renames, moves, or retires).

---

## GBRAIN HEALTH CHECK

```bash
gbrain doctor
```

---

## GBRAIN TEST — confirm search is working

```bash
gbrain query "client instance architecture"
```
Should return results. If toast error appears — see Fix below.

---

## FIX: GBRAIN WASM ERROR / WON'T START

Close the WSL terminal completely. Open a fresh one. Retry.

If still broken:
```bash
rm -rf ~/.gbrain/brain.pglite
gbrain init
gbrain import ~/gbrain-backup-20260615/
gbrain embed --stale
```

---

## FIX: GBRAIN SYNC SAYS "NO ALLOWLISTED CHANGES"

This is normal — means nothing new to sync from GStack working state.  
Your git push already backed up the project-library. Nothing to worry about.

---

## CHECK WHAT'S NOT COMMITTED YET

```bash
cd /mnt/c/dev/dev-mb-3-cs/mb-3-cs-project-library
git status
```

---

## OPENCODE — INFRASTRUCTURE (type in the chat box)

### GBrain
| Command | What it does |
|---|---|
| `/setup-gbrain doctor` | GBrain health check (equivalent of `gbrain doctor` in WSL) |
| `/sync-gbrain` | Sync GBrain state (equivalent of `gbrain sync` in WSL) |

### Save and Restore Your Place
| Command | What it does |
|---|---|
| `/context-save` | Save where you are before closing OpenCode |
| `/context-restore` | Pick up where you left off in a new session |

### Tell the Agent to Search GBrain
Add this to the start of any prompt:
```
Before starting, search GBrain for: [your topic here]
```

### MCP / GBrain Not Responding in OpenCode
Restart OpenCode from WSL:
```bash
pkill -f "opencode web"
cd /mnt/c/dev/dev-mb-3-cs/mb-3-cs-project-library
opencode web --hostname 0.0.0.0
```

---

## OPENCODE — USEFUL KEYS

| Key | What it does |
|---|---|
| Tab | Switch Plan / Build mode |
| Ctrl+G | Stop the agent mid-response |
| Ctrl+X then N | New session |

---

*That's it. Everything else lives in the reference docs.*
