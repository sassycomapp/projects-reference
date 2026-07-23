# GitHub Version Control Reference

## What is this?

This is your reference for Git and GitHub operations across all three repos. It covers commands, workflows, commit conventions, and repo-specific rules.

**Your environment:** WSL (Ubuntu) on Windows 11. All Git commands run in WSL terminal.

---

## Section 1: Your Repos

| Repo | Local Path | Remote | Branch | Purpose | Anvil Sync |
|---|---|---|---|---|---|
| project-library | `/mnt/c/mybizz/project-library` | `github.com/sassycomapp/project-library` | `main` | Planning & documentation | No |
| mb-3-cs | `/mnt/c/mybizz/mb-3-cs` | `github.com/sassycomapp/mb-3-cs` | `master` | App code (Anvil.works) | Yes (both ways) |
| gbrain-repo | Auto-managed by GStack | `github.com/sassycomapp/gbrain-repo` | `main` | GStack brain sync | No (auto-managed) |

**Windows paths:**

| Repo | Windows Path |
|---|---|
| project-library | `C:\mybizz\project-library` |
| mb-3-cs | `C:\mybizz\mb-3-cs` |

---

## Section 2: Git Commands

### Status and Inspection

**Apply in:** WSL terminal  
**Path:** any repo directory

| Command | Usage | Output |
|---|---|---|
| `git status` | Show changed, staged, and untracked files | Current branch + file status |
| `git status -sb` | Compact status with branch summary | Short format, easier to scan |
| `git diff` | Show unstaged changes | Line-by-line diff of modified files |
| `git diff --staged` | Show staged changes | What will be in the next commit |
| `git log --oneline -10` | Recent commit history | Last 10 commits, one line each |
| `git log --oneline --graph --all` | Full history graph | All branches as compact graph |

---

### Stage and Commit

**Apply in:** WSL terminal  
**Path:** repo directory

| Command | Usage | Output |
|---|---|---|
| `git add <file>` | Stage a specific file | File added to staging area |
| `git add .` | Stage all changes in current directory | All tracked/untracked changes staged |
| `git restore --staged <file>` | Unstage a file | File removed from staging area |
| `git restore <file>` | Discard changes to a file | File reverted to last committed state |
| `git commit -m "type: description"` | Commit with message | New commit created |
| `git commit --amend` | Modify the last commit | Last commit updated (message + contents) |

---

### Branch Management

**Apply in:** WSL terminal  
**Path:** repo directory

| Command | Usage | Output |
|---|---|---|
| `git branch` | List local branches | Current branch highlighted |
| `git branch <name>` | Create a new branch | Branch created (does not switch) |
| `git switch <name>` | Switch to a branch | Working tree updated |
| `git switch -c <name>` | Create and switch | New branch + immediate switch |

**Note:** You use single-branch workflows (main/master). Branch commands are for future use or when you need to isolate experimental work.

---

### Sync with Remote

**Apply in:** WSL terminal  
**Path:** repo directory

| Command | Usage | Output |
|---|---|---|
| `git fetch` | Download remote changes | Remote refs updated (local branches unchanged) |
| `git pull` | Fetch + merge into current branch | Local branch updated with remote changes |
| `git push` | Upload local commits | Current branch pushed to remote |
| `git push -u origin <branch>` | Push and set upstream | New branch pushed + tracking configured |

**Common pattern:**
```bash
cd /mnt/c/mybizz/mb-3-cs
git pull    # get latest from GitHub
# ... make changes ...
git add .
git commit -m "feat: add customer search"
git push    # send to GitHub
```

---

### History and Diff

**Apply in:** WSL terminal  
**Path:** repo directory

| Command | Usage | Output |
|---|---|---|
| `git show <commit>` | Show commit details | Commit metadata + patch |
| `git diff <branch1>..<branch2>` | Diff between branches | Changes between two branches |
| `git stash push -m "msg"` | Save uncommitted work | Working tree cleaned, changes saved |
| `git stash list` | List saved stashes | All stashes with messages |
| `git stash apply` | Reapply last stash | Stash applied (kept in list) |
| `git stash drop` | Remove last stash | Stash entry deleted |

---

## Section 3: Commit Message Convention

Use this format for all commits:

```
type: short description of what changed
```

### Types

| Type | When to use |
|---|---|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `chore` | Maintenance, dependencies, config |
| `refactor` | Code restructuring (no behavior change) |
| `style` | Formatting, whitespace (no logic change) |
| `test` | Adding or updating tests |

### Examples

```
feat: add customer search filter to contact list
fix: resolve login timeout on slow connections
docs: update onboarding ADR with credential change policy
chore: update dependencies to latest versions
refactor: simplify invoice calculation logic
style: format code consistency in bookings module
test: add unit tests for availability settings
```

### Rules

- Keep the description under 72 characters
- Use imperative mood ("add" not "added")
- Don't end with a period
- If you need more detail, add a blank line after the first line, then write a longer description

---

