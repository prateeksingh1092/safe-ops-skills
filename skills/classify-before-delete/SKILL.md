---
name: classify-before-delete
description: Use when planning to remove or disable files in a directory that may contain a mix of user data, config, and reconstructible artifacts — before any destructive operation.
---

# Classify Before Delete

## Overview

Before touching anything, classify every target. User data gets backed up and hash-verified. Config gets snapshotted. Reconstructible metadata needs nothing. Misclassifying user data as reconstructible is data loss.

## Three Classes

| Class | Definition | Action before deletion |
|---|---|---|
| **USER DATA** | Irreplaceable: memories, personal notes, project artifacts, curated exports | Backup + sha256 verify; MISSING=N→hard stop |
| **CONFIG** | Reconstructible from defaults/docs, but need restore point: settings, rc files, plists | Snapshot; keep external copy |
| **META** | Auto-generated, reconstructible, no user content: logs, caches, lock files, temp dirs | No backup needed |

**Conservative fallback:** anything ambiguous under a user-data-adjacent path → USER DATA.

## Manifest Pattern

Build a manifest of all USER DATA paths before any operation:

```bash
# Example manifest (backup-manifest.jsonl)
{"path": "~/.claude/PAI/MEMORY/RELATIONSHIP/", "class": "USER_DATA", "reason": "memory files", "backed_up": false}
{"path": "~/.claude/PAI/MEMORY/LEARNING/",     "class": "USER_DATA", "reason": "learning files", "backed_up": false}
```

Verify every manifest entry is backed up before proceeding:

```bash
# Verify backup
sha256sum -c backup-hashes.txt && echo "VERIFY OK" || echo "MISSING — STOP"
```

**MISSING=N>0 = hard stop.** Never proceed past a missing-backup warning.

## Guard Hook Pattern

A PreToolUse hook can auto-append unrecognized user-data paths to the manifest — never blocks, just marks for review:

```typescript
// If path matches user-data pattern and not in manifest → append + warn
// Never auto-block based on manifest alone; let human review
```

## Minimum Backup Standard

- Use sha256 hashes (not just file count) for USER DATA backups
- External backup dir (outside the target dir) survives target-dir damage
- Verify backup BEFORE any destructive operation, not after
- Permission: `chmod -R go-rwx ~/backups/` — owner-only

## Common Misclassifications

| Item | Common mistake | Correct class |
|---|---|---|
| JSON files with user-authored content | META (it's a file, must be generated) | USER DATA |
| Per-session memory/state | CONFIG (it's config-like) | USER DATA |
| Installer scripts | USER DATA (it's code!) | META (reconstructible from repo) |
| Log files with unique session content | META | USER DATA (if content is irreplaceable) |
