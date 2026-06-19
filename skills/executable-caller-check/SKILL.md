---
name: executable-caller-check
description: Use when renaming or removing a file or directory that might be referenced by other scripts, config files, cron jobs, or system services — before any rename or deletion.
---

# Executable Caller Check

## The Distinction

| Reference type | Safe to have dangling? | Action |
|---|---|---|
| **Executable** — scripts, JSON config, plists, cron, launchd | ❌ Will error at runtime | Must update or block rename |
| **Documentation** — comments, READMEs, docs, data files | ✅ Harmless | Leave as-is |

A string match in a README for the old filename is fine. A string match in a cron job or launchd plist is not.

## Search Scope

Before renaming/removing `<filename>`:

```bash
# 1. Hook registrations (JSON — executable)
grep -r "filename.hook.ts" ~/.claude/settings.json ~/.claude/settings.local.json

# 2. Shell rc files (executable)
grep -r "filename" ~/.zshrc ~/.zprofile ~/.zshenv ~/.bash_profile

# 3. LaunchAgents / LaunchDaemons (executable)
grep -r "filename" ~/Library/LaunchAgents/ /Library/LaunchAgents/ 2>/dev/null

# 4. Cron (executable)
crontab -l 2>/dev/null | grep "filename"

# 5. Project scripts (executable)
grep -r "filename" <project>/teardown/ <project>/scripts/ 2>/dev/null

# 6. Other config files (JSON/YAML — executable)
grep -r "filename" ~/.claude.json ~/.claude/*.json 2>/dev/null
```

**"0 live callers" confirmation = gate before proceeding.**

## Variable-Expanded Paths (flag explicitly)

Static grep misses:
```bash
HOOK_DIR=~/.claude/hooks
$HOOK_DIR/filename.hook.ts   # won't match grep for "filename.hook.ts"
```

Flag variable-expanded or dynamically-constructed path references in your audit doc. Search for the variable name too:

```bash
grep -r "HOOK_DIR\|hooks_dir\|hook_path" <scope>
```

## What "0 Callers" Looks Like

```bash
$ grep -r "LoadContext.hook.ts" ~/.claude/settings.json
# (no output)
$ crontab -l | grep "LoadContext"
# (no output)
✓ 0 live callers — safe to rename
```

## When You Find Callers

1. Update the caller first (change the reference to the new name)
2. Then rename/remove the target
3. Verify the caller still works with the new name
4. Only then consider the rename complete

Don't rename the file and plan to "fix callers later" — later gets skipped.