## Section 4: Repo-Specific Workflows

### project-library (Planning & Documentation)

**Branch:** `main`  
**Anvil sync:** No  
**Purpose:** ADRs, specs, design docs, wireframes, GStack outputs

**Workflow:**
1. Edit files in WSL or OpenCode
2. `git add .`
3. `git commit -m "docs: describe what changed"`
4. `git push`

**No special constraints.** All files can be edited locally.

---

### mb-3-cs (Application Code)

**Branch:** `master`  
**Anvil sync:** Yes (both ways)  
**Purpose:** Anvil.works application code

**Critical rules:**
- **ALL files must be created in Anvil first**, even if empty
- **YAML files are Anvil-only** — never edit locally:
  - `anvil.yaml`
  - `**/form_template.yaml`
  - `parameters.yaml`
  - `templates.yaml`
- Python files are edited locally after pulling from Anvil

**Workflow:**

```
1. Create file/element in Anvil (even if empty)
2. git pull                          ← get the new file locally
3. Edit Python/config locally        ← code here, NOT in Anvil
4. git add .
5. git commit -m "feat: describe change"
6. git push                          ← Anvil auto-syncs from GitHub
```

**If Anvil has changes you don't have locally:**
```bash
cd /mnt/c/mybizz/mb-3-cs
git pull
```

**If you have local changes and Anvil also changed something:**
```bash
git pull    # may need to resolve merge conflicts
```

---

### gbrain-repo (GStack Brain Sync)

**Branch:** `main`  
**Anvil sync:** No  
**Purpose:** Auto-managed by GStack brain sync

**No manual workflow needed.** GStack automatically pushes learnings, plans, and retros to this repo.

**To check status:**
```bash
bash /mnt/c/mybizz/gstack/bin/gstack-brain-sync --status
```

**To force a sync:**
```bash
bash /mnt/c/mybizz/gstack/bin/gstack-brain-sync --once
```

---

## Section 5: Pull Requests

### What is a Pull Request?

A Pull Request (PR) is a request to merge changes from one branch into another. It's a way to:
- Review changes before they're merged
- Run automated checks (CI/CD)
- Discuss and approve changes
- Keep a clean history

### When to Use PRs

| Situation | Use PR? |
|---|---|
| Solo work on main/master | Optional — direct push is fine |
| Collaborating with others | Yes — for code review |
| Automated CI/CD checks | Yes — triggers checks |
| Experimental changes | Yes — isolate on a branch first |

### How to Create a PR (GitHub Web UI)

1. Push your branch to GitHub:
   ```bash
   git push -u origin feature-branch
   ```
2. Go to the repo on GitHub
3. Click "Compare & pull request"
4. Add a title and description
5. Click "Create pull request"

### How to Merge a PR

1. Review the changes in the PR
2. Click "Merge pull request"
3. Confirm the merge
4. Delete the branch (optional)

### Current Status

PRs are not yet part of your workflow. This section is for future use when you establish a testing/CI protocol.

---

## Section 6: Git with OpenCode

### When OpenCode Commits

OpenCode agents follow these rules (from `AGENTS.md`):
- **Permission-only:** Never commit, push, reset, rebase, or force-pull without explicit user instruction
- **Ask first:** The agent will ask before making any git changes
- **Propose diffs:** The agent prefers showing diffs in chat over auto-applying changes

### How to Request Git Operations from OpenCode

**Apply in:** OpenCode web UI (type in chat)

| Request | What the agent does |
|---|---|
| "Commit these changes" | Stages files, writes commit message, commits |
| "Push to GitHub" | Pushes current branch to remote |
| "Pull from GitHub" | Pulls latest changes from remote |
| "Show me the diff" | Shows unstaged or staged changes |
| "Create a branch called X" | Creates and switches to new branch |
| "What's the status?" | Shows git status |

### Agent Commit Behavior

The agent will:
- Propose a commit message following the convention (`type: description`)
- Show you the diff before committing
- Ask for confirmation before pushing
- Never force-push or reset without explicit permission

---

## Section 7: Best Practices

### What to Commit

| Commit | Don't Commit |
|---|---|
| Source code (.py, .html, .css) | Build artifacts |
| Configuration files | IDE settings (.vscode/) |
| Documentation (.md) | OS files (.DS_Store) |
| YAML config (if allowed) | Secrets, API keys |
| Tests | Large binary files |

### .gitignore Rules

Both repos have `.gitignore` files. Common ignores:
- `__pycache__/`
- `*.pyc`
- `.env`
- `node_modules/`
- `.DS_Store`

### Branch Strategy

**Current:** Single-branch workflow (main/master)

**When to use branches:**
- Experimental features you want to isolate
- Changes that need review before merging
- Work that spans multiple sessions

**How to use branches:**
```bash
git switch -c feature-name
# ... make changes ...
git add .
git commit -m "feat: describe change"
git push -u origin feature-name
# ... create PR on GitHub ...
# ... merge PR ...
git switch main
git pull
git branch -d feature-name
```

