# Workflow Skills Revision Plan

**Status:** Final Plan - Ready for Implementation
**Created:** 2025-12-08
**Updated:** 2025-12-08 (Post-clarification)

---

## Executive Summary

Comprehensive revision of the Skills workflow to:
1. **Primary**: Implement Environment Registry with selective data access (Phase 0 pattern)
2. **Secondary**: Extend Planning Mindset beyond project-brief-writer where beneficial
3. **Tertiary**: Align with Anthropic's domain memory principles, cherry-picking best elements

User priorities confirmed:
- Environment Registry is top priority (learning investment + workflow efficiency)
- Reference materials alongside skills (not centralized) for human visibility
- Current handoff system adequate; cherry-pick from claude-progress.txt pattern
- 500-line guideline, not rule (Anthropic says "under 500 lines for optimal performance")
- User-level storage preferred where reasonable

---

## Research Summary

### Anthropic's 500-Line Guidance (from Best Practices)

> "Keep SKILL.md body under 500 lines for optimal performance. If your content exceeds this, split it into separate files using progressive disclosure patterns."

Key insight: The limit is about **progressive disclosure**, not arbitrary constraint. Skills with code snippets vs. dense narrative have different density profiles. The solution is splitting to separate files, not cramming.

### Anthropic's Progressive Disclosure Pattern

Three levels of loading:
1. **Level 1 - Metadata** (always loaded): ~100 tokens per skill (name + description)
2. **Level 2 - Instructions** (when triggered): Under 5k tokens (SKILL.md body)
3. **Level 3 - Resources** (as needed): Effectively unlimited (bundled files)

This validates the project-spinup model (SKILL.md + EXAMPLES.md + QUICK_REFERENCE.md) as the target pattern.

### Domain Memory Principles to Cherry-Pick

From Anthropic's "Effective Harnesses for Long-Running Agents":

| Principle | Current State | Action |
|-----------|---------------|--------|
| External artifacts as memory | ✅ `.docs/` handoffs | Keep |
| JSON for machine state | ✅ `brief.json`, etc. | Keep |
| Standardized bootup ritual | ⚠️ Partial | Add Phase 0 |
| Progress logging | ⚠️ Missing | Consider adding to handoff system |
| State verification before work | ⚠️ Partial | Strengthen in Phase 0 |

---

## Implementation Plan

### Phase A: Environment Registry Foundation

**Files to create/modify:**

1. **Create `~/.claude/environment.json`** (user-level)
   - Full schema per `environment-registry-planning.md`
   - Include `skill_data_access` block per `skill-data-access-update.md`

2. **Create `~/.claude/environment-schema.json`** (optional validation)
   - JSON Schema for validation

**Critical paths:**
- [environment-registry-planning.md](docs/environment-registry-planning.md) - Full schema
- [skill-data-access-update.md](docs/skill-data-access-update.md) - Phase 0 template

---

### Phase B: Universal Phase 0 Implementation

**Add Phase 0 to each skill** (per `skill-data-access-update.md`):

```xml
<phase id="0" name="load-environment">
  <description>Load environment context scoped to this skill</description>
  <steps>
    1. Attempt to read ~/.claude/environment.json
    2. If not found: Note and proceed (graceful degradation)
    3. If found:
       a. Read skill_data_access["{this-skill-name}"]
       b. Extract ONLY the listed paths
       c. Do NOT reference out-of-scope data
  </steps>
</phase>
```

**Implementation order** (pilot → rollout):
1. `deployment-advisor` - Primary registry consumer, best test case
2. `tech-stack-advisor` - Second-highest registry dependency
3. `project-spinup` - Uses services and credentials
4. `deploy-guide` - Heavy infrastructure user
5. `ci-cd-implement` - GitHub account context
6. `test-orchestrator` - Minimal registry needs
7. `project-brief-writer` - Empty access (no registry needed, add for consistency)

**Skill-specific Phase 0 data access:**
| Skill | Registry Paths |
|-------|---------------|
| deployment-advisor | `deployment_options`, `infrastructure.vps`, `storage_options`, `skill_guidance.skip_questions`, `skill_guidance.never_suggest` |
| tech-stack-advisor | `database_options`, `skill_guidance.preferences` |
| project-spinup | `infrastructure.services`, `credentials.api_endpoints`, `established_choices` |
| deploy-guide | `infrastructure`, `credentials`, `established_choices` |
| ci-cd-implement | `deployment_options`, `external_accounts.github` |
| test-orchestrator | `established_choices.containerization` |
| project-brief-writer | `[]` (empty - no registry access) |

---

### Phase C: Planning Mindset Extension

**Current state**: project-brief-writer implements full Planning Mindset:
- Discovery Protocol (Scope → Opportunity → Curiosity)
- Framing Exercise (Solution + Outcome + Exploration)
- Approval Gates (Understanding Gate, Handoff Gate)

**Extension opportunities** (where exploration-before-commitment adds value):

| Skill | Planning Mindset Element | Rationale |
|-------|-------------------------|-----------|
| tech-stack-advisor | Lightweight Discovery | Understanding constraints before recommending |
| deployment-advisor | Decision Framing | Reflecting options back before committing |
| project-spinup | Approval Gate | Confirming foundation before generation |

