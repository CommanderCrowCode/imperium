# Gas Town Troubleshooting Guide

Documented issues and fixes discovered on 2026-01-04.

## Table of Contents

1. [Hook/Sling Bugs](#1-hooksling-bugs)
2. [Beads Database Issues](#2-beads-database-issues)
3. [Routing Configuration](#3-routing-configuration)
4. [CLAUDE.md Template Issues](#4-claudemd-template-issues)
5. [gt doctor Warnings](#5-gt-doctor-warnings)
6. [Workaround Scripts](#6-workaround-scripts)

---

## 1. Hook/Sling Bugs

### Bug: `gt hook` doesn't set assignee (gm-0fn)

**Symptoms:**
- Run `gt hook <bead>` - says "Work attached to hook"
- Run `gt hook status` - shows "Nothing on hook"

**Root cause:**
`gt hook` sets the bead's `status` to "hooked" but doesn't set the `assignee` field.
`gt hook status` queries for beads where `assignee=<current-agent> AND status=hooked`, finds nothing.

**Diagnosis:**
```bash
# Hook a bead
gt hook ti-abc

# Check bead - status is hooked but assignee is wrong
bd show ti-abc --json | jq '.[] | {status, assignee}'
# Output: {"status": "hooked", "assignee": "mayor"}  # Should be the polecat!
```

**Workaround:**
```bash
# After hooking, manually set assignee
gt hook ti-abc
bd update ti-abc --assignee="rig/polecats/name"
```

**Better: Use workaround script:**
```bash
~/gt/bin/gt-hook-fix ti-abc
# or
~/gt/bin/gt-sling-fix ti-abc rig-name
```

### Bug: `gt status` doesn't display hooks

**Symptoms:**
- `gt hook status` shows `has_work: true` with the bead
- `gt status` shows `hook: (none)` for that polecat

**Root cause:**
`gt status` reads hook info from agent bead description field (`hook_bead:`), not from bead query.
Even when agent bead has `hook_bead: xxx` in description, parsing may fail.

**Impact:** Cosmetic only. Polecats still find their work on startup via `gt hook status`.

---

## 2. Beads Database Issues

### Issue: Circular redirect in .beads

**Symptoms:**
```
Warning: redirect chains not allowed, ignoring redirect in .beads
Error: no issue found matching "xxx"
```

**Diagnosis:**
```bash
# Check for self-referential redirect
cat /path/to/rig/mayor/rig/.beads/redirect
# If it says "../../mayor/rig/.beads" - that's a circular reference!
```

**Fix:**
```bash
# Remove the broken redirect (if actual DB is in same directory)
rm /path/to/rig/mayor/rig/.beads/redirect
```

### Issue: Database not initialized

**Symptoms:**
```
Error: database not initialized: issue_prefix config is missing
```

**Fix:**
```bash
cd /path/to/rig/mayor/rig
bd init --prefix=XX
```

### Issue: Prefix mismatch

**Symptoms:**
```
Error: prefix mismatch: database uses 'X' but you specified 'Y'
```

**Diagnosis:**
```bash
# Check what prefix the database uses
cd /path/to/rig/mayor/rig
bd list | head -1  # Look at the prefix on bead IDs
```

**Fix:** Either:
1. Update routes.jsonl to match actual prefix
2. Or use `--force` flag (not recommended)

### Issue: Multiple databases in same rig

**Symptoms:**
Beads created in one location not visible from another.

**Common scenario:**
- `crew/tanwa/.beads` has full database
- `mayor/rig/.beads` has different database
- `polecats/furiosa/.beads` redirects to wrong one

**Diagnosis:**
```bash
# Check where each location points
cat /path/crew/tanwa/.beads/redirect || echo "No redirect - local DB"
cat /path/mayor/rig/.beads/redirect || echo "No redirect - local DB"
cat /path/polecats/furiosa/.beads/redirect
```

**Fix:** Ensure all worktrees redirect to the same canonical database:
```bash
# For polecats, typically redirect to mayor/rig or crew/tanwa
echo "../../mayor/rig/.beads" > /path/polecats/furiosa/.beads/redirect
```

---

## 3. Routing Configuration

Routes are defined in `~/gt/.beads/routes.jsonl`:

```json
{"prefix":"ti-","path":"tanwa_info/crew/tanwa"}
{"prefix":"ds-","path":"10x_screening/mayor/rig"}
{"prefix":"inventory_management-","path":"lumicello_inventory/mayor/rig"}
{"prefix":"fd-","path":"flamingo_dominion/mayor/rig"}
```

### Verify routing works

```bash
# Debug routing for a specific bead
BD_DEBUG_ROUTING=1 bd show ti-abc

# Test each prefix
bd show ti-xxx 2>&1 | head -2
bd show ds-xxx 2>&1 | head -2
```

### Fix route prefix mismatch

```bash
# Edit routes.jsonl directly
# Change "wrong-" to "correct-"
cat ~/gt/.beads/routes.jsonl | sed 's/"wrong-"/"correct-"/' > /tmp/routes.jsonl
mv /tmp/routes.jsonl ~/gt/.beads/routes.jsonl
```

---

## 4. CLAUDE.md Template Issues

### Issue: Rig has Mayor-context CLAUDE.md

**Symptoms:**
Crew/polecat session reads CLAUDE.md and thinks it's the Mayor.

**Diagnosis:**
```bash
head -5 /path/to/rig/crew/tanwa/CLAUDE.md
# If it says "# Mayor Context" - wrong template!
```

**Fix:**
Create project-specific CLAUDE.md with:
- Project overview
- Common tasks and commands
- Key files and directory structure
- Development workflow

**Track via bead:**
```bash
bd create --title="Replace Mayor-context CLAUDE.md with project-specific instructions" \
  --type=task --priority=2
```

---

## 5. gt doctor Warnings

### Missing agent beads

**Symptoms:**
```
✗ agent-beads-exist: X agent bead(s) missing
```

**Fix:**
```bash
# Create missing agent beads
bd create --id=PREFIX-RIG-witness --type=agent --title="Witness for RIG"
bd create --id=PREFIX-RIG-refinery --type=agent --title="Refinery for RIG"
bd create --id=PREFIX-RIG-crew-NAME --type=agent --title="Crew NAME for RIG"
```

### Orphaned Claude processes

**Fix:**
```bash
gt doctor --fix
# Or manually kill
kill PID
```

### Missing role prompt templates

**Symptoms:**
```
⚠ patrol-roles-have-prompts: X role prompt template(s) missing
```

Templates should be in `internal/templates/roles/`:
- deacon.md.tmpl
- witness.md.tmpl
- refinery.md.tmpl

### Missing .runtime gitignore

**Fix:**
```bash
echo '.runtime/' >> /path/to/rig/crew/tanwa/.gitignore
```

---

## 6. Workaround Scripts

Located in `~/gt/bin/`:

### gt-sling-fix

Properly slings beads to polecats with all required fields.

```bash
~/gt/bin/gt-sling-fix <bead-id> <rig> [polecat-name]

# Examples:
~/gt/bin/gt-sling-fix ds-ctz 10x_screening
~/gt/bin/gt-sling-fix ti-abc tanwa_info furiosa
```

**What it does:**
1. Creates polecat worktree if missing
2. Creates beads redirect if missing
3. Creates agent bead if missing
4. Unhooks any existing beads from that polecat
5. Hooks the new bead
6. Sets `assignee` field correctly
7. Updates agent bead description with `hook_bead`

### gt-hook-fix

Hooks beads from within a worker directory with correct assignee.

```bash
cd /path/to/rig/polecats/name
~/gt/bin/gt-hook-fix <bead-id>
```

---

## Quick Health Check

Run this to verify Gas Town health:

```bash
# 1. Check gt doctor
gt doctor

# 2. Verify routes
cat ~/gt/.beads/routes.jsonl

# 3. Test each rig's beads
for rig in tanwa_info 10x_screening lumicello_inventory flamingo_dominion; do
  echo "=== $rig ==="
  cd ~/gt/$rig/mayor/rig 2>/dev/null && bd list --status=open 2>&1 | head -3 || echo "No beads"
done

# 4. Check polecat hooks
for dir in ~/gt/*/polecats/*/; do
  [ -d "$dir" ] && (cd "$dir" && echo "=== $dir ===" && gt hook status --json | jq '{has_work, bead: .pinned_bead.id}' 2>/dev/null)
done
```

---

## HQ Beads for Tracking

| ID | Priority | Description |
|----|----------|-------------|
| gm-0fn | P1 | gt hook doesn't set assignee - breaks hook lookup |
| gm-yds | P3 | Check if workarounds still needed after gt upgrade |
| gm-y06 | P2 | Add missing role prompt templates |
| gm-3fi | P2 | Initialize daemon config for patrol hooks |
| gm-prr | P3 | Add .runtime/ to crew gitignore files |
| gm-atq | P3 | Kill orphaned Claude processes |

---

*Last updated: 2026-01-04*
