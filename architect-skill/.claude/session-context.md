# Session Context: solution-architect Skill Development

## Session Purpose

Creating a new solution-architect skill that fits between project-brief-writer and tech-stack-advisor in the workflow chain.

## Problem Statement

The current workflow has a gap: project-brief-writer produces a technology-neutral brief, but tech-stack-advisor needs architectural context that the brief intentionally doesn't provide. Key questions like:

- Greenfield vs brownfield?
- Platform constraints (iOS, web, desktop)?
- Integration requirements?
- Scale expectations?
- Team/skill context?

...were falling through the cracks, forcing tech-stack-advisor to ask architectural questions that feel out of scope for a tech stack recommendation.

## Solution

New skill: `solution-architect` that:

1. Reads the brief from project-brief-writer
2. Asks targeted questions to resolve architectural ambiguity
3. Produces `.docs/architecture-context.json` handoff
4. Constrains downstream tech-stack and deployment decisions

## Updated Workflow Chain

```text
Phase 0: project-brief-writer     → brief.json, PROJECT-MODE.md
Phase 1: solution-architect       → architecture-context.json [NEW]
Phase 2: tech-stack-advisor       → tech-stack-decision.json
Phase 3: deployment-advisor       → deployment-strategy.json
Phase 4: project-spinup           ← TERMINATION POINT
Phase 5: test-orchestrator        (optional)
Phase 6: deploy-guide             ← TERMINATION POINT
Phase 7: ci-cd-implement          ← TERMINATION POINT
```

## Work Completed

- [x] Created SKILL.md with full skill definition
  - Six analysis dimensions (project context, platform, integrations, scale, team, constraints)
  - Mode-aware checkpoints (LEARNING/BALANCED/DELIVERY)
  - Handoff document schema (architecture-context.json)
  - Edge case handling
  - Example interactions
- [x] Copied SKILL.md to `/Users/john/.claude/skills/solution-architect/`
- [x] Updated tech-stack-advisor to read architecture-context.json
  - Added to expected documents
  - Added architecture-context-integration section
  - Updated workflow-status to Phase 2 of 8
- [x] Updated deployment-advisor to read architecture-context.json
  - Added to expected documents
  - Added architecture-context-integration section
  - Updated workflow-status to Phase 3 of 8
- [x] Created INTEGRATION-PATCHES.md documenting all changes

## Key Decisions Made

1. **Skill name:** solution-architect (not architect-skill)
2. **Role:** Advisory (asks questions, documents decisions) not prescriptive
3. **Platform handling:** Captures platform decision, lets tech-stack-advisor handle implementation choices (e.g., SwiftUI vs React Native)
4. **Output:** Structured JSON handoff matching existing workflow patterns
5. **Flexible entry:** All skills continue to work without architecture-context.json

## Handoff Document Schema

Location: `.docs/architecture-context.json`

Key sections:

- `project_context`: greenfield/brownfield/migration + existing system details
- `platform`: primary/secondary platforms, native capabilities, offline needs
- `integrations`: required/optional systems, protocols, constraint levels
- `scale`: profile, concurrent users, data volume, availability
- `team`: size, skills, learning goals, technologies to avoid
- `constraints`: hard/soft constraints with impacts
- `architecture_implications`: guidance for downstream skills
- `decisions_locked`: architectural decisions with LOCKED status

## Files Created/Modified

- `architect-skill/SKILL.md` - Main skill definition
- `architect-skill/INTEGRATION-PATCHES.md` - Documentation of downstream skill changes
- `architect-skill/.claude/session-context.md` - This file
- `/Users/john/.claude/skills/solution-architect/SKILL.md` - Deployed skill
- `/Users/john/.claude/skills/tech-stack-advisor/SKILL.md` - Updated with integration
- `/Users/john/.claude/skills/deployment-advisor/SKILL.md` - Updated with integration

## Integration Points

- **Upstream:** Reads `.docs/brief.json` and `.docs/PROJECT-MODE.md` from project-brief-writer
- **Downstream:** tech-stack-advisor and deployment-advisor read `.docs/architecture-context.json`

## MCP Servers for This Session

None needed - this is skill definition work.

## Notes

- The skill follows the same patterns as existing workflow skills:
  - Flexible entry (works with or without prerequisites)
  - Mode-aware checkpoints
  - JSON handoff documents
  - LOCKED decision status
  - handoff_to field for downstream skills
- All downstream skills maintain backward compatibility (work with or without architecture context)
