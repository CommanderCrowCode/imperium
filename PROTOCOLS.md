# Gas Town Operational Protocols

> **Universal protocols for ALL Gas Town agents**
> These protocols apply to Mayor, Witness, Refinery, Polecats, and Crew

---

## 🔄 Communication Protocol (MANDATORY)

**When sending mail expecting action or response, you MUST follow all steps:**

### 1. Send Mail (Audit Trail)
```bash
gt mail send <address> -s "Subject" -m "Message content"
```

### 2. Verify Recipient Session Exists
```bash
tmux ls | grep <expected-session-name>
```

### 3. Send tmux Notification (CRITICAL)
```bash
# Send notification message
tmux send-keys -t <session-name> "New mail from <your-role>. Check: gt mail inbox"

# Send Enter key SEPARATELY (required for execution)
tmux send-keys -t <session-name> Enter
```

### 4. Verify Notification Sent
Check that tmux command returned successfully (exit code 0).

---

## ⚠️ Why This Matters

**Without tmux notification:**
- Mail sits unread indefinitely
- Agents don't auto-poll mail
- Work stalls silently
- No error messages
- Pipeline blocks waiting for responses

**This is a resilience-by-design requirement:**
- No single point of failure: Mail persists even if notification fails
- Observability: Both audit trail (mail) and immediate alert (tmux)
- Self-healing: Agent checks mail on next prompt if notification missed

---

## 📋 Session Completion Protocol (MANDATORY)

**Before ending ANY session, complete ALL steps in order:**

### Checklist

```
[ ] 1. File remaining work as beads (don't leave TODOs in memory)
[ ] 2. Run quality gates (if code changed: tests, linters, builds)
[ ] 3. Update bead status (close completed, update in-progress)
[ ] 4. Push to remote (MANDATORY):
        git pull --rebase
        bd sync
        git commit -m "..." (if local changes)
        bd sync
        git push
        git status  # MUST show "up to date with origin"
[ ] 5. Verify tmux notifications sent (if mail sent expecting action)
[ ] 6. Clean up (clear stashes, prune remote branches)
[ ] 7. Hand off context (create handoff mail if work incomplete)
```

### CRITICAL RULES

- ❌ Work is NOT complete until `git push` succeeds
- ❌ NEVER stop before pushing - that leaves work stranded locally
- ❌ NEVER say "ready to push when you are" - YOU must push
- ❌ If push fails, resolve conflicts and retry until it succeeds
- ✅ If you sent mail expecting action, tmux notification is MANDATORY

---

## 🏷️ Session Naming Conventions

Gas Town uses consistent tmux session naming patterns for all agents.

### Pattern Reference

| Agent Type | Pattern | Example |
|------------|---------|---------|
| **Mayor** | `gt-mayor` | `gt-mayor` |
| **Deacon** | `gt-deacon` | `gt-deacon` |
| **Rig Witness** | `gt-<rig>-witness` | `gt-psu_teaching-witness` |
| **Rig Refinery** | `gt-<rig>-refinery` | `gt-lumicello_website-refinery` |
| **Polecat** | `gt-<rig>-<polecat-name>` | `gt-10x_screening-furiosa` |
| **Crew** | `gt-<rig>-crew-<member>` | `gt-psu_teaching-crew-planner` |

### Common Mistakes (DO NOT USE)

❌ `psu_teaching-writer` (missing `gt-` prefix)
❌ `gt-writer` (missing rig context)
❌ `psu_teaching/crew/writer` (using slashes instead of hyphens)
❌ `gt-psu_teaching-polecats/furiosa` (wrong polecat format)

### How to Verify Session Names

```bash
# List all Gas Town sessions
tmux ls | grep gt-

# List sessions for a specific rig
tmux ls | grep gt-psu_teaching

# Verify a specific session exists before sending keys
tmux has-session -t gt-psu_teaching-crew-planner 2>/dev/null && echo "exists" || echo "not found"
```

---

## 🎯 Blocker Escalation Format (RECOMMENDED)

When escalating critical issues via mail or beads, use structured format:

```markdown
**Severity:** CRITICAL | HIGH | MEDIUM | LOW
**Status:** NOT RESOLVED | WORKAROUND EXISTS | NEEDS FIXING | RESOLVED
**Owner:** <agent-name> or <human-role>
**Time to fix:** <estimate>

**Problem:**
<Clear description of what's broken>

**Impact:**
<What fails if not fixed>

**Required Actions:**
1. <Specific action>
2. <Specific action>
3. <Specific action>
```

