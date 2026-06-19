---
name: confidence-gate-before-deletion
description: Use when classifying artifacts as safe to delete or disable — where misidentification would destroy something important that isn't yours to remove.
---

# Confidence Gate Before Deletion

## Rule

**≥2 independent methods of provenance before disabling or deleting anything.** One source can be wrong. Two agreeing independent sources is the minimum gate.

## Provenance Methods

| Method | How |
|---|---|
| **Online source repo** | Find the file in the upstream repo at that version |
| **Bundle byte-match** | SHA256 the local file against the known bundle |
| **File-header content** | Header comment names the system ("PAI Hook System v5.0") |
| **Registration/audit trail** | Settings.json registers it; audit log shows it was installed |
| **Installation manifest** | Installer's own list of what it deployed |
| **Path convention** | File lives exclusively in the overlay's namespace |

**Anything below 100% confidence: flag and skip, never remove.**

## Bundle Completeness Gate

Map every component in the installed bundle to a planned removal step before starting:

```
Bundle component → removal chunk → verify method
LoadContext.hook.ts → Chunk 4 → byte-match v5.0.0 bundle
SecurityPipeline.hook.ts → Chunk 5 → registration + header
...
```

If a component can't be mapped → research more before proceeding. No gaps allowed.

## Confidence Table (document before acting)

| File | Method 1 | Method 2 | Confidence | Action |
|---|---|---|---|---|
| `LoadContext.hook.ts` | bundle match (SHA256 OK) | header "PAI v5.0.0" | 100% | DISABLE |
| `some-other.ts` | not in bundle | no header | <100% | FLAG, SKIP |

The confidence table is a **reviewable artifact** before destructive action — not just an internal note.

## Common Misidentification Traps

| Trap | How it misleads |
|---|---|
| "It's in the hooks dir" | User may have added non-overlay hooks there |
| "It was installed at the same time" | Coincidental timing ≠ overlay origin |
| "The filename looks PAI-ish" | Naming conventions aren't provenance |
| "No one remembers adding it" | Absence of memory ≠ provenance |

## When You're Uncertain

1. Flag it in the audit doc with your uncertainty
2. Ask the principal to confirm
3. Skip it in this pass
4. Document the skip so it can be revisited

A skipped item leaves harmless residue. A misidentified deletion may be unrecoverable.
