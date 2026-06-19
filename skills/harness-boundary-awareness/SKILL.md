---
name: harness-boundary-awareness
description: Use when running Claude Code agents in auto-mode to perform irreversible mass operations — the harness blocks agents from self-authorizing destructive actions by design.
---

# Harness Boundary Awareness

## What the Harness Blocks

Claude Code's auto-mode classifier blocks agents from:

1. **Running irreversible mass-`rm`** of pre-existing data — even with "run autonomously" instruction
2. **Self-adding `rm` permissions** — treated as self-modification, blocked regardless of instruction

These are **features, not bugs.** The harness treats a generic autonomy grant as insufficient license for irreversible deletion.

## The Self-Grant Rule

An agent cannot add its own permission to do irreversible things. Even if the user said "do everything autonomously," the agent self-adding `"Bash(rm -rf:~/**)"` to settings.json is classified as self-modification and blocked.

**Why:** An agent that can grant itself permissions can escalate without bound.

## What the Principal Must Do

For irreversible mass operations, the **principal** must add the scoped permission themselves:

```json
"permissions": {
  "allow": [
    "Bash(rm -rf:~/.graveyard/**)",
    "Bash(rm -rf:/Users/you/.graveyard/**)"
  ]
}
```

This authorizes only graveyard reclaim — not open-ended `rm`. Scope matters: the narrower the permission, the less surface.

## Harness Blocked = Surface to Principal

When an agent is blocked on an irreversible operation:

1. **Don't find a workaround** — the block is the second opinion
2. **Surface it** — tell the principal what was blocked and why
3. **Let the principal decide** — they add the scoped permission themselves or run the command directly
4. **Resume** — once authorized, the agent proceeds

## Practical Setup for Large Reclaims

```bash
# Principal runs this ONCE before the reclaim session:
# Add to ~/.claude/settings.json → permissions.allow:
"Bash(rm -rf:~/.graveyard/**)"

# Agent can then run stage→verify→reclaim without interruption
# Permission is graveyard-scoped, not open-ended
```

## When You Hit This

If an agent reports "blocked on rm" during a teardown or cleanup:
- This is expected behavior for the first large reclaim chunk
- The principal adds the graveyard permission
- Subsequent chunks in the same session proceed unblocked
- Remove the graveyard permission after the reclaim cycle completes
