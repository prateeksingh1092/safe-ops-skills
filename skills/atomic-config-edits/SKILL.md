---
name: atomic-config-edits
description: Use when editing JSON config files that gate a tool's ability to launch — where malformed output means total outage and recovery must not depend on the broken tool.
---

# Atomic Config Edits

## Overview

A malformed `settings.json` means Claude Code won't launch at all. Recovery must be pure bash — no Claude, no bun, no anything that might be broken. Design for that failure mode before you start.

## Edit Protocol

```bash
# 1. Snapshot before first edit of the session
cp settings.json settings.json.bak-$(date +%s)

# 2. Edit to a temp file, validate, then mv (never edit live file directly)
cp settings.json settings.json.tmp
# ... make edits to settings.json.tmp ...
python3 -m json.tool settings.json.tmp > /dev/null && \
  mv settings.json.tmp settings.json || \
  echo "INVALID — keeping original"

# 3. Keep external snapshot outside the config dir
cp ~/.claude/settings.json ~/settings-backup.json  # survives ~/.claude damage
```

## Verify After Every Session-Fatal Chunk

```bash
# Validate JSON
python3 -m json.tool ~/.claude/settings.json > /dev/null && echo "valid"

# Fresh probe (changes take effect next session, not current)
claude -p "test" -d /tmp
```

Never stack two session-fatal edits without a fresh probe between them.

## Recovery Must Be Tool-Free

Write your recovery one-liner before executing any session-fatal step:

```bash
# Recovery if Claude won't launch after a bad settings edit:
cp ~/settings-backup.json ~/.claude/settings.json
```

Paste it into the active chat so it's instantly accessible if the tool crashes mid-edit.

## Claude Code Specifics

| Config file | If malformed |
|---|---|
| `settings.json` | Claude won't launch — recovery must be pure bash |
| `settings.local.json` | Same — local overrides, same risk |
| `.claude.json` | MCP servers won't load — less catastrophic |
| `CLAUDE.md` | Bad but recoverable in-session |

**Take-effect timing:** `settings.json`, `CLAUDE.md`, hooks, system prompt — all take effect on the **next** session. Verify with a fresh probe, never in the current session.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Editing live settings.json with sed in place | Write to .tmp, validate, mv |
| Removing a `}` thinking JSON allows trailing content | Validate every edit |
| Stacking 3 setting changes before probing | One chunk → one probe |
| Recovery procedure depends on Claude launching | Make it pure bash |