**Example:**
```markdown
**Severity:** CRITICAL
**Status:** NOT RESOLVED
**Owner:** lumicello_inventory/refinery
**Time to fix:** 30 minutes

**Problem:**
- MR #47 has merge conflicts in schema.sql
- Refinery stuck waiting for resolution

**Impact:**
- Blocks 3 downstream MRs
- Week 4 migration at risk

**Required Actions:**
1. Mayor or human resolve conflicts manually
2. Update schema.sql in refinery worktree
3. Notify refinery to retry merge
```

---

## 🚨 Commit Message Guidelines (RECOMMENDED)

For multi-agent rigs with high activity:

**Format:**
```
<agent-prefix>: <description>

Examples:
planner: Add workflow improvements and marker analysis
refinery: Merge MR #47 after conflict resolution
🚨 CRITICAL: Week 0 BLOCKED by missing quiz
```

**Severity Emoji:**
- 🚨 CRITICAL: Launch blockers, data loss risk, security issues
- ⚠️ HIGH: Feature blockers, broken tests, performance degradation
- ℹ️ MEDIUM: Improvements, refactoring, documentation

**Benefits:**
- Git log shows which agent made changes
- Severity visible at a glance
- Easier to track multi-agent coordination

---

## 📚 Session Name Reference: psu_teaching Rig

**Pattern:** `gt-psu_teaching-crew-<member>`

| Crew Member | tmux Session Name |
|-------------|-------------------|
| planner | `gt-psu_teaching-crew-planner` |
| researcher | `gt-psu_teaching-crew-researcher` |
| writer | `gt-psu_teaching-crew-writer` |
| writer_2 | `gt-psu_teaching-crew-writer_2` |
| writer_3 | `gt-psu_teaching-crew-writer_3` |
| writer_4 | `gt-psu_teaching-crew-writer_4` |
| writer_5 | `gt-psu_teaching-crew-writer_5` |
| assessment_specialist | `gt-psu_teaching-crew-assessment_specialist` |
| audio_engineer | `gt-psu_teaching-crew-audio_engineer` |
| moodle_admin | `gt-psu_teaching-crew-moodle_admin` |
| slide_designer | `gt-psu_teaching-crew-slide_designer` |
| video_producer | `gt-psu_teaching-crew-video_producer` |
| qa_specialist | `gt-psu_teaching-crew-qa_specialist` |

**Note:** This table extracted from psu_teaching/crew/planner/TMUX_SESSION_NAMES.md after real tmux notification failures (Jan 16, 2026)

---

## 🛡️ Alignment with Resilience-by-Design

These protocols implement Gas Town's resilience-by-design principles:

| Principle | Protocol Implementation |
|-----------|------------------------|
| **No Single Point of Failure** | Mail (persistent) + tmux (immediate) = redundant communication |
| **Defense in Depth** | Multiple verification steps (session check, notification verify) |
| **Graceful Degradation** | Mail survives if tmux fails; agents check on next prompt |
| **Self-Healing** | Auto-recovery via `gt prime` loads mail on restart |
| **Observability** | Git audit trail + session completion checklist |
| **Workflow Completeness** | Mandatory push requirement prevents orphaned work |

---

## 📝 Hook Injection Notes

**For Gas Town maintainers:**

These protocols should be injected via startup hooks to ensure all agents receive them:

**Target hooks:**
- `~/.config/claude-code/hooks/user-prompt-submit.sh` (all agents)
- Rig-specific hooks in `<rig>/.claude/hooks/` (if needed)

**Injection strategy:**
- Full PROTOCOLS.md for Mayor, Witness, Refinery (coordinators)
- Communication + Session Completion sections for Polecats, Crew (workers)
- Session naming table for all agents

**Current injection status:**
- ✅ Communication best practices in ~/gt/CLAUDE.md (Mayor only)
- ❌ Not yet injected for other agent types
- ❌ Session completion not enforced universally

---

**Version:** 1.0
**Created:** 2026-01-18
**Based on:** psu_teaching crew best practices + Mayor context
**Maintainer:** overseer

