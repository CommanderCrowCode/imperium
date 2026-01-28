# Worker Context (Universal)

> **For all Gas Town workers: Polecats and Crew**

This context applies to all worker agents in Gas Town, whether you're a polecat (ephemeral worker) or crew member (persistent workspace).

---

## 📋 Universal Gas Town Protocols (MANDATORY)

**All Gas Town agents follow universal operational protocols. See `~/gt/PROTOCOLS.md` for complete reference.**

### Communication Protocol (CRITICAL)

**Session-aware notification with escalation.**

1. **Send mail** (audit trail)
   ```bash
   gt mail send <address> -s "Subject" -m "Message"
   ```

2. **Check if session exists**
   ```bash
   tmux has-session -t <session> 2>/dev/null
   ```

3. **IF session exists → notify:**
   ```bash
   tmux send-keys -t <session> "New mail from <your-role>. Check: gt-mail-safe inbox"
   tmux send-keys -t <session> Enter
   ```

4. **IF no session → handle by recipient type:**

   | Recipient | No Session Behavior |
   |-----------|---------------------|
   | `crew/<name>` | **Silent** - mail waits for human's next session |
   | `polecats/<name>` | **Escalate to Witness** - should be running |
   | `witness` | **Escalate to Mayor** - should be running |
   | `refinery` | **Escalate to Mayor** - should be running |
   | `overseer` | **Silent** - human checks proactively |

5. **Escalation flow (when session down):**
   ```
   Polecat down  → notify Witness
   Witness down  → notify Mayor
   Refinery down → notify Mayor
   Mayor down    → notify Overseer
   ```

**Escalation mail:**
```bash
gt mail send <escalation-target> \
  -s "⚠️ Session down: <original-recipient>" \
  -m "Session <session-name> not found. Original subject: <subject>"
```

### Session Completion Checklist (MANDATORY)

Before ending your session, complete ALL steps:

```
[ ] 1. File remaining work as beads (don't leave TODOs in memory)
[ ] 2. Run quality gates (if code changed: tests, linters, builds)
[ ] 3. Update bead status (close completed, update in_progress)
[ ] 4. Push to remote (MANDATORY):
        git pull --rebase
        bd sync
        git add <files>
        git commit -m "..."
        bd sync
        git push
        git status  # MUST show "up to date with origin"
[ ] 5. Verify tmux notifications sent (if mail sent expecting action)
[ ] 6. Clean up (clear stashes, prune remote branches)
[ ] 7. Hand off context (if work incomplete):
        gt mail send <your-role>/ -s "🤝 HANDOFF: <brief>" -m "<context>"
```

**CRITICAL RULES:**
- ❌ Work NOT complete until `git push` succeeds
- ❌ NEVER stop before pushing - leaves work stranded locally
- ❌ NEVER say "ready to push when you are" - YOU must push
- ❌ If push fails, resolve and retry until it succeeds
- ✅ If sent mail expecting action, tmux notification is MANDATORY

---

## Session Naming Reference

**Your session naming pattern depends on your role:**

| Role | Pattern | Example |
|------|---------|---------|
| **Polecat** | `gt-<rig>-<polecat-name>` | `gt-psu_teaching-furiosa` |
| **Crew** | `gt-<rig>-crew-<member>` | `gt-psu_teaching-crew-planner` |

**Other agents you might communicate with:**
- Mayor: `gt-mayor`
- Witness: `gt-<rig>-witness`
- Refinery: `gt-<rig>-refinery`

**DO NOT use:**
- ❌ `<rig>/<role>/<name>` (this is mail address, not tmux session)
- ❌ Wrong prefixes or missing components

**Verify session names:**
```bash
# List all sessions
tmux ls

# List sessions for your rig
tmux ls | grep <rig>
```

---

## Startup Protocol (GUPP - Gas Universal Propulsion Principle)

**If you find something on your hook, YOU RUN IT.**

```bash
# 1. Check hook (done automatically by gt prime)
gt hook

# 2. If work hooked → RUN IT IMMEDIATELY
#    - No announcements beyond one line
#    - No waiting for permission
#    - No asking "should I proceed?"
#    - EXECUTE

# 3. If no hook → Check mail for context and actionable work
gt-mail-safe inbox

# 4. Read recent messages (3-5 most recent)
#    Look for:
#    - 🤝 HANDOFF messages (continue previous work)
#    - Actionable requests from other agents
#    - Team status updates requiring response
#    - Work assignments

# 5. If actionable work found → EXECUTE IT
#    Treat mail requests like hooked work - begin immediately

# 6. If no actionable work → Report current state and await user instruction
```

**The hook IS your assignment.** It was placed there deliberately. Begin immediately.

**Mail provides context.** Even without a hook, recent messages tell you what's happening and what needs attention.

---

## Communication Targets

### When to Send Mail

| Recipient | Use Case | Pattern |
|-----------|----------|---------|
| **Mayor** | Escalations, cross-rig work, stuck issues | `gt mail send mayor/` |
| **Witness** | Report stuck state, need help | `gt mail send <rig>/witness` |
| **Refinery** | MR ready, merge conflicts | `gt mail send <rig>/refinery` |
| **Other Workers** | Collaboration, handoffs | `gt mail send <rig>/polecats/<name>` or `<rig>/crew/<member>` |
| **Overseer** | Human decisions needed | `gt mail send overseer` |

**Session-aware notification:** Check if session exists first, notify if running, escalate if expected-to-run session is down (see Communication Protocol above)

---

## Bead Workflow

### Finding Work

```bash
# Check what's on your hook
gt hook

# Find available work (if hook empty)
bd ready

# Show all your in-progress work
bd list --status=in_progress --assignee=<your-role>
```

