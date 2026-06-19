---
name: reversible-disable-primitive
description: Use when disabling software artifacts (files, hooks, agents, config keys) where permanent deletion is premature — need full rollback capability without scattered disabled dirs.
---

# Reversible Disable Primitive

## Overview

Prepend a fixed prefix to the artifact's basename **in place**. No moving to `.disabled/` dirs. The artifact stays at its original parent, discoverable, and rollback is one `sed` rename sweep.

## The Pattern

```bash
# Disable
mv ~/.claude/hooks/LoadContext.hook.ts \
   ~/.claude/hooks/PAI-DEAD-LoadContext.hook.ts

# Restore
mv ~/.claude/hooks/PAI-DEAD-LoadContext.hook.ts \
   ~/.claude/hooks/LoadContext.hook.ts
```

**Kill-list query** — find everything disabled in one shot:
```bash
find ~/.claude <project> -name 'PREFIX-DEAD-*'
```

## Three Variants

| Artifact type | What to do |
|---|---|
| Presence-loaded files/dirs (skills, agents, unwired hooks) | Rename only — loader won't find it |
| Wired artifacts (registered hooks, cron jobs, launchd plists) | Rename file **AND** remove its registration — dangling registration errors at runtime |
| Settings keys / `@import` lines | No file to rename — comment-out or remove the key; snapshot settings first |

**Wired hook example:**
```bash
# 1. Remove registration from settings.json (atomic write + validate)
# 2. Then rename the file
mv hooks/SecurityPipeline.hook.ts hooks/PAI-DEAD-SecurityPipeline.hook.ts
# Revert = restore registration AND strip prefix
```

## Rules

- **No `rm` in the disable phase.** Every action must be reversible.
- Verify after each chunk with a fresh probe session — never assume in-session.
- Confirm 0 dangling registrations after deregistering wired artifacts.
- For batch reversal: `find . -name 'PREFIX-DEAD-*' | sed 's/PREFIX-DEAD-//'` to generate restore commands.

## Why Not `.disabled/` Dirs

- Scattered dirs break `find`-based discovery
- Moving to a different parent dir loses the original location context
- Prefix-in-place: `find -name 'PREFIX-DEAD-*'` is the complete state audit

## Revert Proof

Before any unattended run-through, prove reversibility on one chunk:
1. Disable artifact
2. Verify it's disabled (fresh probe)
3. Re-enable it
4. Verify it's re-enabled
5. Re-disable back to end-state

This proves your rollback mechanic works before you're several chunks in.
