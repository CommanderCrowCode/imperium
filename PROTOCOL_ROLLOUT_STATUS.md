# Gas Town Protocol Rollout Status

**Implementation Strategy:** Gradual (Mayor → Coordinators → Workers)
**Started:** 2026-01-18
**Last Updated:** 2026-01-18

---

## Implementation Decisions (Resilience-by-Design)

| Question | Decision | Rationale |
|----------|----------|-----------|
| **Hook Injection** | Both (Option C) | Defense in depth: Universal + rig-specific. No single point of failure. |
| **Enforcement** | Hybrid (Medium + Critical) | Medium warnings for guidelines, hard enforcement for critical protocols. Observability without blocking. |
| **Rollout Pace** | Gradual (3 phases) | Catch issues early before wide deployment. Self-healing through iterative fixes. |

---

## Phase 1: Mayor ✅ COMPLETE

**Status:** Implemented 2026-01-18
**Agent:** Mayor (gt-mayor)

### Changes Made

1. ✅ Created `~/gt/PROTOCOLS.md` (universal reference)
2. ✅ Created `~/gt/PROTOCOLS_IMPLEMENTATION.md` (rollout guide)
3. ✅ Updated `~/gt/CLAUDE.md`:
   - Added "Universal Gas Town Protocols" section (line 142)
   - Updated "Communication Protocol" to MANDATORY (line 297)
   - Strengthened "Session End Checklist" (line 495)
4. ✅ Committed and pushed (commits 50dc6a1, fb4bb92)

### Protocols Enforced

| Protocol | Status | Implementation |
|----------|--------|----------------|
| **Communication (mail+tmux)** | ✅ MANDATORY | Referenced in CLAUDE.md line 142-152 |
| **Session Completion** | ✅ MANDATORY | 7-step checklist, line 495-522 |
| **Session Naming** | ✅ REFERENCE | Table included, line 163-169 |
| **Blocker Escalation** | ℹ️ OPTIONAL | Available in PROTOCOLS.md |
| **Commit Messages** | ℹ️ OPTIONAL | Guidelines in PROTOCOLS.md |

### Testing

**Protocol Compliance Test:**
- [x] Session started with `gt hook` check
- [x] Work documented in TodoWrite tool
- [x] Changes committed incrementally (PROTOCOLS.md, CLAUDE.md)
- [x] bd sync run before commits
- [x] git push executed successfully
- [x] No mail sent (no tmux notification needed)
- [ ] Quality gates: N/A (documentation only, no code)
- [x] git status shows up to date

**Result:** ✅ PASS - Mayor session followed all protocols

---

## Phase 2: Coordinators ⏳ PENDING

**Target Agents:**
- Witnesses (`gt-<rig>-witness`)
- Refineries (`gt-<rig>-refinery`)

**Plan:**
1. Update rig-specific CLAUDE.md files (if they exist)
2. Add session naming reference table for their rig
3. Emphasize communication protocol (they coordinate multiple agents)
4. Test with 2-3 rigs before wide rollout

**Priority Rigs:**
- psu_teaching (active, already follows most protocols)
- lumicello_website (active, refinery in use)
- 10x_screening (moderate activity)

**Timeline:** Week 1 (after Mayor validation)

---

## Phase 3: Workers ⏳ PENDING

**Target Agents:**
- Polecats (`gt-<rig>-<polecat-name>`)
- Crew (`gt-<rig>-crew-<member>`)

**Plan:**
1. Universal hook injection in `~/.config/claude-code/hooks/user-prompt-submit.sh`
2. Inject Communication + Session Completion sections
3. Test with sample polecats across 3 rigs
4. Monitor for compliance over 1 week

**Timeline:** Week 2 (after Coordinator validation)

---

## Universal Hook Injection ⏳ PENDING

**Target:** All future agent spawns receive protocols automatically

**Implementation:**
```bash
# Edit ~/.config/claude-code/hooks/user-prompt-submit.sh
# Inject Communication Protocol + Session Completion sections
# Test with new polecat spawn
```

**Timeline:** Week 3 (after Worker validation)

---

## Success Metrics

### Phase 1 (Mayor) ✅
- [x] PROTOCOLS.md created and pushed
- [x] CLAUDE.md updated with references
- [x] Mayor session demonstrates compliance
- [x] No regressions in Mayor operations

### Phase 2 (Coordinators) - Target Metrics
- [ ] 0 "mail not checked" incidents
- [ ] 100% of coordinator sessions push before end
- [ ] 0 tmux session naming errors

### Phase 3 (Workers) - Target Metrics
- [ ] 100% of new polecats receive protocols
- [ ] 80%+ protocol compliance across all workers
- [ ] Protocol violations flagged by monitoring

---

## Known Gaps & Next Steps

### Immediate (Week 1)
1. Test Mayor compliance over multiple sessions
2. Prepare Witness/Refinery context updates
3. Identify rigs for Phase 2 rollout

### Short-term (Week 2-3)
4. Implement universal hook injection
5. Create automated compliance checking (`gt doctor` integration)
6. Generate session name tables per rig

### Long-term (Month 1)
7. Protocol violation reporting
8. Compliance metrics dashboard
9. Auto-remediation for common violations

---

## Rollback Plan

If protocols cause issues:

1. **Immediate:** Revert CLAUDE.md to pre-protocol version
2. **Phase-specific:** Only rollback affected phase (Mayor/Coordinators/Workers)
3. **Graceful degradation:** Protocols are guidelines - agents work without them
4. **No data loss:** All protocols enforce what should already happen

**Rollback trigger:** >3 protocol-related failures per week

---

## Maintenance

**Protocol updates:**
1. Update `PROTOCOLS.md` (source of truth)
2. Regenerate context injections
3. Test with sample agents
4. Update rollout status

**Review cadence:**
- Weekly during rollout (Weeks 1-3)
- Monthly after full deployment

---

**Status Summary:**
- ✅ Phase 1 (Mayor): COMPLETE
- ⏳ Phase 2 (Coordinators): PENDING
- ⏳ Phase 3 (Workers): PENDING
- ⏳ Universal Hook: PENDING

**Next Action:** Monitor Mayor compliance, prepare Phase 2 rollout
