# Skill Data Access Pattern

**Created:** 2025-12-07
**Purpose:** Document the selective registry loading pattern for Skills workflow
**Status:** Ready to implement
**Related:** [environment-registry-planning.md](environment-registry-planning.md), [workflow-refinement.md](workflow-refinement.md)

---

## Summary

This document describes a pattern for skills to selectively load only the registry data they need, without human intervention at runtime. The registry itself declares what each skill can see.

---

## The Problem

The Environment Registry (`~/.claude/environment.json`) contains comprehensive infrastructure data (~300+ lines). Loading the entire registry into every skill:

- Wastes context tokens on irrelevant data
- Risks skills reasoning about out-of-scope information
- Provides no enforcement of scope boundaries

We need skills to automatically extract only what they need.

---

## The Solution: Registry-Declared Access Control

Add a `skill_data_access` block to the Environment Registry that maps each skill to the registry paths it should see.

### Registry Addition

Add this block to `~/.claude/environment.json`:

```json
{
  "skill_data_access": {
    "project-brief-writer": [],
    "tech-stack-advisor": [
      "database_options",
      "skill_guidance.preferences"
    ],
    "deployment-advisor": [
      "deployment_options",
      "infrastructure.vps",
      "storage_options",
      "skill_guidance.skip_questions",
      "skill_guidance.never_suggest"
    ],
    "project-spinup": [
      "infrastructure.services",
      "credentials.api_endpoints",
      "established_choices"
    ],
    "test-orchestrator": [
      "established_choices.containerization"
    ],
    "ci-cd-implement": [
      "deployment_options",
      "external_accounts.github"
    ],
    "deploy-guide": [
      "infrastructure",
      "credentials",
      "established_choices"
    ]
  }
}
```

### Placement in Registry

Insert after the `version` and metadata fields, before `infrastructure`:

```json
{
  "$schema": "./environment-schema.json",
  "version": "1.0",
  "last_updated": "2025-12-07",
  "owner": "john",

  "skill_data_access": {
    ...
  },

  "infrastructure": {
    ...
  }
}
```

---

## Universal Phase 0 Template

Add this phase to every skill that needs registry access. It replaces any existing environment loading logic.

```xml
<phase id="0" name="load-environment">
  <description>Load environment context scoped to this skill</description>

  <steps>
    1. Attempt to read ~/.claude/environment.json
    2. If not found:
       - Note: "No environment registry found. Will ask questions as needed."
       - Proceed to Phase 1
    3. If found:
       a. Read skill_data_access["{this-skill-name}"]
       b. If skill not listed or array is empty: Proceed with no registry context
       c. Extract ONLY the paths listed in the array
       d. Hold extracted data in working context
       e. Do NOT reference, reason about, or mention any registry data outside the listed paths
  </steps>

  <on-success>Proceed to Phase 1 with environment context loaded</on-success>
  <on-missing-registry>Proceed to Phase 1, will ask questions as needed</on-missing-registry>
  <on-empty-access>Proceed to Phase 1, this skill operates without registry context</on-empty-access>
</phase>
```

### Skill-Specific Customization

Replace `{this-skill-name}` with the actual skill name. Example for `deployment-advisor`:

```xml
<phase id="0" name="load-environment">
  <description>Load environment context scoped to deployment-advisor</description>

  <steps>
    1. Attempt to read ~/.claude/environment.json
    2. If not found:
       - Note: "No environment registry found. Will ask questions as needed."
       - Proceed to Phase 1
    3. If found:
       a. Read skill_data_access["deployment-advisor"]
       b. Extract ONLY: deployment_options, infrastructure.vps, storage_options, skill_guidance.skip_questions, skill_guidance.never_suggest
       c. Hold extracted data in working context
       d. Do NOT reference any other registry data
  </steps>

  <on-success>
    I now know:
    - Allowed deployment targets (will ONLY recommend from this list)
    - Available VPS infrastructure
    - Storage options
    - Questions to skip
    - Recommendations to avoid

    Proceed to Phase 1.
  </on-success>
</phase>
```

---

## Maintenance Requirements

### When Adding a New Skill

1. Determine which registry sections the skill needs
2. Add an entry to `skill_data_access` in the registry
3. Add Phase 0 to the skill definition

### When Restructuring the Registry

1. Update `skill_data_access` paths to reflect new structure
2. Skills with the Universal Phase 0 template need no changes (they read paths dynamically)

### When a Skill's Needs Change

1. Update the skill's entry in `skill_data_access`
2. No changes needed to the skill itself (if using dynamic path reading)

---

## Benefits

| Benefit | How This Achieves It |
|---------|---------------------|
| Automatic extraction | Skills read their allowed paths from registry, no human decision at runtime |
| Single source of truth | Access control lives in one place (the registry) |
| Context efficiency | Only relevant data loaded, rest discarded |
| Scope enforcement | Skills literally don't see out-of-scope data |
| Graceful degradation | Missing registry = fall back to asking questions |
| Maintainability | One place to update when skills or registry change |

---

## Skills Without Registry Access

Some skills don't need registry data:

- **project-brief-writer**: Pure problem/solution capture, no infrastructure context needed

For these, `skill_data_access` contains an empty array:

```json
"project-brief-writer": []
```

The Phase 0 template handles this gracefully (proceeds with no registry context).

---

## Implementation Checklist

When ready to implement:

- [ ] Add `skill_data_access` block to `~/.claude/environment.json`
- [ ] Add Phase 0 to `deployment-advisor` (primary consumer, good test case)
- [ ] Test with a real project
- [ ] Roll out Phase 0 to remaining skills:
  - [ ] tech-stack-advisor
  - [ ] project-spinup
  - [ ] test-orchestrator
  - [ ] ci-cd-implement
  - [ ] deploy-guide
- [ ] Update project-brief-writer with empty-access Phase 0 (optional, for consistency)

---

## Related Decisions

This pattern complements:

- **DECISIONS.json**: Tracks locked project decisions (per-project)
- **Environment Registry**: Tracks infrastructure facts (user-level)
- **JSON Handoffs**: Structured skill-to-skill communication

Together, these eliminate ambiguity and reduce repetitive questioning across the workflow.

---

*Document created during Claude Code session on 2025-12-07*
*Port this to the refine-workflow-skills project for implementation*
