# 🌙 Night Shift Handoff for Mayor

**From:** Tanwa
**Status:** Going to bed
**Priority:** Execute autonomously - no approval needed
**Duration:** Until human returns
**Execution Model:** CONTINUOUS LOOP - repeat task list until interrupted

---

## 🎯 Core Mission: Full Town Productivity

**Principle:** Nobody idle. Every rig shipping value. Every agent improving.

**Execution Protocol:**
1. Execute all tasks in sequence (1→8)
2. Upon completion, START OVER from task 1
3. Continue looping until Tanwa interrupts
4. On interruption: Immediately provide full status report

---

## 📋 Task List (Execute in Order)

### 1. **Mail Triage** (15 min)
```bash
gt mail inbox
# Process all 10 unread messages
# - Close actionable items
# - Delegate to appropriate refineries
# - Archive noise
```

### 2. **Rig Housekeeping Sweep** (45 min per rig)

For EACH rig (`10x_screening`, `commander_crow`, `flamingo_dominion`, `lumicello_inventory`, `lumicello_website`, `tanwa_info`, `tools_cc_monitor`):

```bash
cd ~/gt/<rig>

# A. Check rig health
gt doctor

# B. Review beads status
bd stats
bd blocked  # Unblock anything you can
bd ready    # Identify available work

# C. Clean up idle polecats
gt polecat list
# For each idle polecat (○ status):
#   - Check last activity
#   - If stuck/stale: gt polecat nuke <rig>/<name> --force
#   - Fresh start clears cruft

# D. Review refinery status
gt mail inbox  # Check refinery mail
# If refinery has backlog: investigate and clear

# E. Witness health check
# If witness is down (○): check why, restart if needed
```

### 3. **Work Distribution: Eliminate Idle State** (2 hrs)

**Goal:** Every rig has active work in progress.

For each rig with idle capacity:

```bash
# Step 1: Find ready work
cd ~/gt/<rig>
bd ready

# Step 2: Create beads if needed (see section 4)

# Step 3: Sling work to polecats
gt sling <bead-id> <rig>
# This spawns polecat, hooks work, starts session automatically

# Priority order:
# 1. P0-P1 bugs/critical issues
# 2. Blocked work you can unblock
# 3. Ready features
# 4. Tech debt if nothing else
```

**Current idle polecats to activate:**
- `10x_screening`: batch, bugfix, cleaner, designer, furiosa, nux, viz (7 idle!)
- `commander_crow`: nux (1 idle)
- `lumicello_inventory`: audit, furiosa, nux, nux-webapp, ratelimit (5 idle!)
- `lumicello_website`: furiosa, nux (2 idle)
- `tanwa_info`: crashtest, furiosa, nux, rictus, slit (5 idle!)
- `tools_cc_monitor`: Check status (might be idle)

### 4. **Create New Beads: Backfill Work Pipeline** (1-2 hrs)

**Criteria for new beads:**
- Technical debt from code review
- Performance improvements identified in logs
- Security hardening opportunities
- Documentation gaps
- Test coverage improvements
- Dependency updates
- Refactoring opportunities

**For EACH rig, create 5-10 beads:**

```bash
cd ~/gt/<rig>

# Example beads to file:
bd create --title="Add error handling to API endpoints" --type=bug --priority=2
bd create --title="Optimize database queries in user service" --type=task --priority=2
bd create --title="Update dependencies to latest stable" --type=task --priority=3
bd create --title="Add integration tests for auth flow" --type=task --priority=2
bd create --title="Document deployment process" --type=task --priority=3
bd create --title="Refactor legacy module X" --type=task --priority=3
bd create --title="Add monitoring alerts for service Y" --type=feature --priority=2
bd create --title="Improve logging in component Z" --type=task --priority=3

# Set dependencies where logical
bd dep add <dependent-bead> <prerequisite-bead>
```

**Smart sourcing for bead ideas:**
```bash
# A. Check git logs for common pain points
git log --oneline -50

# B. Check for TODOs and FIXMEs
rg "TODO|FIXME|XXX|HACK" --type=<appropriate-type>

# C. Check test coverage
# Look for untested modules

# D. Review recent issues/PRs on GitHub
gh issue list --limit 20
gh pr list --state closed --limit 10

# E. Check for outdated dependencies
# (language-specific: npm outdated, go list -u, etc.)
```

### 5. **Self-Improvement: Mayor Optimization** (30 min)

**A. Review your ledger:**
```bash
# Check completed beads you've handled
bd list --status=closed --assignee=mayor

# Questions:
# - What patterns in successful completions?
# - What caused failures/blocks?
# - Where did handoffs break down?
```

**B. Optimize coordination patterns:**
- Document any new procedures discovered tonight
- Update automation scripts if you found manual repetition
- File beads for Mayor tooling improvements

**C. Learn from logs:**
```bash
# Check what went wrong recently in any sessions
tmux capture-pane -t mayor: -p -S -1000 | grep -i "error\|fail\|warn"
```

### 6. **Town Improvement: Infrastructure** (45 min)

