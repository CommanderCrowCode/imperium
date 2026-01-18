# PROTOCOLS.md Implementation Guide

> **For Overseer: How to inject universal protocols into Gas Town agents**

---

## Overview

`PROTOCOLS.md` contains universal operational protocols learned from psu_teaching crew. These need to be injected via hooks so all agents receive them automatically.

---

## Implementation Strategy (Option 3: Both)

### 1. Human Reference (✅ DONE)
- Created `/Users/tanwa/gt/PROTOCOLS.md`
- Serves as canonical reference for humans
- Documents all patterns in one place

### 2. Hook Injection (⏳ TODO)
Inject relevant sections via startup hooks so agents receive protocols automatically.

---

## Hook Injection Plan

### A. Universal Injection (All Agents)

**Target:** `~/.config/claude-code/hooks/user-prompt-submit.sh`

**Content to inject:**
```markdown
## 🔄 Communication Protocol (MANDATORY)

When sending mail expecting action/response:
1. Send mail (audit trail)
2. Verify session exists: tmux ls | grep <session>
3. Send tmux notification (CRITICAL - agents don't auto-poll):
   tmux send-keys -t <session> "New mail from <you>. Check: gt mail inbox"
   tmux send-keys -t <session> Enter
4. Verify notification sent

## 📋 Session Completion Checklist (MANDATORY)

[ ] File remaining work as beads
[ ] Run quality gates (if code changed)
[ ] Update bead status
[ ] Push to remote (git pull --rebase, bd sync, git push)
[ ] Verify tmux notifications sent (if mail sent)
[ ] Clean up (stashes, branches)
[ ] Hand off context (if incomplete)
```

**Why universal:**
- All agents (Mayor, Witness, Refinery, Polecats, Crew) send mail
- All agents need session completion discipline
- Prevents protocol divergence across agent types

---

### B. Coordinator-Specific Injection (Mayor, Witness, Refinery)

**Target:** Role-specific CLAUDE.md files
- `~/gt/CLAUDE.md` (Mayor)
- `<rig>/witness/CLAUDE.md` (if exists)
- `<rig>/refinery/CLAUDE.md` (if exists)

**Additional content:**
```markdown
## Session Naming Conventions (Reference)

[Include full table from PROTOCOLS.md]

## Blocker Escalation Format

[Include template from PROTOCOLS.md]
```

**Why coordinator-specific:**
- Coordinators need session naming reference (they send to many agents)
- Coordinators handle escalations more frequently
- Workers can reference PROTOCOLS.md if needed

---

### C. Rig-Specific Injection (psu_teaching example)

**Target:** `<rig>/.claude/hooks/user-prompt-submit.sh` (if rig needs custom patterns)

**Content:**
```markdown
## Session Names for This Rig

[Include rig-specific table from PROTOCOLS.md]
```

**Why rig-specific:**
- Each rig has different crew/polecat names
- Prevents copy-paste errors when notifying agents
- Can be auto-generated from `gt polecat list` + `gt crew list`

---

## Current Gaps Analysis

| Protocol | Current Status | After Implementation |
|----------|----------------|---------------------|
| **Communication (mail+tmux)** | ✅ Documented in Mayor CLAUDE.md | ✅ Injected for all agents |
| **Session Completion** | ⚠️ Partial in beads hooks | ✅ Enforced universally |
| **Session Naming** | ❌ Not documented | ✅ Reference available |
| **Blocker Escalation** | ❌ Ad-hoc | ✅ Structured template |
| **Commit Messages** | ❌ Inconsistent | ✅ Guidelines available |

---

## Implementation Steps

### Step 1: Update Universal Hook
```bash
# Edit ~/.config/claude-code/hooks/user-prompt-submit.sh
# Add Communication + Session Completion sections from PROTOCOLS.md
```

**Test:**
- Spawn new polecat → Verify sees protocols in startup context
- Check Mayor session → Verify protocols injected

### Step 2: Update Mayor CLAUDE.md
```bash
# Edit ~/gt/CLAUDE.md
# Add reference to PROTOCOLS.md
# Include session naming table
# Include blocker escalation template
```

