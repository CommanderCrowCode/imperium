# Night Shift Learnings - 2026-01-27

## Operational Insights

### Beads Routing Discovery
**Issue:** Multiple rigs showing HQ (gm-) beads instead of rig-specific beads
**Root Cause:** Must run `bd` commands from `mayor/rig` subdirectories for proper routing
**Pattern:** Routes defined in `~/gt/.beads/routes.jsonl` map prefixes to paths

### Witness Management  
**Pattern:** Witnesses can go down during operations
**Resolution:** Simple session restart resolves most issues
**Prevention:** Need automated witness health monitoring

### Work Distribution Efficiency
**Success:** Spawned 3 polecats in ~10 minutes using gt-sling-fix
**P0 Priority:** Always check for production bugs first (ve-wso example)
**Lesson:** Use `bd ready | grep -E "^\[● P[01]\]"` to surface critical work

### Mail Context Injection
**Status:** Phase 1 deployed (SessionStart hook)
**Gap:** No ToolUse hook yet - long-running polecats miss mid-task mail
**Next:** Implement Phase 2 (gm-m943b filed)

## Infrastructure Improvements Filed

1. **gm-m943b** (P1): ToolUse hook for mail robustness
2. **gm-xad45** (P2): Fix beads routing for lumicello_inventory
3. **gm-ro5uk** (P2): Automated rig housekeeping script
4. **gm-9bdeh** (P3): Night shift status dashboard

## Process Optimizations Needed

- Automated rig sweep (currently manual 7× iteration)
- Better polecat idle detection
- Beads creation from codebase analysis (TODOs, FIXMEs)
- Cross-rig dependency visualization

## Mayor Capability Growth

**Demonstrated:**
- Autonomous escalation handling (witness restart)
- Multi-rig coordination (7 rigs assessed)
- Priority-based work distribution (P0→P1→P2)
- Infrastructure bead filing (4 improvements)

**To Develop:**
- Automated housekeeping loops
- Predictive polecat allocation
- Smart bead generation from code analysis
- Performance dashboards