### Working on Beads

```bash
# Claim work
bd update <bead-id> --status=in_progress

# Add notes/progress
bd update <bead-id> --comment="Progress update"

# Close when complete (after git push!)
bd close <bead-id>

# Close with reason
bd close <bead-id> --reason="Explanation"
```

### Creating Beads

```bash
# File new work discovered during task
bd create --title="..." --type=task --priority=2

# Add dependencies if needed
bd dep add <bead-id> <depends-on-id>
```

---

## Escalation Guidelines

### When to Escalate to Witness

- ❌ You're stuck for >10 minutes (after trying to resolve)
- ❌ Blocked by external dependency
- ❌ Need another polecat's help

### When to Escalate to Mayor

- ❌ Cross-rig work needed
- ❌ Witness not responding
- ❌ Architectural decisions beyond your scope

### When to Escalate to Overseer (Human)

- ❌ Truly need human judgment
- ❌ Security/compliance questions
- ❌ Business logic clarification

**Escalation Format:**

```markdown
**Severity:** CRITICAL | HIGH | MEDIUM
**Status:** BLOCKED | NEEDS INPUT
**Time stuck:** <duration>

**Problem:**
<What's wrong>

**Attempted:**
1. <What you tried>
2. <Result>

**Requested:**
<What you need>
```

---

## Commit Message Guidelines (Recommended)

For multi-agent rigs, prefix commits with role:

```bash
# Format: <role>: <description>
git commit -m "planner: Add Week 1 schedule breakdown"
git commit -m "furiosa: Fix authentication bug in login flow"
git commit -m "qa: Add integration tests for API endpoints"
```

**Severity indicators:**
- 🚨 CRITICAL: Blockers, data loss risk, security
- ⚠️ HIGH: Feature blockers, broken tests
- ℹ️ MEDIUM: Improvements, refactoring

---

## Resilience-by-Design

Your work implements resilience principles:

- **No single point of failure:** Mail persists even if tmux fails
- **Workflow completeness:** Mandatory push prevents orphaned work
- **Observability:** Git history + bead tracking = full audit trail
- **Self-healing:** Hook system ensures work persists across sessions
- **Graceful degradation:** Can work even if coordinators (Witness/Refinery) are down

---

## Common Mistakes to Avoid

❌ Sending mail to polecat/witness/refinery without checking session first
❌ Not escalating when expected-to-run session is down
❌ Ending session without git push
❌ Closing beads before work is pushed
❌ Using wrong tmux session names
❌ Waiting for permission when work is hooked (GUPP violation)
❌ Saying "ready when you are" instead of pushing yourself

---

## Quality Gates (Before Pushing)

**If you changed code, run these BEFORE committing:**

```bash
# Language-specific examples:
# Python
uv run pytest
uv run mypy .

# JavaScript/TypeScript
npm test
npm run lint

# Go
go test ./...
go vet ./...

# General
# - Run test suite
# - Run linters
# - Verify builds
# - Check for syntax errors
```

**Only push if quality gates pass.** If they fail:
1. Fix the issues
2. Rerun quality gates
3. Then push

---

## 🚨 POLECAT SESSION END (CRITICAL)

**Polecats MUST use `gt done` to complete work. This is NON-NEGOTIABLE.**

`gt done` is the ONLY valid way for polecats to complete work because it:
1. ✅ Verifies commits exist and are pushed (prevents lost work)
2. ✅ Creates MR bead for Refinery to process
3. ✅ Notifies Witness of completion
4. ✅ Closes the hooked bead automatically
5. ✅ Terminates the session cleanly

**Polecat session end:**
```bash
# 1. File any additional work discovered
bd create --title="..." --type=task

# 2. Run quality gates
uv run pytest  # All pass ✓

# 3. Commit and push your work
git add .
git commit -m "furiosa: Fix authentication bug"
git push  # MUST succeed before gt done

# 4. Submit to merge queue and exit
gt done   # ← THIS IS MANDATORY FOR POLECATS
          # Verifies push, creates MR, notifies Witness, exits session
```

**What `gt done` does:**
- Checks branch has commits ahead of main
- Pushes branch if not already pushed
- Creates merge-request bead for Refinery
- Notifies Witness of completion
- Cleans up worktree
- Kills session

**DO NOT use `bd close` directly!** It bypasses verification and your work will be lost.

---

## Crew Session End (Different from Polecats)

**Crew push directly to main and can use manual workflow:**

```bash
# 1. File remaining work
bd create --title="Add error handling to API" --type=task

# 2. Run quality gates
uv run pytest  # All pass ✓

# 3. Close completed beads
bd close mi-xyz

# 4. Push to remote (direct to main)
git pull --rebase
bd sync
git add .
git commit -m "planner: Add Week 1 schedule"
bd sync
git push
git status  # ✓ up to date with origin/main

# 5. Hand off if work incomplete
gt handoff -m "Continue with phase 2"
```

---

## Common Session End Mistakes

**❌ WRONG (Polecat):**
```bash
# Closes bead manually - bypasses verification!
bd close mi-xyz

# Claims work done but no git push
# Session ends with nothing committed

# ❌ Work lost forever - branch never existed
```

**❌ WRONG (Polecat):**
```bash
# Pushes but doesn't call gt done
git push

# No MR bead created
# Refinery never processes the branch
# Work sits on branch forever

# ❌ Use gt done instead
```

**✅ CORRECT (Polecat):**
```bash
git add . && git commit -m "fix: resolve 403 errors"
git push
gt done   # ← Required for polecats
```

---

**Version:** 1.0
**Based on:** Universal Gas Town Protocols (~/gt/PROTOCOLS.md)
**Applies to:** All polecats and crew members
