---
name: staged-delete-safety
description: Use when performing irreversible deletions — mass rm, secret removal, or large disk reclaims — where there is no undo and you need a safety window between staging and permanent removal.
---

# Staged Delete Safety

## Overview

Never `rm` directly. Move to a graveyard staging dir first. Verify everything still works. Then reclaim from the graveyard. Space isn't freed until reclaim — that's the feature.

## The Pattern

```bash
# 1. Stage (instant same-volume mv — no I/O, no space freed yet)
TS=$(date +%s)
mkdir -p ~/.graveyard/$TS
mv <target> ~/.graveyard/$TS/<target-name>

# 2. Verify: probe works, tools work, df shows pending reclaim
claude -p "are you stock?" -d /tmp    # or your equivalent probe
df -h ~

# 3. Reclaim only after verify passes
rm -rf ~/.graveyard/$TS/<target-name>

# 4. Clean up empty graveyard when cycle is done
rmdir ~/.graveyard 2>/dev/null
```

## Enforce Structurally with a Guard Hook

A PreToolUse hook on Bash/Write can enforce the invariant automatically:

```
rm/rmdir/trash on live absolute path → BLOCKED
rm inside ~/.graveyard/ → ALLOWED
KEEP-list paths (backups, personal-archive, tool dirs) → ALWAYS BLOCKED
Catastrophic dirs ($HOME, /, ~/.claude, ~/Documents) → ALWAYS BLOCKED
```

**Guard honest limits:** Only catches literal-path `rm`. These bypass it:
- `find -delete`
- Variable-expanded paths (`rm $TARGET`)
- `> file` truncation
- `unlink`

Backups + the staged window remain the primary net; the guard is the backstop.

## Per-Chunk Discipline

One chunk = one move + one verify + one reclaim. Never batch stage→verify→reclaim across multiple targets without a verify between.

| Step | What to verify |
|---|---|
| After stage | Target gone from live path; dependent tools still work |
| After reclaim | Space actually freed (`df`); graveyard empty |

## Secrets

For secrets specifically: delete file → verify no process holds an open fd to it → rotate at provider. On-disk deletion alone doesn't revoke a compromised secret.

## Don't Use This For

- `/tmp` debris that macOS will auto-purge on reboot — not worth staging
- Files you already know are reconstructible with zero user data — backup then direct rm is fine
