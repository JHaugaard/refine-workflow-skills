# Workflow Refinement Study Document

**Created:** 2025-12-04
**Purpose:** Capture decisions, architecture, and open questions for refining the Skills workflow
**Status:** In Progress - Awaiting Review

---

## Table of Contents

1. [Context & Problem Statement](#context--problem-statement)
2. [The Workflow Overview](#the-workflow-overview)
3. [Identified Problems](#identified-problems)
4. [Proposed Solutions](#proposed-solutions)
5. [Architecture Decisions](#architecture-decisions)
6. [Implementation Approach](#implementation-approach)
7. [Open Questions Requiring Decisions](#open-questions-requiring-decisions)
8. [Reference: Anthropic's Guidance](#reference-anthropics-guidance)
9. [Next Steps](#next-steps)

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

### Skill Sequence (Revised)

```
Phase 0: project-brief-writer
    ↓
Phase 1: tech-stack-advisor
    ↓
Phase 2: deployment-advisor
    ↓
Phase 3: project-spinup ← TERMINATION POINT (localhost projects)
    ↓
    [User builds features]
    ↓
Phase 4: test-orchestrator (optional, can run standalone)
    ↓
Phase 5: ci-cd-implement (optional, can run standalone)
    ↓
Phase 6: deploy-guide ← TERMINATION POINT (manual deploy)
    ↓
    [If CI/CD desired, ci-cd-implement runs after deploy-guide]
    ← TERMINATION POINT (full automation)
```

**Note:** The original ordering placed deploy-guide immediately after project-spinup. The revised ordering (test-orchestrator → ci-cd-implement → deploy-guide) means deployment happens with a more complete, production-ready application including testing infrastructure and CI/CD pipelines.

**Correction from discussion:** The final agreed sequence is:
```
project-spinup → test-orchestrator → ci-cd-implement → deploy-guide
```

### Skill Roles

| Skill | Role | Scope Boundary |
|-------|------|----------------|
| **project-brief-writer** | Capture WHAT and WHY | NO tech, NO platforms, NO architecture |
| **tech-stack-advisor** | Recommend technologies | NO deployment targets, NO infrastructure |
| **deployment-advisor** | Recommend hosting strategy | NO code snippets, NO Docker configs, NO .env contents |
| **project-spinup** | Generate project foundation | Uses decisions from above, NO new tech/deployment decisions |
| **test-orchestrator** | Set up testing infrastructure | Analyzes existing project, NO tech stack changes |
| **ci-cd-implement** | Create CI/CD pipelines | Reads deployment target, NO deployment execution |
| **deploy-guide** | Execute deployment | Follows established strategy, NO strategy changes |
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
- User context sections in some skills create implicit biases

**Evidence from current skills:**
- `deployment-advisor` lines 116-134: Embedded `<user-context>` with VPS specs
- `project-spinup` lines 430-490: Detailed tech stack templates that could influence decisions
- Skills reference what previous phases "should have" produced but don't validate

### Problem 2: Decision Drift

**Symptoms:**
- User decides "deploy everything on fly.io" during deployment-advisor discussion
- Later, deploy-guide or ci-cd-implement starts splitting between fly.io and self-hosted
- Key decisions are buried in prose within handoff documents

**Root Causes:**
- No single source of truth for decisions
- No concept of "LOCKED" vs "tentative" decisions
- Decisions scattered across multiple markdown documents
- Skills might read one document but miss related decisions in another
- No cross-reference validation between skills

**Evidence:**
- Current handoffs are prose-heavy markdown
- No structured decision registry
- No validation that a skill's output aligns with prior decisions

---

## Proposed Solutions

### Solution A: Hard Boundaries at Top of Each Skill

Move critical guardrails to the very top of each skill definition and phrase them as imperatives:

```xml
<hard-boundaries>
BEFORE ANY OUTPUT, I MUST VERIFY:
1. I will NOT suggest technologies - that is tech-stack-advisor's scope
2. I will NOT suggest deployment platforms - that is deployment-advisor's scope
3. I will ONLY work within: [explicit scope definition]

If a previous decision seems wrong, I will ASK THE USER before contradicting it.
</hard-boundaries>
```

### Solution B: Explicit Scope Lists

Replace prose descriptions with explicit in-scope/out-of-scope lists:

```xml
<scope-boundaries>
  <in-scope>
    <item>Deployment workflow steps</item>
    <item>Cost estimates for hosting</item>
    <item>Scaling recommendations</item>
    <item>Backup strategies</item>
  </in-scope>

  <out-of-scope>
    <item reason="tech-stack-advisor">Technology recommendations</item>
    <item reason="project-spinup">.env file contents</item>
    <item reason="project-spinup">Docker configuration details</item>
    <item reason="ci-cd-implement">CI/CD pipeline setup</item>
  </out-of-scope>
</scope-boundaries>
```

### Solution C: JSON Decision Registry

Create a single `.docs/DECISIONS.json` file that accumulates all locked decisions:

```json
{
  "project": "my-project",
  "mode": "LEARNING",
  "last_updated": "2025-12-04",

  "decisions": [
    {
      "id": "D001",
      "phase": 1,
      "skill": "tech-stack-advisor",
      "category": "tech-stack",
      "key": "frontend",
      "value": "Next.js 15",
      "status": "LOCKED",
      "rationale": "Modern React patterns, SSR support",
      "locked_at": "2025-12-04"
    },
    {
      "id": "D002",
      "phase": 2,
      "skill": "deployment-advisor",
      "category": "deployment",
      "key": "target",
      "value": "fly.io",
      "status": "LOCKED",
      "rationale": "Full-stack with managed database",
      "locked_at": "2025-12-04"
    },
    {
      "id": "D003",
      "phase": 2,
      "skill": "deployment-advisor",
      "category": "deployment",
      "key": "database_hosting",
      "value": "fly.io managed postgres",
      "status": "LOCKED",
      "rationale": "Keeps everything on one platform",
      "locked_at": "2025-12-04"
    }
  ],

  "workflow_state": {
    "current_phase": 3,
    "completed_phases": [0, 1, 2],
    "termination_eligible": false
  }
}
```

### Solution D: JSON Handoff Documents

Replace prose markdown handoffs with structured JSON:

```json
{
  "document_type": "tech-stack-decision",
  "version": "1.0",
  "created": "2025-12-04",
  "project": "my-project",
  "mode": "LEARNING",

  "primary_recommendation": {
    "name": "Next.js + Supabase",
    "frontend": "Next.js 15 (App Router)",
    "backend": "Next.js API Routes",
    "database": "Supabase (PostgreSQL)",
    "auth": "Supabase Auth",
    "styling": "Tailwind CSS v4"
  },

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

  "user_stated_preferences": {
    "noted": ["interested in fly.io"],
    "binding": false
  },

  "handoff_to": ["deployment-advisor", "project-spinup"]
}
```

### Solution E: Mandatory Decision Verification Phase

Each skill starts by reading and displaying locked decisions:

```xml
<phase id="0" name="verify-decisions">
<action>Read DECISIONS.json, display relevant locked decisions, get confirmation</action>

<display-to-user>
I found these locked decisions from previous phases:

**From tech-stack-advisor:**
- Frontend: Next.js 15 [LOCKED]
- Database: Supabase [LOCKED]

**From deployment-advisor:**
- Target: fly.io (app + database) [LOCKED]
- Environment: Production only [LOCKED]

I will work within these constraints. Proceed? [yes / need to revise a decision]
</display-to-user>

<if-user-wants-revision>
To revise a locked decision, you'll need to re-run the skill that made it.
Which decision do you want to reconsider?
</if-user-wants-revision>
</phase>
```

---

## Architecture Decisions

### Decision 1: XML for Skills, JSON for Data

| Component | Format | Rationale |
|-----------|--------|-----------|
| Skill definitions (SKILL.md) | XML with semantic tags | Claude was trained on XML; provides "semantic anchoring" for complex prompts |
| Handoff documents | JSON | Machine-readable, schema-validatable, unambiguous |
| Decision registry | JSON | Structured, queryable, prevents prose-buried decisions |
| Human summaries | Minimal markdown | Only where user needs to read (README, etc.) |

**Anthropic's position:** XML tags are recommended for prompt structure ("can be a game-changer"). JSON is recommended for structured outputs with their new constrained decoding feature.

### Decision 2: Single Decision Registry

All locked decisions accumulate in `.docs/DECISIONS.json` rather than being scattered across multiple files.

**Benefits:**
- Single source of truth
- Easy to display "what's been decided"
- Easy to validate new actions against existing decisions
- workflow-status can read one file to show complete state

### Decision 3: Handoffs Are JSON, Not Markdown

Each skill produces a JSON handoff document in addition to (or instead of) markdown.

**Naming convention:**
- `.docs/handoff-phase-0-brief.json`
- `.docs/handoff-phase-1-tech-stack.json`
- `.docs/handoff-phase-2-deployment.json`
- `.docs/DECISIONS.json` (cumulative registry)

### Decision 4: Hard Boundaries at Top of Skills

Critical scope restrictions go at the very top of each skill, not buried in guardrails at the bottom.

---

## Implementation Approach

### Option A: Big Bang Refactor

Refactor all 8 skills simultaneously to the new architecture.

**Pros:** Consistent from the start
**Cons:** Large effort, harder to test incrementally

### Option B: Incremental Migration

Start with one handoff pair (e.g., project-brief-writer → tech-stack-advisor), prove the pattern works, then extend.

**Pros:** Lower risk, can learn and adjust
**Cons:** Temporary inconsistency between old and new skills

### Option C: Registry-First

Implement DECISIONS.json and the verification phase first, without changing handoff formats. Then migrate handoffs to JSON.

**Pros:** Solves the "decision drift" problem immediately
**Cons:** Two-phase effort

### Recommended Approach

**Option C (Registry-First)** followed by **Option B (Incremental Migration)**:

1. Define DECISIONS.json schema
2. Add Phase 0 "verify-decisions" to all skills
3. Migrate project-brief-writer → tech-stack-advisor handoff to JSON
4. Test the pattern
5. Migrate remaining handoffs one by one

---

## Open Questions Requiring Decisions

### Question 1: JSON Schema Strictness

Should we define formal JSON schemas (with validation) for each handoff type?

**Option A:** Strict schemas with Pydantic/Zod-style definitions
- Pro: Catches malformed handoffs immediately
- Con: More upfront definition work

**Option B:** Flexible schemas with required fields only
- Pro: Faster to implement
- Con: May allow inconsistencies

**My lean:** Start with Option B (required fields only), add strictness if problems emerge.

---

### Question 2: Decision Locking Ceremony

When does a decision become LOCKED?

**Option A:** Auto-lock when skill completes
- Pro: Simple, no extra step
- Con: User might not realize decisions are locked

**Option B:** Explicit user confirmation to lock
- Pro: User consciously commits
- Con: Extra friction, might be skipped

**Option C:** Lock on handoff creation, but allow "unlock and revise" by re-running the skill
- Pro: Balance of commitment and flexibility
- Con: Need to handle "re-run" logic

**My lean:** Option C - lock on handoff, revise by re-running the originating skill.

---

### Question 3: workflow-status Role

Should workflow-status become an active gatekeeper or remain passive?

**Option A:** Passive display only
- workflow-status shows state but doesn't enforce anything
- Each skill handles its own validation

**Option B:** Active gatekeeper
- Other skills must "check in" with workflow-status
- workflow-status validates before allowing skill to proceed

**Option C:** Hybrid
- workflow-status provides validation functions/templates
- Each skill uses them but runs independently

**My lean:** Option A or C - keep skills autonomous but with shared validation patterns.

---

### Question 4: User Context Handling

The current skills embed user-specific context (VPS specs, preferences). Should this:

**Option A:** Stay in skills (current approach)
- Pro: Skills have context they need
- Con: Creates implicit bias

**Option B:** Move to a separate user-profile.json
- Pro: Separates user context from skill logic
- Con: Another file to manage

**Option C:** Remove entirely, gather conversationally
- Pro: No implicit bias
- Con: More questions every time

**My lean:** Option B - separate user-profile.json that skills can reference but that doesn't live inside skill definitions.

---

### Question 5: Handoff File Naming

What naming convention for JSON handoffs?

**Option A:** By phase number
- `.docs/handoff-phase-0.json`
- `.docs/handoff-phase-1.json`

**Option B:** By skill name
- `.docs/handoff-project-brief.json`
- `.docs/handoff-tech-stack.json`

**Option C:** By document type
- `.docs/brief.json`
- `.docs/tech-stack-decision.json`
- `.docs/deployment-strategy.json`

**My lean:** Option C - descriptive names are clearer than phase numbers.

---

### Question 6: Backward Compatibility

Should skills still produce markdown for human reference, or go JSON-only?

**Option A:** JSON only
- Pro: Single source of truth
- Con: Harder for humans to review

**Option B:** JSON primary + markdown summary
- Pro: Best of both worlds
- Con: Maintenance burden, potential drift

**Option C:** JSON with a "render to markdown" utility
- Pro: Single source, human-readable on demand
- Con: Need to build the utility

**My lean:** Option A for now (JSON only). Markdown can be generated later if needed.

---

## Reference: Anthropic's Guidance

### On XML Tags (for prompts/skills)

From [Anthropic's documentation](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/use-xml-tags):

> "XML tags can be a game-changer. They help Claude parse your prompts more accurately, leading to higher-quality outputs."

**Key benefits:**
- Clarity — separating prompt components prevents confusion
- Accuracy — reduces misinterpretation errors
- Flexibility — easy modification without full rewrites
- Parseability — structured output enables post-processing

**Best practices:**
- Consistency in tag names
- Nesting for hierarchical content
- Semantic naming (tag names reflect content)

### On JSON (for structured outputs)

From [Anthropic's Structured Outputs documentation](https://platform.claude.com/docs/en/build-with-claude/structured-outputs):

- Native structured outputs (November 2025) use **constrained decoding**
- The model literally cannot produce tokens that violate the schema
- **Zero JSON parsing errors** - no retries, no validation loops
- Supports Pydantic (Python) and Zod (TypeScript)

### XML vs JSON Summary

| Use Case | Recommended Format |
|----------|-------------------|
| Prompt structure (skills) | XML |
| Data interchange (handoffs) | JSON |
| Decision registry | JSON |
| Human documentation | Markdown (minimal) |

**Key insight from research:** XML creates "semantic anchoring" - discrete boundary conditions that help models maintain context. JSON's bracket structure can create "nesting anxiety" in complex reasoning, but is ideal for structured data output.

---

## Next Steps

### Immediate Actions (Before Next Session)

1. **Review this document** - Identify questions, concerns, or disagreements
2. **Make decisions** on the open questions above
3. **Prioritize** which problems to solve first

### When Ready to Implement

1. Define DECISIONS.json schema
2. Create Phase 0 "verify-decisions" template
3. Refactor project-brief-writer as proof of concept
4. Test with tech-stack-advisor
5. Iterate based on findings
6. Roll out to remaining skills

### Session Continuity

This document serves as context anchor. To resume:
- Reference this file path: `docs/workflow-refinement.md`
- State which open questions have been decided
- Indicate which implementation phase to begin

---

## Appendix: Current Skill Files

For reference, all skills are located in:
```
/Volumes/dev/skill-foundry/refine-workflow-skills-json/skills-working-copies/
```

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

*Document generated by Claude Code session on 2025-12-04*