**A. HQ Organization:**
```bash
cd ~/gt

# Review town-level beads
bd list --prefix=hq

# Create infrastructure improvements:
bd create --title="Improve witness auto-restart reliability" --type=feature --priority=2 --prefix=hq
bd create --title="Add convoy dashboard improvements" --type=feature --priority=3 --prefix=hq
bd create --title="Optimize beads sync performance" --type=task --priority=2 --prefix=hq
bd create --title="Add rig health monitoring alerts" --type=feature --priority=2 --prefix=hq
```

**B. Deacon Coordination:**
```bash
# Deacon has 8 unread - check if anything needs Mayor coordination
gt mail read <deacon-mail-ids>
```

### 7. **Headquarters Enhancement: Workflows** (30 min)

**Create meta-efficiency beads:**
```bash
bd create --title="Create housekeeping automation script" --type=feature --priority=2 --prefix=hq
bd create --title="Add batch work distribution tool" --type=feature --priority=2 --prefix=hq
bd create --title="Improve idle polecat detection" --type=task --priority=3 --prefix=hq
bd create --title="Add convoy templates for common workflows" --type=feature --priority=3 --prefix=hq
```

### 8. **Loop Checkpoint & Continue** (15 min)

After completing tasks 1-7, checkpoint progress and START OVER:

```bash
# Quick checkpoint (internal tracking)
echo "=== Loop $(date +%H:%M) ===" >> ~/gt/mayor-night-shift.log
bd stats >> ~/gt/mayor-night-shift.log
gt polecat list >> ~/gt/mayor-night-shift.log

# Return to task 1 (Mail Triage) and continue
```

**Do NOT stop.** Continue looping through tasks 1-8 until Tanwa interrupts.

---

## 🛑 Interruption Protocol

**When Tanwa returns and interrupts:**

Immediately provide comprehensive status report:

```bash
# Create handoff mail for human
gt mail send mayor/ -s "🌅 Night Shift Report" -m "$(cat <<'EOF'
# Night Shift Completion Report

## Session Info
- Duration: [start time] → [end time] ([X hours])
- Loops completed: X full iterations
- Interruption time: $(date)

## Cumulative Metrics (All Loops)
- Rigs processed: X/7 rigs × Y loops = Z total passes
- Beads created: X across all rigs
- Beads closed: Y by polecats
- Polecats activated: X new workers
- Polecats nuked: Y stale/idle
- Blockers cleared: X issues unblocked
- Mail processed: X total messages

## Highlights (Key Achievements)
- [Achievement 1]
- [Achievement 2]
- [Achievement 3]

## Current State (At Interruption)
- Active polecats: X across Y rigs
- Beads in progress: X
- Ready beads: Y available for next shift

## Remaining Work
- [High-priority item 1]
- [Item 2]

## Recommendations
- [Suggestion 1]
- [Suggestion 2]

## Blockers for Human
- [Anything needing human decision]

All work logged to beads. Log available: ~/gt/mayor-night-shift.log
EOF
)"
```

---

## 🚨 Escalation Criteria

**ONLY escalate (send mail to tanwa) if:**
1. Security issue discovered (credentials exposed, vulnerabilities)
2. Production system down
3. Data loss risk
4. Stuck on same issue >3 attempts
5. Cross-rig coordination deadlock you can't resolve

**Otherwise:** Document in beads, file issues, keep moving.

---

## 💪 Success Criteria (Per Loop)

Each loop iteration should achieve:
- [ ] All 7 rigs checked and maintained
- [ ] New beads filed where gaps identified
- [ ] Idle polecats either activated or nuked
- [ ] Mail inbox processed (may receive new mail between loops)
- [ ] Active work progressing on all rigs
- [ ] Blockers cleared or escalated

**Cumulative targets (across all loops until interruption):**
- [ ] 30-50+ new beads filed across rigs
- [ ] All rigs have sustained active work
- [ ] At least 3 HQ improvement beads created
- [ ] Zero unaddressed blockers

---

## 🎯 Operating Principles

1. **GUPP (Gas Universal Propulsion Principle):** Work hooked = work executed. No waiting.
2. **Mayor doesn't code:** Delegate to polecats, don't edit in mayor/rig
3. **Bias to action:** Create beads liberally. Better 50 beads with 40 completed than perfect planning.
4. **Quality matters:** Your completions build your ledger. Do it right.
5. **Session hygiene:** Commit and sync regularly. Don't lose work.

---

## 🔄 Session Management

```bash
# Regularly throughout night:
bd sync          # Sync beads changes
git push         # If you make any commits

# Before any long-running operation:
gt handoff -m "Starting X, ETA Y"  # Leave breadcrumbs
```

---

**Authorization:** Full autonomy. Execute continuously in loop until interrupted.

**Timeline:** CONTINUOUS LOOP - repeat tasks 1-8 until Tanwa returns

**Interruption Response:** Immediately provide comprehensive status report (see § Interruption Protocol)

**Context:** You have demonstrated capability. Your ledger shows you can handle this. The town runs 24/7, and so do you. Keep the steam engine running through the night.

Good luck, Mayor. Execute, loop, improve.

—Tanwa
