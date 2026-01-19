# Worker Context (Universal)

> **For all Gas Town workers: Polecats and Crew**

This context applies to all worker agents in Gas Town, whether you're a polecat (ephemeral worker) or crew member (persistent workspace).

---

## 📋 Universal Gas Town Protocols (MANDATORY)

**All Gas Town agents follow universal operational protocols. See `~/gt/PROTOCOLS.md` for complete reference.**

### Communication Protocol (CRITICAL)

When sending mail expecting action or response, you MUST:

1. **Send mail** (audit trail)
   ```bash
   gt mail send <address> -s "Subject" -m "Message"
   ```

2. **Verify session exists**
   ```bash
   tmux ls | grep <expected-session>
   ```

3. **Send tmux notification** (agents don't auto-poll)
   ```bash
   tmux send-keys -t <session> "New mail from <your-role>. Check: gt mail inbox"
   tmux send-keys -t <session> Enter
   ```

4. **Verify notification sent**

**Without tmux notification, mail sits unread and work stalls.**

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

# 3. If no work hooked → Check mail
gt mail inbox

# 4. If no mail → Wait for user instructions
```

**The hook IS your assignment.** It was placed there deliberately. Begin immediately.

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

**Always pair mail with tmux notification** (see Communication Protocol above)

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

❌ Sending mail without tmux notification
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

## Session End Example

**Good session end:**
```bash
# 1. File remaining work
bd create --title="Add error handling to API" --type=task

# 2. Run quality gates
uv run pytest  # All pass ✓

# 3. Update bead
bd close mi-xyz  # My completed work

# 4. Push to remote
git pull --rebase
bd sync
git add .
git commit -m "audio: Regenerate Week 0 audio with markers"
bd sync
git push
git status  # ✓ up to date with origin/main

# 5. No mail sent (skip)

# 6. Clean up
git stash list  # Empty ✓

# 7. No handoff needed (work complete)
```

**Bad session end (DO NOT DO THIS):**
```bash
# Closes bead before pushing
bd close mi-xyz

# Never pushes
# Leaves work stranded locally
# Session ends

# ❌ WRONG - Work lost on session restart
```

---

**Version:** 1.0
**Based on:** Universal Gas Town Protocols (~/gt/PROTOCOLS.md)
**Applies to:** All polecats and crew members
