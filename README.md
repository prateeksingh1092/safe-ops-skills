# safe-ops-skills

Claude Code plugin — 11 skills for safe irreversible operations.

Distilled from a real production teardown (399GB reclaimed, zero data loss).

## Skills

| Priority | Skill | When to use |
|---|---|---|
| HIGH | `reversible-disable-primitive` | Disabling artifacts before permanent deletion is decided |
| HIGH | `staged-delete-safety` | Irreversible deletions — mass rm, secret removal, large reclaims |
| HIGH | `atomic-config-edits` | Editing JSON config files that gate a tool's ability to launch |
| HIGH | `fresh-probe-verification` | Verifying Claude Code hook/settings/CLAUDE.md changes |
| MED | `classify-before-delete` | Dirs with mixed user data, config, and reconstructible artifacts |
| MED | `blast-radius-ordering` | Sequencing risky operations by impact scope |
| MED | `confidence-gate-before-deletion` | Classifying artifacts as safe to delete (≥2 provenance methods) |
| MED | `external-recovery-anchor` | Risky ops where the tool itself might break |
| LOW | `harness-boundary-awareness` | Claude Code agents in auto-mode doing irreversible mass operations |
| LOW | `executable-caller-check` | Renaming/removing files that might be referenced by scripts/cron/launchd |
| — | `claude-stock-audit` | Verifying/restoring ~/.claude to vanilla stock Claude Code state |

## Install

Add to your `~/.claude/settings.json`:

```json
"extraKnownMarketplaces": {
  "safe-ops-skills": {
    "source": {
      "source": "github",
      "repo": "prateeksingh1092/safe-ops-skills"
    }
  }
}
```

Then enable the plugin in your settings.
