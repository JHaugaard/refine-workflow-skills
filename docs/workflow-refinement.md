# Workflow Refinement Study Document

**Created:** 2025-12-04
**Revised:** 2025-12-10 (v2 - Decisions Finalized)
**Purpose:** Capture decisions, architecture, and implementation approach for refining the Skills workflow
**Status:** Ready for Implementation

---

## Table of Contents

1. [Context & Problem Statement](#context--problem-statement)
2. [The Workflow Overview](#the-workflow-overview)
3. [Identified Problems](#identified-problems)
4. [Architecture Decisions](#architecture-decisions)
5. [Implementation Approach](#implementation-approach)
6. [Resolved Questions](#resolved-questions)
7. [Reference: Anthropic's Guidance](#reference-anthropics-guidance)
8. [Next Steps](#next-steps)

---

## Context & Problem Statement

### What We're Building

A workflow of Claude Code skills that guides a user from initial project idea through to deployed, automated application. The workflow is designed to:

- Preserve learning opportunities (especially in LEARNING mode)
- Produce neutral handoffs between skills (no premature decisions)
- Allow flexible entry points (skills can run standalone)
- Support natural termination points (not every project needs CI/CD)

### The Core Problems Identified

After running the workflow twice, two critical issues emerged:

1. **Insufficient or ignored guardrails** - Skills bleed into each other's scope (e.g., project-brief-writer suggesting tech, deployment-advisor providing Docker snippets)

2. **Losing the thread of decisions** - Key decisions made during discussion get lost or contradicted later (e.g., deciding "deploy entirely on fly.io" but later the skill splits deployment between fly.io and self-hosted database)

### The Goal

Optimize the skills and handoffs for Claude's consumption, not human readability. Use structured formats (XML for skills, JSON for handoffs) to eliminate ambiguity and enforce decision boundaries.

---

## The Workflow Overview

### Skill Sequence

```
Phase 0: project-brief-writer
    |
Phase 1: tech-stack-advisor
    |
Phase 2: deployment-advisor
    |
Phase 3: project-spinup <-- TERMINATION POINT (localhost projects)
    |
    [User builds features]
    |
Phase 4: test-orchestrator (optional, can run standalone)
    |
Phase 5: deploy-guide <-- TERMINATION POINT (manual deploy)
    |
Phase 6: ci-cd-implement <-- TERMINATION POINT (full automation)
```

**Rationale for deploy-guide before ci-cd-implement:**
- "Make it work, then automate" — don't automate a process you've never done manually
- Manual deployment surfaces issues (missing env vars, permissions, misconfigurations) that are frustrating to debug inside CI/CD
- For learning-focused workflows, manual deploy teaches what's actually happening
- CI/CD pipelines need deployment knowledge (secrets, targets, order) that a successful manual deploy provides

### Skill Roles

| Skill | Role | Scope Boundary |
|-------|------|----------------|
| **project-brief-writer** | Capture WHAT and WHY | NO tech, NO platforms, NO architecture |
| **tech-stack-advisor** | Recommend technologies | NO deployment targets, NO infrastructure |
| **deployment-advisor** | Recommend hosting strategy | NO code snippets, NO Docker configs, NO .env contents |
| **project-spinup** | Generate project foundation | Uses decisions from above, NO new tech/deployment decisions |
| **test-orchestrator** | Set up testing infrastructure | Analyzes existing project, NO tech stack changes |
| **deploy-guide** | Execute deployment | Follows established strategy, NO strategy changes |
| **ci-cd-implement** | Create CI/CD pipelines | Reads deployment target, NO deployment execution |
| **workflow-status** | Display progress & validate | Read-only, reports state |

### Termination Points

The workflow has three natural stopping points:

1. **After project-spinup** - For localhost/learning projects
2. **After deploy-guide** - For projects with manual deployment
3. **After ci-cd-implement** - For fully automated projects

Users don't need to complete all phases.

---

## Identified Problems

### Problem 1: Scope Creep Between Skills

**Symptoms:**
- project-brief-writer might suggest "this sounds like a good fit for Next.js"
- deployment-advisor provides `.env` file snippets or Docker compose fragments
- Skills reference user infrastructure in ways that bias recommendations

**Root Causes:**
- Guardrails are buried at the bottom of skill definitions
- Guardrails are phrased aspirationally, not as hard stops
- No enforcement mechanism to verify scope compliance

**Solution:** Hard boundaries at top of each skill, explicit in-scope/out-of-scope lists

### Problem 2: Decision Drift

**Symptoms:**
- User decides "deploy everything on fly.io" during deployment-advisor discussion
- Later, deploy-guide or ci-cd-implement starts splitting between fly.io and self-hosted
- Key decisions are buried in prose within handoff documents

**Root Causes:**
- No single source of truth for decisions
- No concept of "LOCKED" vs "tentative" decisions
- Decisions scattered across multiple markdown documents

**Solution:** JSON handoff documents, single DECISIONS.json registry, lock-on-handoff pattern

---

## Architecture Decisions

### Decision 1: XML for Skills, JSON for Data

| Component | Format | Rationale |
|-----------|--------|-----------|
| Skill definitions (SKILL.md) | XML with semantic tags | Claude was trained on XML; provides "semantic anchoring" for complex prompts |
| Handoff documents | JSON | Machine-readable, schema-validatable, unambiguous |
| Decision registry | JSON | Structured, queryable, prevents prose-buried decisions |
| Human summaries | None (JSON only) | Single source of truth; markdown can be generated later if needed |

### Decision 2: Single Decision Registry

All locked decisions accumulate in `.docs/DECISIONS.json` rather than being scattered across multiple files.

**Benefits:**
- Single source of truth
- Easy to display "what's been decided"
- Easy to validate new actions against existing decisions
- workflow-status can read one file to show complete state

### Decision 3: JSON Handoffs with Narrative Rationale

Each skill produces a JSON handoff document with a human-readable RATIONALE section for debugging.

**Naming convention (descriptive names):**
- `.docs/brief.json`
- `.docs/tech-stack-decision.json`
- `.docs/deployment-strategy.json`
- `.docs/DECISIONS.json` (cumulative registry)

**Structure example:**
```json
{
  "document_type": "tech-stack-decision",
  "version": "1.0",
  "created": "2025-12-10",
  "project": "my-project",
  "mode": "LEARNING",

  "rationale": "Next.js was chosen for its SSR capabilities and learning value. Supabase provides managed PostgreSQL with built-in auth, reducing infrastructure complexity.",

  "decisions": [
    {
      "id": "TSA-001",
      "category": "frontend",
      "decision": "Next.js 15 with App Router",
      "status": "LOCKED",
      "rationale": "Modern React patterns, SSR support, learning value"
    }
  ],

  "alternatives_considered": [
    {
      "name": "FastAPI + React",
      "ruled_out_reason": "More complex setup, less integrated"
    }
  ],

  "handoff_to": ["deployment-advisor", "project-spinup"]
}
```

### Decision 4: Hard Boundaries at Top of Skills

Critical scope restrictions go at the very top of each skill, not buried in guardrails at the bottom.

```xml
<hard-boundaries>
BEFORE ANY OUTPUT, I MUST VERIFY:
1. I will NOT suggest technologies - that is tech-stack-advisor's scope
2. I will NOT suggest deployment platforms - that is deployment-advisor's scope
3. I will ONLY work within: [explicit scope definition]

If a previous decision seems wrong, I will ASK THE USER before contradicting it.
</hard-boundaries>
```

### Decision 5: User Context in Environment Registry

User-specific context (infrastructure, preferences, skip-questions) stays consolidated in `~/.claude/environment.json` rather than being split into a separate user-profile file.

Skills access only their designated paths via the `skill_data_access` block.

---

## Implementation Approach

### Status

| Step | Status |
|------|--------|
| Environment Registry (`~/.claude/environment.json`) | DONE |
| Verify project-brief-writer is final | PENDING |
| Adjust tech-stack-advisor for JSON handoffs | PENDING |
| Adjust deployment-advisor for JSON handoffs | PENDING |
| User tests pattern (brief -> tech-stack -> deployment) | PENDING |
| Roll out remaining skills per cozy-dazzling-pixel.md | PENDING |

### Implementation Order

1. **Verify** project-brief-writer produces correct JSON handoff
2. **Adjust** tech-stack-advisor for JSON handoffs
3. **Adjust** deployment-advisor for JSON handoffs
4. **Test** the three-skill sequence with a real project
5. **Roll out** to remaining skills:
   - project-spinup
   - test-orchestrator
   - deploy-guide
   - ci-cd-implement
   - workflow-status

---

## Resolved Questions

| Question | Decision | Rationale |
|----------|----------|-----------|
| **Q1: JSON Schema Strictness** | Option B: Flexible, required fields only | Start simple, add strictness if problems emerge |
| **Q2: Decision Locking Ceremony** | Option C: Lock on handoff, revise by re-running skill | Balance of commitment and flexibility; by handoff completion, decisions have been discussed |
| **Q3: workflow-status Role** | Option C: Hybrid | Provides validation templates; skills run independently but with shared patterns |
| **Q4: User Context Handling** | Keep in environment.json | Already working, no need for separate file |
| **Q5: Handoff File Naming** | Option C: Descriptive names | `brief.json`, `tech-stack-decision.json` clearer than phase numbers |
| **Q6: Backward Compatibility** | Option A: JSON only | Single source of truth; markdown can be generated later if needed |

---

## Reference: Anthropic's Guidance

### On XML Tags (for prompts/skills)

From Anthropic's documentation:

> "XML tags can be a game-changer. They help Claude parse your prompts more accurately, leading to higher-quality outputs."

**Key benefits:**
- Clarity — separating prompt components prevents confusion
- Accuracy — reduces misinterpretation errors
- Flexibility — easy modification without full rewrites
- Parseability — structured output enables post-processing

### On JSON (for structured outputs)

From Anthropic's Structured Outputs documentation:

- Native structured outputs use **constrained decoding**
- The model literally cannot produce tokens that violate the schema
- Supports flexible schemas with required fields only

### XML vs JSON Summary

| Use Case | Recommended Format |
|----------|-------------------|
| Prompt structure (skills) | XML |
| Data interchange (handoffs) | JSON |
| Decision registry | JSON |
| Human documentation | None (JSON serves as source of truth) |

---

## Next Steps

### Immediate

1. Verify project-brief-writer produces correct JSON handoff
2. Adjust tech-stack-advisor and deployment-advisor for JSON handoffs
3. Test the three-skill sequence

### After Testing

Follow implementation plan in `.docs/cozy-dazzling-pixel.md` for:
- Phase 0 (environment loading) rollout to all skills
- Progressive disclosure restructuring (external reference files)
- Planning Mindset extension where beneficial

---

## Appendix: Current Skill Files

All skills are located in: `/Users/john/.claude/skills/`

| Skill | File |
|-------|------|
| project-brief-writer | `project-brief-writer/SKILL.md` |
| tech-stack-advisor | `tech-stack-advisor/SKILL.md` |
| deployment-advisor | `deployment-advisor/SKILL.md` |
| project-spinup | `project-spinup/SKILL.md` |
| test-orchestrator | `test-orchestrator/SKILL.md` |
| ci-cd-implement | `ci-cd-implement/SKILL.md` |
| deploy-guide | `deploy-guide/SKILL.md` |
| workflow-status | `workflow-status/SKILL.md` |

---

*Document revised 2025-12-10 with finalized decisions*
