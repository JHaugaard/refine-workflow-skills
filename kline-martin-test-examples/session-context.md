# Session Context: JSON Handoffs for Skills Workflow

## What We're Exploring

Converting the inter-skill handoff documents from markdown to JSON to improve accuracy and reduce ambiguity when skills consume output from prior phases.

## The Workflow Being Optimized

```
PLANNING PHASE
   Phase 0: project-brief-writer    → .docs/PROJECT-MODE.md, .docs/brief-*.md
   Phase 1: tech-stack-advisor      → .docs/tech-stack-decision.md
   Phase 2: deployment-advisor      → .docs/deployment-strategy.md

SETUP PHASE
   Phase 3: project-spinup          → Project foundation + claude.md
   TERMINATION POINT: Localhost/learning projects

[MANUAL DEVELOPMENT]

QUALITY PHASE
   Phase 4: test-orchestrator       → .docs/test-strategy.md

RELEASE PHASE
   Phase 5: deploy-guide            → .docs/deployment-log.md
   Phase 6: ci-cd-implement         → .github/workflows/*
```

## Core Insight

The handoff documents function as **structured prompts** for the next skill. Each skill is both a prompt consumer and prompt producer. If these are prompts, can we gain efficiency and accuracy by framing them as JSON?

## Hypothesis

JSON handoffs could offer:
- Structured parsing (no prose interpretation)
- Schema validation (required fields enforced)
- Reduced ambiguity (concrete values, not prose descriptions)
- Numerical precision (`memory_mb: 2000` not `"~2 GB"`)
- Composability (easy to merge/reference data programmatically)

## Trade-offs Acknowledged

- Human readability decreases (but these are primarily Claude-to-Claude handoffs)
- Flexibility reduced (prose captures nuance; schemas are rigid)
- User modification harder (JSON syntax vs markdown)

Decision: Optimize for machine consumption since these are primarily Claude-to-Claude handoffs.

## Test Conducted

**Test case:** tech-stack-advisor → deployment-advisor handoff

**Files created:**

1. `tech-stack-decision.json` - Structured version of tech stack output
   - Typed data (numbers as numbers, arrays as arrays)
   - Explicit service definitions with ports/memory
   - Database schema as structured objects
   - Decisions with rationale as array of objects
   - `deployment_hints` section explicitly for next phase

2. `deployment-advisor-json.md` - Modified skill to consume JSON
   - Parses JSON as primary input
   - Direct field extraction (no markdown parsing)
   - Outputs both .md (human) and .json (machine) files

## Test Results (in test-1-outcome/)

**Problem observed:** The JSON-fed skill produced MORE detailed output than the original, crossing into deploy-guide territory.

**Specifics:**
| Section | ORIGINAL | NEW (JSON-fed) |
|---------|----------|----------------|
| Environment Variables | Lists variables + sources | Full `.env` templates |
| Docker Compose | Not included | Complete `docker-compose.yml` |
| Database Schema | Not included | Migration commands |
| Deployment Commands | General workflow | Specific ssh/docker commands |
| Caddy Config | Not included | Actual Caddyfile snippets |

## Root Cause Analysis

1. **JSON enabled overreach**: Structured data made it trivially easy to generate concrete configs. The skill had building blocks and assembled them.

2. **Guardrails failed**: Skill says "CONSULTANT role, not BUILDER" but JSON gave it building blocks. Temptation was too strong.

3. **Root problem isn't JSON format**: It's **skill discipline**. The original markdown version already had some boundary violations (environment variable specifics, etc.).

## Key Realization

The real problem is **skill boundary enforcement**, not data format. Both issues need addressing:

1. **Data format** (JSON vs markdown) - affects parsing reliability
2. **Skill discipline** (who owns what) - affects output appropriateness

These are separate concerns. JSON might help with #1 but exacerbates #2 unless boundaries are explicit.

## Next Steps Identified

1. **Define ownership explicitly**: Which skill owns which data?
   - Example: "Environment variables are OWNED by deploy-guide"
   - Example: "Docker Compose configuration is OWNED by project-spinup or deploy-guide"

2. **Add explicit exclusions to skills**: "DO NOT INCLUDE" lists
   - deployment-advisor: DO NOT include docker-compose, .env files, Caddyfile snippets
   - tech-stack-advisor: DO NOT include deployment topology

3. **Consider hybrid approach**: JSON for structured data, but with clear scope limits

4. **Schema design**: Define what each handoff document MUST contain and MUST NOT contain

## Open Questions

1. Should all handoffs be JSON, or only certain ones?
2. How granular should the ownership model be?
3. Is the problem really about data format, or about Claude being too eager to help?
4. Would tighter skill prompts alone solve this without JSON?

## Files in This Test Directory

```
temp-json-test/
├── session-context.md          # This file
├── PROJECT-MODE.md             # Sample project mode declaration
├── brief-kline-martin-photos.md # Sample project brief
├── tech-stack-decision.md      # Original markdown output
├── tech-stack-decision-v2.md   # Revised markdown output
├── tech-stack-decision.json    # NEW: JSON version we created
├── deployment-strategy.md      # Original deployment strategy
├── project-foundation-complete.md # Sample spinup completion doc
├── deployment-advisor-json.md  # NEW: Modified skill for JSON input
└── test-1-outcome/
    ├── deployment-advisor-json.md    # Copy of modified skill
    ├── deployment-strategy-ORIGINAL.md # Baseline output
    └── deployment-strategy-NEW.md    # JSON-fed output (with problems)
```

## Skills to Include in New Project

Copy these from ~/.claude/skills/:
- tech-stack-advisor/SKILL.md
- deployment-advisor/SKILL.md
- deploy-guide/SKILL.md
- project-spinup/SKILL.md
- ci-cd-implement/SKILL.md

These represent the chain where handoffs matter most.

## Resume Instructions

When resuming:
1. Read this session-context.md
2. Review the test outputs in test-1-outcome/
3. Continue with: defining skill ownership boundaries OR refining JSON schema OR both

## Motivation

"I'm doing this for the learning and the joy."

The goal is not speed but understanding. The skills workflow is a learning tool, and optimizing the handoffs is part of that learning.
