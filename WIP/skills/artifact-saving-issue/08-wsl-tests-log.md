dev-p@dev-p:~$ echo "=== 1: Claude-path binary exists? ==="
ls -la ~/.claude/skills/gstack/bin/gstack-slug 2>&1

echo "=== 2: how many skill files leak the Claude path? ==="
grep -rl '~/.claude/skills/gstack' /mnt/c/mybizz/skills/ | wc -l

echo "=== 5: state-dir env vars ==="
echo "GSTACK_HOME=${GSTACK_HOME:-NOT_SET}"
echo "GSTACK_STATE_ROOT=${GSTACK_STATE_ROOT:-NOT_SET}"

echo "=== 6: which skill locations actually exist? ==="
ls -la ~/.claude/skills/ 2>&1
echo "---"
ls -la ~/.config/opencode/skills/ 2>&1

echo "=== 7: is gstack-slug on PATH? ==="
which gstack-slug 2>&1
=== 1: Claude-path binary exists? ===
-rwxrwxrwx 1 dev-p dev-p 2420 Jun 15 23:41 /home/dev-p/.claude/skills/gstack/bin/gstack-slug
=== 2: how many skill files leak the Claude path? ===
182
=== 5: state-dir env vars ===
GSTACK_HOME=NOT_SET
GSTACK_STATE_ROOT=NOT_SET
=== 6: which skill locations actually exist? ===
total 8
drwxr-xr-x 2 dev-p dev-p 4096 Jun 15 23:39 .
drwxr-xr-x 3 dev-p dev-p 4096 Jun 15 23:39 ..
lrwxrwxrwx 1 dev-p dev-p   25 Jun 15 23:39 gstack -> /mnt/c/mybizz/gstack
---
total 28
drwxr-xr-x 2 dev-p dev-p 4096 Jun 13 01:20 .
drwxr-xr-x 4 dev-p dev-p 4096 Jun 16 12:47 ..
lrwxrwxrwx 1 dev-p dev-p   35 Jun 13 01:14 autoplan -> /mnt/c/mybizz/skills/autoplan/
lrwxrwxrwx 1 dev-p dev-p   36 Jun 13 01:14 benchmark -> /mnt/c/mybizz/skills/benchmark/
lrwxrwxrwx 1 dev-p dev-p   43 Jun 13 01:14 benchmark-models -> /mnt/c/mybizz/skills/benchmark-models/
lrwxrwxrwx 1 dev-p dev-p   33 Jun 13 01:14 browse -> /mnt/c/mybizz/skills/browse/
lrwxrwxrwx 1 dev-p dev-p   33 Jun 13 01:14 canary -> /mnt/c/mybizz/skills/canary/
lrwxrwxrwx 1 dev-p dev-p   34 Jun 13 01:14 careful -> /mnt/c/mybizz/skills/careful/
lrwxrwxrwx 1 dev-p dev-p   32 Jun 13 01:14 codex -> /mnt/c/mybizz/skills/codex/
lrwxrwxrwx 1 dev-p dev-p   41 Jun 13 01:14 connect-chrome -> /mnt/c/mybizz/skills/connect-chrome/
lrwxrwxrwx 1 dev-p dev-p   42 Jun 13 01:14 context-restore -> /mnt/c/mybizz/skills/context-restore/
lrwxrwxrwx 1 dev-p dev-p   39 Jun 13 01:14 context-save -> /mnt/c/mybizz/skills/context-save/
lrwxrwxrwx 1 dev-p dev-p   30 Jun 13 01:14 cso -> /mnt/c/mybizz/skills/cso/
lrwxrwxrwx 1 dev-p dev-p   46 Jun 13 01:14 design-consultation -> /mnt/c/mybizz/skills/design-consultation/
lrwxrwxrwx 1 dev-p dev-p   38 Jun 13 01:14 design-html -> /mnt/c/mybizz/skills/design-html/
lrwxrwxrwx 1 dev-p dev-p   40 Jun 13 01:14 design-review -> /mnt/c/mybizz/skills/design-review/
lrwxrwxrwx 1 dev-p dev-p   41 Jun 13 01:14 design-shotgun -> /mnt/c/mybizz/skills/design-shotgun/
lrwxrwxrwx 1 dev-p dev-p   39 Jun 13 01:14 devex-review -> /mnt/c/mybizz/skills/devex-review/
lrwxrwxrwx 1 dev-p dev-p   34 Jun 13 01:14 diagram -> /mnt/c/mybizz/skills/diagram/
lrwxrwxrwx 1 dev-p dev-p   44 Jun 13 01:15 document-generate -> /mnt/c/mybizz/skills/document-generate/
lrwxrwxrwx 1 dev-p dev-p   43 Jun 13 01:15 document-release -> /mnt/c/mybizz/skills/document-release/
lrwxrwxrwx 1 dev-p dev-p   33 Jun 13 01:15 freeze -> /mnt/c/mybizz/skills/freeze/
lrwxrwxrwx 1 dev-p dev-p   69 Jun 13 01:20 gstack-openclaw-ceo-review -> /mnt/c/mybizz/skills/openclaw/skills/gstack-openclaw-ceo-review/
lrwxrwxrwx 1 dev-p dev-p   70 Jun 13 01:20 gstack-openclaw-investigate -> /mnt/c/mybizz/skills/openclaw/skills/gstack-openclaw-investigate/
lrwxrwxrwx 1 dev-p dev-p   71 Jun 13 01:20 gstack-openclaw-office-hours -> /mnt/c/mybizz/skills/openclaw/skills/gstack-openclaw-office-hours/
lrwxrwxrwx 1 dev-p dev-p   64 Jun 13 01:20 gstack-openclaw-retro -> /mnt/c/mybizz/skills/openclaw/skills/gstack-openclaw-retro/
lrwxrwxrwx 1 dev-p dev-p   41 Jun 13 01:15 gstack-upgrade -> /mnt/c/mybizz/skills/gstack-upgrade/
lrwxrwxrwx 1 dev-p dev-p   32 Jun 13 01:15 guard -> /mnt/c/mybizz/skills/guard/
lrwxrwxrwx 1 dev-p dev-p   62 Jun 13 01:20 hackernews-frontpage -> /mnt/c/mybizz/skills/browser-skills/hackernews-frontpage/
lrwxrwxrwx 1 dev-p dev-p   33 Jun 13 01:15 health -> /mnt/c/mybizz/skills/health/
lrwxrwxrwx 1 dev-p dev-p   38 Jun 13 01:15 investigate -> /mnt/c/mybizz/skills/investigate/
lrwxrwxrwx 1 dev-p dev-p   36 Jun 13 01:15 ios-clean -> /mnt/c/mybizz/skills/ios-clean/
lrwxrwxrwx 1 dev-p dev-p   44 Jun 13 01:15 ios-design-review -> /mnt/c/mybizz/skills/ios-design-review/
lrwxrwxrwx 1 dev-p dev-p   34 Jun 13 01:15 ios-fix -> /mnt/c/mybizz/skills/ios-fix/
lrwxrwxrwx 1 dev-p dev-p   33 Jun 13 01:15 ios-qa -> /mnt/c/mybizz/skills/ios-qa/
lrwxrwxrwx 1 dev-p dev-p   35 Jun 13 01:15 ios-sync -> /mnt/c/mybizz/skills/ios-sync/
lrwxrwxrwx 1 dev-p dev-p   42 Jun 13 01:15 land-and-deploy -> /mnt/c/mybizz/skills/land-and-deploy/
lrwxrwxrwx 1 dev-p dev-p   41 Jun 13 01:15 landing-report -> /mnt/c/mybizz/skills/landing-report/
lrwxrwxrwx 1 dev-p dev-p   32 Jun 13 01:15 learn -> /mnt/c/mybizz/skills/learn/
lrwxrwxrwx 1 dev-p dev-p   35 Jun 13 01:15 make-pdf -> /mnt/c/mybizz/skills/make-pdf/
lrwxrwxrwx 1 dev-p dev-p   39 Jun 13 01:15 office-hours -> /mnt/c/mybizz/skills/office-hours/
lrwxrwxrwx 1 dev-p dev-p   46 Jun 13 01:15 open-gstack-browser -> /mnt/c/mybizz/skills/open-gstack-browser/
lrwxrwxrwx 1 dev-p dev-p   37 Jun 13 01:15 pair-agent -> /mnt/c/mybizz/skills/pair-agent/
lrwxrwxrwx 1 dev-p dev-p   42 Jun 13 01:15 plan-ceo-review -> /mnt/c/mybizz/skills/plan-ceo-review/
lrwxrwxrwx 1 dev-p dev-p   45 Jun 13 01:15 plan-design-review -> /mnt/c/mybizz/skills/plan-design-review/
lrwxrwxrwx 1 dev-p dev-p   44 Jun 13 01:15 plan-devex-review -> /mnt/c/mybizz/skills/plan-devex-review/
lrwxrwxrwx 1 dev-p dev-p   42 Jun 13 01:15 plan-eng-review -> /mnt/c/mybizz/skills/plan-eng-review/
lrwxrwxrwx 1 dev-p dev-p   36 Jun 13 01:15 plan-tune -> /mnt/c/mybizz/skills/plan-tune/
lrwxrwxrwx 1 dev-p dev-p   29 Jun 13 01:15 qa -> /mnt/c/mybizz/skills/qa/
lrwxrwxrwx 1 dev-p dev-p   34 Jun 13 01:15 qa-only -> /mnt/c/mybizz/skills/qa-only/
lrwxrwxrwx 1 dev-p dev-p   32 Jun 13 01:15 retro -> /mnt/c/mybizz/skills/retro/
lrwxrwxrwx 1 dev-p dev-p   33 Jun 13 01:15 review -> /mnt/c/mybizz/skills/review/
lrwxrwxrwx 1 dev-p dev-p   33 Jun 13 01:15 scrape -> /mnt/c/mybizz/skills/scrape/
lrwxrwxrwx 1 dev-p dev-p   48 Jun 13 01:15 setup-browser-cookies -> /mnt/c/mybizz/skills/setup-browser-cookies/
lrwxrwxrwx 1 dev-p dev-p   39 Jun 13 01:15 setup-deploy -> /mnt/c/mybizz/skills/setup-deploy/
lrwxrwxrwx 1 dev-p dev-p   39 Jun 13 01:15 setup-gbrain -> /mnt/c/mybizz/skills/setup-gbrain/
lrwxrwxrwx 1 dev-p dev-p   31 Jun 13 01:15 ship -> /mnt/c/mybizz/skills/ship/
lrwxrwxrwx 1 dev-p dev-p   35 Jun 13 01:15 skillify -> /mnt/c/mybizz/skills/skillify/
lrwxrwxrwx 1 dev-p dev-p   31 Jun 13 01:15 spec -> /mnt/c/mybizz/skills/spec/
lrwxrwxrwx 1 dev-p dev-p   38 Jun 13 01:15 sync-gbrain -> /mnt/c/mybizz/skills/sync-gbrain/
lrwxrwxrwx 1 dev-p dev-p   35 Jun 13 01:15 unfreeze -> /mnt/c/mybizz/skills/unfreeze/
=== 7: is gstack-slug on PATH? ===
dev-p@dev-p:~$

---

dev-p@dev-p:~$ ls -la ~/.gstack/slug-cache/
cat ~/.gstack/slug-cache/*
total 16
drwxr-xr-x 2 dev-p dev-p 4096 Jun 26 00:34 .
drwxr-xr-x 8 dev-p dev-p 4096 Jun 15 23:58 ..
-rw------- 1 dev-p dev-p    5 Jun 25 21:54 _home_dev-p
-rw------- 1 dev-p dev-p   27 Jun 26 00:34 _mnt_c_mybizz_project-library
dev-psassycomapp-project-librarydev-p@dev-p:~$

---

dev-p@dev-p:~$ grep -n "SLUG=" /mnt/c/mybizz/skills/context-save/SKILL.md
grep -n "SLUG=" /mnt/c/mybizz/skills/health/SKILL.md
842:TITLE_SLUG=$(printf '%s' "$RAW" | tr '[:upper:]' '[:lower:]' | tr -s ' \t' '-' | tr -cd 'a-z0-9.-' | cut -c1-60)
843:TITLE_SLUG="${TITLE_SLUG:-untitled}"
dev-p@dev-p:~$
