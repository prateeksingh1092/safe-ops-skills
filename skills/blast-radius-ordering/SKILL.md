---
name: blast-radius-ordering
description: Use when planning a sequence of risky or irreversible operations — order by impact scope so failures are recoverable and the principal sees low-risk work before high-risk work runs unattended.
---

# Blast Radius Ordering

## Core Rule

**Lowest blast radius first. Highest last.** A failure in chunk 1 doesn't block chunk 2; a failure in chunk 8 has already proven chunks 1-7 are safe.

## Blast Radius Tiers

| Tier | Characteristics | Examples |
|---|---|---|
| **1 — Isolated/dormant** | Already inactive, no live callers, easily reversed | Dormant hook files, unused dirs, orphaned configs |
| **2 — Active but low coupling** | Live but few dependents, single-file scope | Unwired hooks, standalone scripts, log files |
| **3 — Wired/registered** | Has live registrations; deregister + rename required | Registered hooks, cron jobs, active aliases |
| **4 — Session-control** | Affects current session or startup; wrong order breaks context | CLAUDE.md, ISA.md, system prompt files |
| **5 — Session-fatal** | Malformed = tool won't launch; external recovery required | settings.json, shell rc files with PATH |
| **6 — Irreversible** | No undo; data permanently gone | Mass rm, secret deletion, large reclaims |

## Planning Protocol

1. List all operations
2. Assign blast radius tier to each
3. Sort by tier ascending
4. Add probe gates between tier jumps
5. Get principal review before tier 5-6

## Calibration Set

Run the first 2-3 lowest-risk chunks **with principal reviewing each result** before switching to unattended run-through. This proves your mechanic works and builds trust before you're in session-fatal territory.

## Gate High-Blast Steps Explicitly

Before any tier 5-6 chunk:
- [ ] External snapshot taken and verified
- [ ] Recovery one-liner pasted into active chat
- [ ] Principal has explicitly green-lit this specific chunk
- [ ] Single-step execution (not batched with other chunks)

## Rollback Structure

Blast-radius ordering also structures rollback:
- Tier 1-2 chunks revert independently
- Tier 3-4 chunks require coordinated undo (restore registration + rename)
- Tier 5-6 chunks require full external restore; no partial undo

**Label each chunk with blast radius estimate before proposing it.** Principals make better decisions when they can see the scope.
