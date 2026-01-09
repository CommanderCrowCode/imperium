# Polecat Crash Recovery Verification Report

**Test ID:** gm-5tc
**Date:** 2026-01-09 22:00
**Tester:** deacon/dogs/boot
**Status:** ⚠️ INCONCLUSIVE - Feature Not Operational

## Expected Behavior

Per Gas Town spec:
- Polecat workflow state persists in molecule
- If polecat crashes mid-workflow, new polecat picks up from last checkpoint
- No work is lost on crash

## Test Approach

1. ✅ Created multi-step workflow formula (`crash-test-workflow`)
   - 4 steps with dependencies: init → middle → final → complete
   - Each step creates marker files for verification
   - Formula verified: `/Users/tanwa/gt/.beads/formulas/crash-test-workflow.formula.toml`

2. ✅ Created test bead (`ti-yd3`) in tanwa_info rig

3. ✅ Added new polecat (`crashtest`) to tanwa_info rig
   - Used `gt polecat add tanwa_info crashtest`
   - Worktree created successfully

4. ✅ Slung bead to polecat using `~/gt/bin/gt-sling-fix`
   - Bead hooked: `ti-yd3` → `tanwa_info/polecats/crashtest`
   - Session spawned: `gt-tanwa_info-crashtest`
   - Polecat auto-executed via GUPP

5. ❌ Molecule attachment failed
   - No molecule was attached to the hooked bead
   - Polecat ran `gt prime` but had no workflow formula
   - Database sync issues encountered

6. ✅ Killed polecat session mid-workflow

## Key Findings

### Molecules Not Operational

**Evidence:**
- No existing molecules found in Gas Town: `find /Users/tanwa/gt -name "*.molecule*"` returned nothing
- `gt formula list` shows formulas exist
- `gt mol` commands exist in CLI
- **BUT**: No automatic molecule attachment when slinging beads
- No mechanism observed for slinging bead + formula together

**Infrastructure Issues Encountered:**
- Polecat's `gt hook` initially showed wrong agent (mayor instead of crashtest)
- Fixed by running from correct directory: `/Users/tanwa/gt/tanwa_info/polecats/crashtest`
- Database sync errors: "Database out of sync with JSONL"
- No database store available for inline import

### What Works

✅ **Formula system:**
- Can create formulas with `gt formula create`
- Formulas properly defined with steps and dependencies
- Formula stored in `.beads/formulas/`

✅ **Polecat spawning:**
- Polecats can be created with `gt polecat add`
- Git worktrees created correctly
- Sessions spawn and auto-execute (GUPP fix working)

✅ **Bead slinging:**
- Beads can be hooked to polecats
- `gt-sling-fix` properly sets assignee field
- Hook file created: `.gt-hook` contains bead ID

### What Doesn't Work

❌ **Molecule lifecycle:**
- No automatic molecule creation when slinging work
- No way to sling "bead + formula" together observed
- No molecule state persistence seen
- No checkpoint/resume behavior observable

❌ **Crash recovery:**
- Cannot test crash recovery without operational molecules
- No state to persist beyond bead ID in `.gt-hook`
- Killing polecat loses all workflow progress (no checkpoints to resume from)

## Verification Result

**⚠️ INCONCLUSIVE**

Cannot verify polecat crash recovery because the prerequisite molecule system is not operational. The test revealed:

1. **Formulas work** - Can define multi-step workflows
2. **Polecats work** - Can spawn, hook beads, execute
3. **Molecules don't work** - No mechanism for attaching formulas to beads as molecules
4. **Crash recovery untestable** - Requires operational molecules to have state to resume from

## Recommendations

1. **Implement molecule attachment** when slinging beads:
   ```bash
   gt sling <bead-id> <target> --formula=<formula-name>
   ```

2. **Auto-attach molecules** for beads with certain labels/types

3. **Molecule persistence** - Implement checkpointing system that:
   - Saves completed step IDs to molecule state file
   - Allows resume from last completed step
   - Survives polecat crashes

4. **Fix database sync issues** - Polecats hitting sync errors on startup

5. **Document molecule workflow** - No clear docs on how to use molecules in practice

## Next Steps

This bead (gm-5tc) should be marked as **BLOCKED** until molecule system is operational. Recommend creating new beads for:

- `gm-XXX: Implement molecule attachment when slinging beads`
- `gm-YYY: Implement molecule checkpoint/resume system`
- `gm-ZZZ: Document molecule workflow usage`

Once those are complete, this verification test can be re-run.

## Test Artifacts

- Formula: `/Users/tanwa/gt/.beads/formulas/crash-test-workflow.formula.toml`
- Test bead: `ti-yd3`
- Polecat: `tanwa_info/polecats/crashtest` (session killed)
- Report: `/Users/tanwa/gt/deacon/dogs/boot/.qa-reports/2026-01-09_220000_polecat-crash-recovery.md`