### Commit Frequency

- Commit early, commit often
- Each commit should be a logical unit of work
- Don't commit broken code
- Don't commit half-finished features (use branches instead)

---

## Section 8: CI/CD

### Current State

**No CI/CD configured.** You don't have:
- GitHub Actions
- Automated tests
- Automated deployment

**Deployment is handled by Anvil.works:**
- mb-3-cs is perpetually live on Anvil
- Anvil auto-syncs from GitHub
- No separate deployment step needed

### Future Possibilities

When you're ready to add CI/CD:
- **GitHub Actions:** Run tests on every push, check code quality
- **PR checks:** Block merging until tests pass
- **Automated deployment:** Deploy to staging/production on merge

This is out of scope for now. Focus on getting the workflow solid first.

---

## Section 9: Hotkeys

### Git Commands Quick Reference

| Command | What it does | Path |
|---|---|---|
| `git status` | Show working tree status | Any repo |
| `git status -sb` | Compact status | Any repo |
| `git diff` | Show unstaged changes | Any repo |
| `git diff --staged` | Show staged changes | Any repo |
| `git add .` | Stage all changes | Any repo |
| `git add <file>` | Stage specific file | Any repo |
| `git restore --staged <file>` | Unstage file | Any repo |
| `git restore <file>` | Discard changes | Any repo |
| `git commit -m "msg"` | Commit with message | Any repo |
| `git commit --amend` | Amend last commit | Any repo |
| `git pull` | Fetch + merge | Any repo |
| `git push` | Push to remote | Any repo |
| `git log --oneline -10` | Recent history | Any repo |
| `git stash push -m "msg"` | Save work temporarily | Any repo |
| `git stash apply` | Restore stashed work | Any repo |
| `git branch` | List branches | Any repo |
| `git switch <branch>` | Switch branch | Any repo |
| `git switch -c <branch>` | Create + switch branch | Any repo |

### Repo-Specific Commands

| Command | What it does | Repo |
|---|---|---|
| `cd /mnt/c/mybizz/project-library && git pull` | Get latest docs | project-library |
| `cd /mnt/c/mybizz/mb-3-cs && git pull` | Get latest from Anvil | mb-3-cs |
| `bash /mnt/c/mybizz/gstack/bin/gstack-brain-sync --once` | Force brain sync | gbrain-repo |
| `bash /mnt/c/mybizz/gstack/bin/gstack-brain-sync --status` | Check brain sync | gbrain-repo |

### Open Code Commands

| Command | What it does |
|---|---|
| "Commit these changes" | Agent stages + commits |
| "Push to GitHub" | Agent pushes to remote |
| "Pull from GitHub" | Agent pulls from remote |
| "Show me the diff" | Agent shows changes |
| "Create a branch called X" | Agent creates branch |

---

## Section 10: Troubleshooting

### Push Rejected (Non-Fast-Forward)

**Problem:** `git push` fails with "rejected (non-fast-forward)"

**Cause:** Remote has commits you don't have locally.

**Fix:**
```bash
git pull    # merge remote changes
git push    # try again
```

### Merge Conflicts

**Problem:** `git pull` shows merge conflicts.

**Cause:** Same file changed locally and on remote.

**Fix:**
1. Open the conflicted file
2. Look for `<<<<<<<` markers
3. Choose which version to keep
4. Remove the conflict markers
5. `git add <file>`
6. `git commit -m "merge: resolve conflict in <file>"`

### Detached HEAD

**Problem:** `git status` shows "HEAD detached at ..."

**Cause:** You checked out a specific commit instead of a branch.

**Fix:**
```bash
git switch main    # or master for mb-3-cs
```

### Accidentally Committed to Wrong Branch

**Fix:**
```bash
git log --oneline -1    # note the commit hash
git switch main         # switch to correct branch
git cherry-pick <hash>  # apply the commit here
git switch wrong-branch
git reset --hard HEAD~1  # remove commit from wrong branch
```

### Need to Undo Last Commit (Not Pushed)

**Fix:**
```bash
git reset --soft HEAD~1    # keeps changes staged
# or
git reset HEAD~1           # keeps changes unstaged
```

### Need to Undo Last Commit (Already Pushed)

**Fix:**
```bash
git revert HEAD            # creates a new commit that undoes the last one
git push
```

### gbrain-repo Sync Issues

**Check status:**
```bash
bash /mnt/c/mybizz/gstack/bin/gstack-brain-sync --status
```

**Force sync:**
```bash
bash /mnt/c/mybizz/gstack/bin/gstack-brain-sync --once
```

---

## Related Documentation

| Document | Purpose |
|---|---|
| `gstack-reference.md` | GStack system overview, brain sync, artifact locations |
| `gstack-skills-reference.md` | Full skill catalog (59 skills) |
| `gbrain-reference.md` | GBrain commands, health, troubleshooting |
| `opencode-reference.md` | OpenCode config, keyboard shortcuts, MCP |
