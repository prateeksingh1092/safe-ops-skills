---
name: claude-stock-audit
description: Use when verifying or restoring ~/.claude to vanilla stock Claude Code state after removing an overlay system, suspecting residual hooks/config/processes from a prior installation, or before declaring a teardown complete.
---

# Claude Stock Audit

## Overview

A Claude Code installation that had a third-party overlay (hooks, agents, skills, custom settings) needs a 4-domain sweep before it can be declared clean. Checking one domain and skipping others misses residue. Run all 4 domains — fan them out in parallel.

## The 4 Domains (always all 4)

| Domain | What to check | Owner |
|---|---|---|
| **Filesystem** | `~/.claude/` structure, unexpected files/dirs | fs agent |
| **Settings** | `settings.json`, `settings.local.json`, `.claude.json` | settings agent |
| **Shell rc** | `.zshrc`, `.zprofile`, `.zshenv`, `.bash_profile` | shell agent |
| **System** | launchd, cron, ports, running processes | system agent |

## Parallel Dispatch Pattern

Fan all 4 out simultaneously — they don't share files:

```
Agent 1 (fs)       → audit + clean ~/.claude filesystem
Agent 2 (settings) → audit + clean settings.json / .claude.json
Agent 3 (shell-rc) → audit shell rc files for overlay refs
Agent 4 (system)   → launchctl, crontab -l, lsof :PORT, ps aux
```

Collect results; each agent reports CLEAN or lists what it found and fixed. Synthesize after all 4 finish.

## What Stock ~/.claude Looks Like

**Expected dirs:**
- `hooks/` — empty (all hook arrays `[]` in settings.json)
- `skills/` — only user-kept skills (non-overlay)
- `commands/` — only non-overlay command files
- `plugins/`, `plugins.off/` — stock plugin dirs
- `projects/`, `sessions/`, `cache/`, `channels/`, `telemetry/` — stock Claude Code runtime dirs

**Should NOT exist post-teardown:**
- Any `OVERLAY-DEAD-*` or disabled variants of hook/agent files
- Overlay-named dirs: `PAI/`, `MEMORY/`, `agents.off/`, `PAI-Install/`
- Hook infrastructure subdirs: `hooks/handlers/`, `hooks/lib/`, `hooks/security/`
- Overlay migration library: `lib/`
- Daemon files: `daemon/`, `daemon.log`, `daemon-auth-*`
- Installer bootstrap: `install.sh`, `statusline-command.sh`
- Overlay README/LICENSE at root
- Temp working dirs: `tmp-*/`
- Git repo artifacts: `.gitignore`, `.gitattributes`, `.gitmodules`
- Stale settings backups: `settings.json.bak*`
- Dangling `.env` symlinks

## Domain 1 — Filesystem Audit

```bash
# Overlay-disabled remnants
find ~/.claude -name '*DEAD*' -o -name '*.disabled' 2>/dev/null

# Top-level unexpected items
ls -la ~/.claude/

# Hook subdirs (should be absent)
ls ~/.claude/hooks/handlers/ 2>/dev/null
ls ~/.claude/hooks/lib/ 2>/dev/null

# Skills / commands content
ls ~/.claude/skills/
ls ~/.claude/commands/

# Overlay-named items (2 levels deep)
find ~/.claude -maxdepth 2 -iname '*overlay*' -o -iname '*pai*' 2>/dev/null \
  | grep -v 'projects/'
```

**Staged-delete protocol** (if a guard hook is active):
```bash
TS=$(date +%s)
mkdir -p ~/.pai-graveyard/$TS
mv ~/.claude/<target> ~/.pai-graveyard/$TS/
# verify
rm -rf ~/.pai-graveyard/$TS/
rmdir ~/.pai-graveyard 2>/dev/null  # when done
```
Direct `rm` on live paths may be blocked; always `mv` to graveyard first.

## Domain 2 — Settings Audit

**settings.json — check for:**
- `env` keys pointing to deleted overlay dirs (e.g. `OVERLAY_DIR`, `PAI_DIR`, `PAI_CONFIG_DIR`)
- `allowedHttpHookUrls` containing overlay ports (e.g. `localhost:31337`)
- `_docs`, `_contextFiles_docs`, `loadAtStartup._docs` — overlay doc blocks
- `counts` block — stale overlay stats
- Hook arrays: all should be `[]`

**settings.local.json — check for:**
- Stale `allow` entries with absolute paths into deleted overlay dirs

**~/.claude.json — check for:**
- Any overlay MCP servers (keep: user's non-overlay MCPs and their API keys)

```bash
python3 -m json.tool ~/.claude/settings.json > /dev/null  # validate JSON after edits
grep -c '"command"' ~/.claude/settings.json  # expect 0 (no registered hooks)
```

## Domain 3 — Shell RC Audit

```bash
grep -n 'overlay\|PAI\|pai\|:31337\|append-system-prompt' \
  ~/.zshrc ~/.zprofile ~/.zshenv ~/.bash_profile 2>/dev/null
```

**Fix:** comment out any live overlay alias/env/flag lines. Keep: bun PATH blocks, general tool aliases, non-overlay env vars.

## Domain 4 — System Audit

```bash
# Launchd
launchctl list | grep -iE 'overlay|pai|claudie|pulse'
find ~/Library/LaunchAgents -iname '*overlay*' -o -iname '*pai*' 2>/dev/null

# Cron
crontab -l 2>/dev/null

# Ports (change to overlay's ports)
lsof -i :31337 2>/dev/null
lsof -i :31888 2>/dev/null

# Processes
ps aux | grep -iE 'overlay|pai|claudie|voice-server' | grep -v grep

# Config dir
ls ~/.config/overlay 2>/dev/null  # should be gone
```

## Common Misses (found in real teardowns)

| Where | What got missed | Why |
|---|---|---|
| `hooks/handlers/`, `hooks/lib/`, `hooks/security/` | Full hook infrastructure | `find -maxdepth 1` didn't recurse |
| `settings.json _docs` block | doc strings | Treated as false positive — they were real |
| `settings.json env` | Dead-path env vars | Not a registered hook, so not noticed |
| `settings.json allowedHttpHookUrls` | Overlay port | Easy to overlook non-hook key |
| `settings.local.json` | Stale `allow` entries | Separate file, separate agent pass |
| `~/.claude/lib/` | Migration library | Not overlay-named obviously |
| `~/.claude/daemon/` | B-daemon files | Launchd was clean but files remained |
| `.gitignore` / `.gitattributes` | Overlay was a git repo | Not overlay-named |
| `settings.json.bak*` | Stale bak variants | Accumulate silently |

## Post-Sweep Checklist

- [ ] `find ~/.claude -name '*DEAD*'` → 0 results
- [ ] `ls ~/.claude/hooks/` → empty (no subdirs)
- [ ] Skills and commands dirs contain only intended non-overlay files
- [ ] `grep -c '"command"' ~/.claude/settings.json` → 0
- [ ] No overlay env vars in `settings.json`
- [ ] `allowedHttpHookUrls` is `[]`
- [ ] No overlay docs blocks (`_docs`, `_contextFiles_docs`, `counts`)
- [ ] No stale overlay `allow` entries in `settings.local.json`
- [ ] No overlay aliases active in shell rc files
- [ ] `launchctl list | grep -i overlay` → only OS entries
- [ ] `crontab -l` → empty or only non-overlay jobs
- [ ] Overlay ports unbound (`lsof` returns nothing)
- [ ] No overlay processes in `ps aux`
- [ ] Overlay config dirs (`~/.config/overlay`) → gone