**NOT recommended for**:
- deploy-guide (BUILDER role - execution, not planning)
- ci-cd-implement (BUILDER role - execution, not planning)
- test-orchestrator (BUILDER role with light guidance)
- workflow-status (UTILITY role - read-only)

---

### Phase D: Progressive Disclosure Restructuring

**Skills requiring externalization** (following project-spinup model):

| Skill | Current Lines | External File(s) | Content to Move |
|-------|---------------|------------------|-----------------|
| deploy-guide | 772 | TROUBLESHOOTING.md | Troubleshooting section |
| ci-cd-implement | 747 | WORKFLOW-TEMPLATES.md | GitHub Actions templates |
| tech-stack-advisor | 656 | DECISION-FRAMEWORKS.md | Enterprise vs Hacker, decision trees |
| deployment-advisor | 633 | HOSTING-TEMPLATES.md | Deployment pattern templates |
| test-orchestrator | 684 | TEST-EXAMPLES.md | Framework-specific examples |

**File placement**: Alongside SKILL.md in skill folder (per user preference)

**Template for externalization**:
```markdown
# SKILL.md (main file)
## [Section Header]
For detailed [content type], see [REFERENCE-FILE.md](REFERENCE-FILE.md)

# REFERENCE-FILE.md (supporting file)
# [Title]
[Detailed content moved from main SKILL.md]
```

---

### Phase E: Handoff System Enhancements

**Cherry-picked from Anthropic's progress file pattern:**

1. **Add workflow state to handoffs**: Each handoff document should include:
   - Previous phase status (complete/partial)
   - Decisions locked vs. open
   - Next expected phase

2. **Consider progress summary in workflow-status**: The workflow-status skill could generate a session summary similar to claude-progress.txt when invoked.

**Gap analysis for memory transfer:**
- Current: Each skill produces specific handoff documents
- Anthropic: Single progress file updated by each session
- Hybrid approach: workflow-status aggregates handoff documents into progress view

---

## Files to Modify

### High Priority (Phase A + B)

| File | Change Type | Description |
|------|-------------|-------------|
| `~/.claude/environment.json` | CREATE | Full environment registry |
| `deployment-advisor/SKILL.md` | MODIFY | Add Phase 0, externalize templates |
| `tech-stack-advisor/SKILL.md` | MODIFY | Add Phase 0, externalize frameworks |
| `project-spinup/SKILL.md` | MODIFY | Add Phase 0 |
| `deploy-guide/SKILL.md` | MODIFY | Add Phase 0, externalize troubleshooting |
| `ci-cd-implement/SKILL.md` | MODIFY | Add Phase 0, externalize templates |
| `test-orchestrator/SKILL.md` | MODIFY | Add Phase 0, externalize examples |
| `project-brief-writer/SKILL.md` | MODIFY | Add Phase 0 (empty access) |

### New Supporting Files (Phase D)

| File | Location | Content |
|------|----------|---------|
| HOSTING-TEMPLATES.md | deployment-advisor/ | Deployment pattern templates |
| DECISION-FRAMEWORKS.md | tech-stack-advisor/ | Enterprise vs Hacker, decision trees |
| TROUBLESHOOTING.md | deploy-guide/ | Target-specific troubleshooting |
| WORKFLOW-TEMPLATES.md | ci-cd-implement/ | GitHub Actions templates |
| TEST-EXAMPLES.md | test-orchestrator/ | Framework-specific test examples |

---

## Implementation Sequence

```
Week 1: Foundation
├── Create ~/.claude/environment.json with full schema
├── Create ~/.claude/environment-schema.json (validation)
├── Add Phase 0 to deployment-advisor (pilot)
└── Test with real project scenario

Week 2: Rollout Phase 0
├── Add Phase 0 to remaining 6 skills
├── Verify graceful degradation (no registry case)
└── Test skill-to-skill handoffs work correctly

Week 3: Progressive Disclosure
├── Create external reference files (5 files)
├── Update SKILL.md files to reference externals
├── Verify line counts are under guidance
└── Test that Claude accesses external files correctly

Week 4: Planning Mindset Extension
├── Add lightweight Discovery to tech-stack-advisor
├── Add Decision Framing to deployment-advisor
├── Add Approval Gate to project-spinup
└── Test workflow end-to-end with new patterns
```

---

## Success Criteria

1. **Environment Registry Working**: Skills load appropriate registry data without asking redundant questions
2. **Graceful Degradation**: Skills function normally when registry is absent
3. **Progressive Disclosure**: Main SKILL.md files under 500 lines, external files loaded on demand
4. **Planning Mindset Effective**: CONSULTANT skills have appropriate exploration phases
5. **Handoff Continuity**: No gaps in memory transfer between workflow steps

---

## Open Items (Resolved)

| Question | Resolution |
|----------|------------|
| Q1: Registry priority | TOP priority - learning investment + efficiency |
| Q2: Reference location | Alongside skills (not centralized) |
| Q3: Progress tracking | Current handoff system + cherry-pick enhancements |
| Q4: 500-line target | Guideline, not rule - progressive disclosure key |
| Q5: Scope | All of the above (A+B+C+D) |
| Q6: User vs project level | User-level preferred where reasonable |
