# Baseline State: Graceful Degradation Test (gm-vhd)

**Test Start:** 2026-01-10 08:30
**Tester:** deacon/dogs/boot

## Current System State (Baseline)

### Deacon
- **Session:** `gt-deacon-dogs` - RUNNING
- **Process:** Active (PID 48470)
- **Location:** `/Users/tanwa/gt/deacon/dogs/boot`

### Witnesses
Multiple witness sessions detected:
- `gt-flamingo_dominion-witness` - RUNNING (PID 43068)
- Additional witness processes for other rigs (PIDs: 57671, 57663, 57655, 57648, 57640)

### Active Polecats (Sample: flamingo_dominion)
```
Active Polecats in flamingo_dominion:
  ● flamingo_dominion/furiosa  done
  ● flamingo_dominion/gather_ds  done
  ● flamingo_dominion/gather_li  done
  ● flamingo_dominion/gather_ti  done
  ● flamingo_dominion/nux  done
  ● flamingo_dominion/slit  working (gm-r8vr)
```

**Active work:** flamingo_dominion/slit working on gm-r8vr

### Total Active Sessions
```
gt-10x_screening: 14 sessions (polecats + refinery)
gt-commander_crow: 2 sessions
gt-deacon: 2 sessions (boot + dogs)
gt-flamingo_dominion: 7+ sessions (polecats + witness)
... (additional rigs)
```

## Test Plan

### Test 1: Stop Deacon, verify Witnesses still work
**Hypothesis:** Witnesses should continue patrol and worker management even without Deacon

**Steps:**
1. Stop gt-deacon-dogs session
2. Monitor gt-flamingo_dominion-witness for continued activity
3. Verify witness can still detect stuck polecats
4. Verify witness patrol continues

### Test 2: Stop Witness, verify Polecats complete work
**Hypothesis:** Polecats should complete their hooked work even without Witness monitoring

**Steps:**
1. Stop gt-flamingo_dominion-witness session
2. Monitor flamingo_dominion/slit polecat (working on gm-r8vr)
3. Verify polecat completes work independently
4. Check bead status updates

### Test 3: Run in no-tmux mode
**Hypothesis:** Workers can run in naked Claude Code sessions without tmux infrastructure

**Steps:**
1. Create test bead
2. Start Claude session manually (no tmux)
3. Hook bead and verify execution
4. Verify gt prime and basic commands work

---

## Expected Graceful Degradation Behaviors

Per Gas Town spec:
✓ Every worker can do their job independently or in groups
✓ Can choose which parts of Gas Town to run
✓ Works in 'no-tmux' mode with naked Claude Code sessions
✓ System limps along without real-time messages