**Test:**
- Restart Mayor → Verify sees updated context
- Check session completion behavior

### Step 3: Create Rig Hook Template
```bash
# Create template: ~/gt/bin/generate-rig-session-names
# Generates session name table from `gt polecat list` + `gt crew list`
# Output can be injected into rig-specific hooks
```

**Test:**
- Run for psu_teaching → Compare with planner's TMUX_SESSION_NAMES.md
- Run for 10x_screening → Verify correct format

### Step 4: Document in Git
```bash
cd ~/gt
git add PROTOCOLS.md PROTOCOLS_IMPLEMENTATION.md
git commit -m "docs: Add universal Gas Town operational protocols

- Communication protocol (mail + tmux notification)
- Session completion checklist
- Session naming conventions
- Blocker escalation format
- Commit message guidelines

Based on psu_teaching crew best practices.
Implements resilience-by-design principles."
git push
```

---

## Testing Protocol

After implementation, verify each protocol works:

### Test 1: Communication Protocol
```bash
# As Mayor, send mail to polecat
gt mail send 10x_screening/polecats/furiosa -s "Test" -m "Protocol test"

# Verify you're reminded to send tmux notification
# Send notification
tmux send-keys -t gt-10x_screening-furiosa "New mail from mayor. Check: gt mail inbox"
tmux send-keys -t gt-10x_screening-furiosa Enter

# Check polecat received notification
# (should see prompt update or response)
```

### Test 2: Session Completion
```bash
# As any agent, make a change
echo "test" > /tmp/test.txt

# Try to end session
# Verify checklist prompts you to:
# - File remaining work
# - Push changes
# - Verify notifications
```

### Test 3: Session Naming
```bash
# As any agent, check if session name reference is available
# Should be able to lookup correct tmux session for any rig agent
```

---

## Rollout Plan

**Phase 1: Mayor (Immediate)**
- Update ~/gt/CLAUDE.md with PROTOCOLS.md reference
- Test Mayor sessions follow protocols

**Phase 2: Universal Hook (Week 1)**
- Update ~/.config/claude-code/hooks/user-prompt-submit.sh
- Test with 2-3 polecats across different rigs
- Verify no regressions

**Phase 3: Rig-Specific (Week 2)**
- Create session name generation script
- Add rig-specific hooks where needed
- Document in each rig's README

**Phase 4: Enforcement (Week 3)**
- Add protocol checks to `gt doctor`
- Flag violations (e.g., mail sent without tmux notification)
- Report compliance metrics

---

## Success Metrics

**After 1 week:**
- [ ] Zero "I sent mail but agent didn't respond" incidents
- [ ] 100% of sessions end with git push
- [ ] Zero tmux session naming errors

**After 1 month:**
- [ ] All agents demonstrate protocol compliance
- [ ] Protocol violations caught by `gt doctor`
- [ ] New agent spawns receive protocols automatically

---

## Maintenance

**Protocol updates:**
1. Update PROTOCOLS.md (source of truth)
2. Regenerate hook injections
3. Test with sample agents
4. Document in git commit

**Adding new rig:**
1. Run session name generator
2. Add rig-specific hook if needed
3. Update PROTOCOLS.md with rig reference

---

## Questions for Overseer

1. **Hook location preference:**
   - Option A: Single universal hook (~/.config/claude-code/hooks/)
   - Option B: Rig-specific hooks (<rig>/.claude/hooks/)
   - Option C: Both (universal + rig-specific supplements)

2. **Enforcement level:**
   - Soft: Guidelines only (current)
   - Medium: Warnings in context
   - Hard: Block actions (e.g., can't end session without push)

3. **Rollout pace:**
   - Immediate: All agents now
   - Gradual: Mayor → Coordinators → Workers
   - Pilot: One rig first (psu_teaching already follows most protocols)

---

**Status:** Ready for review
**Next:** Awaiting overseer decision on implementation approach
**Created:** 2026-01-18
