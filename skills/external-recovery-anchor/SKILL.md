---
name: external-recovery-anchor
description: Use when running risky operations in an environment where the tool you're working in might itself break — recovery must not depend on the broken thing.
---

# External Recovery Anchor

## Core Principle

**Recovery cannot depend on what might be broken.** If you're editing Claude Code's settings, recovery can't require Claude to launch. If you're editing shell PATH, recovery can't require the shell to source its rc files.

## What "External" Means

| ❌ Not external | ✅ External |
|---|---|
| Backup in `~/.claude/` | Backup in `~/claude-backup.json` |
| Backup in the project dir | Backup outside the project dir |
| Recovery uses Claude | Recovery is pure `bash cp/mv` |
| Recovery uses bun | Recovery uses only POSIX shell builtins |
| Recovery in a var in the current session | Recovery as a plain file readable from a cold Terminal |

## Minimum Recovery Kit

Before any session-fatal operation, have:

1. **External snapshot** — copy of the config file outside its own dir
2. **Recovery one-liner** — pasted into the active chat, not just documented
3. **Tool-free mechanic** — the one-liner uses only `cp`, `mv`, `cat`, `echo`

```bash
# Example recovery for Claude Code settings
cp ~/settings-backup.json ~/.claude/settings.json

# Example recovery for shell rc
cp ~/zshrc-backup.bak ~/.zshrc
```

## Refresh Timing

Refresh the external snapshot **before each session-fatal chunk** — not once at the start of the session. Config may have changed between chunks.

```bash
# Before each session-fatal chunk
cp ~/.claude/settings.json ~/settings-backup-$(date +%s).json
```

## Recovery Doc Naming

Name recovery files to be readable from a cold Terminal with no tool context:
- ✅ `~/RECOVERY-claude-settings.md`
- ✅ `~/settings-backup-20260619.json`
- ❌ `./teardown/backups/b3/` (requires knowing project context)

## Verify Recovery Works Before You Need It

Dry-run the recovery one-liner on a non-fatal config first. If it fails, debug it before you're in an emergency.

## Preflight Checklist (session-fatal operations)

- [ ] External snapshot taken and verified (sha256 matches)
- [ ] Recovery one-liner pasted into active chat
- [ ] Recovery one-liner tested on equivalent non-fatal file
- [ ] Recovery requires only POSIX shell (no Claude, no bun, no npm)
- [ ] Recovery file named descriptively, at `~/` level
