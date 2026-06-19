---
name: fresh-probe-verification
description: Use when verifying Claude Code hook, settings, CLAUDE.md, or system-prompt changes — in-session verification silently lies because changes only take effect on the next session.
---

# Fresh Probe Verification

## The Core Fact

Changes to `settings.json`, `CLAUDE.md`, hooks, and the system prompt take effect on the **next** Claude session — not the current one. Verifying in the current session always passes even if the change is broken or completely missing.

**This is the most common verification mistake in Claude Code customization.**

## The Probe Mechanic

```bash
# Always in a fresh scratch dir (not the project dir)
claude -p "your verification question" -d /tmp
```

`-d /tmp` ensures no project-level `CLAUDE.md` or settings bleed in.
`-p` gives a one-shot non-interactive probe.

## What to Verify

Verify **absence** AND **presence** in the same probe:

```bash
# Example: after removing PAI skills
claude -p "Do you know a skill called SkillName? List your skills." -d /tmp

# Check for:
# ABSENCE: PAI skills/agents/hooks not present
# PRESENCE: stock behavior intact, expected skills still work
```

## When to Probe

| Operation | Probe required |
|---|---|
| Any `settings.json` edit | Yes — after each session-fatal chunk |
| Hook registration change | Yes |
| `CLAUDE.md` change | Yes |
| Skill/agent rename or remove | Yes |
| Non-config code changes | No |

Never run two session-fatal chunks back-to-back without a probe between.

## Reference Check Before Renaming

Before renaming/removing a file, check for **executable callers** (not just string matches in docs):

```bash
# Check: scripts, JSON config, plists, crontabs — not docs
grep -r "filename.hook.ts" ~/.claude ~/.claude/settings.json ~/Library/LaunchAgents
crontab -l | grep "filename"
```

Docs referencing the old name are fine to leave dangling. Executable callers must be updated first.

## Reversibility Proof

Before any unattended multi-chunk run: prove one chunk reverses cleanly. Disable → probe (confirms off) → re-enable → probe (confirms on) → re-disable → continue. If reversal breaks, fix it before proceeding.
