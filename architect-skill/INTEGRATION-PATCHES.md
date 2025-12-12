# Integration Patches for solution-architect

These patches update tech-stack-advisor and deployment-advisor to read `.docs/architecture-context.json` from the new solution-architect skill.

---

## Patch 1: tech-stack-advisor/SKILL.md

### Location: Phase 0 (initialize) - expected-documents section

**FIND:**
```xml
<expected-documents>
- .docs/PROJECT-MODE.md (workflow mode declaration)
- .docs/brief.json (structured project brief)
</expected-documents>
```

**REPLACE WITH:**
```xml
<expected-documents>
- .docs/PROJECT-MODE.md (workflow mode declaration)
- .docs/brief.json (structured project brief)
- .docs/architecture-context.json (architectural decisions from solution-architect) [optional but recommended]
</expected-documents>
```

---

### Location: Phase 0 (initialize) - check-process section

**FIND:**
```xml
<check-process>
1. Load environment context (if available)
2. Scan .docs/ for expected handoff documents
3. If found: Load context and summarize conversationally
4. If missing: Gather equivalent information through questions
5. Proceed with skill workflow regardless
</check-process>
```

**REPLACE WITH:**
```xml
<check-process>
1. Load environment context (if available)
2. Scan .docs/ for expected handoff documents
3. If architecture-context.json found:
   - Load architectural constraints (platform, integrations, scale, team, constraints)
   - These CONSTRAIN tech stack recommendations (don't re-ask resolved questions)
   - Note: Platform decision is LOCKED if present—recommend implementations, not alternative platforms
4. If found: Load context and summarize conversationally
5. If missing: Gather equivalent information through questions
6. Proceed with skill workflow regardless
</check-process>
```

---

### Location: Phase 0 (initialize) - when-prerequisites-exist section

**FIND:**
```xml
<when-prerequisites-exist>
"I can see you've completed the project brief phase. Your project is in {MODE} mode, and your brief describes {brief-summary}.

Ready to explore technology stack options?"

Then proceed with the skill's main workflow.
</when-prerequisites-exist>
```

**REPLACE WITH:**
```xml
<when-prerequisites-exist>
If architecture-context.json exists:
"I can see you've completed both the project brief and architecture context phases. Your project is in {MODE} mode, targeting **{platform}** as a **{project-type}** with **{scale-profile}** scale.

Your architecture context has already resolved:
- Platform: {platform} {native_capabilities if any}
- Integration profile: {integration-profile}
- Scale: {scale-profile} ({concurrent-users})
- Key constraints: {constraints-summary}

I'll recommend tech stacks that fit these architectural decisions. Ready to explore options?"

If only brief.json exists (no architecture-context.json):
"I can see you've completed the project brief phase. Your project is in {MODE} mode, and your brief describes {brief-summary}.

Note: You haven't run solution-architect yet. I can still recommend a tech stack, but I may need to ask some architectural questions (platform, integrations, scale) that solution-architect would normally resolve.

Ready to proceed, or would you prefer to run solution-architect first?"

Then proceed with the skill's main workflow.
</when-prerequisites-exist>
```

---

### Location: Phase 1 (gather-information) - NEW section to add after platform-signals

**ADD after platform-signals section:**
```xml
<architecture-context-integration>
If .docs/architecture-context.json exists, the following are ALREADY DECIDED and should NOT be re-asked:

From architecture-context.json:
- project_context.type → Greenfield/Brownfield/Migration (affects stack constraints)
- project_context.existing_system → If brownfield, existing tech stack to match/integrate
- platform.primary → Platform is LOCKED (recommend implementations, not alternatives)
- platform.native_capabilities → Native APIs needed (affects framework choice)
- platform.offline_requirement → Offline needs (affects state management, sync strategy)
- integrations.required → Systems that must integrate (affects backend choices)
- scale.profile → Scale expectations (affects database, architecture patterns)
- scale.concurrent_users → Concurrency needs (affects backend framework)
- team.primary_skills → Existing expertise (influences framework recommendations)
- team.learning_goals → What user wants to learn (prioritize in recommendations)
- team.avoid → Technologies to rule out
- constraints.hard → Non-negotiable constraints (filter recommendations)

Use these to CONSTRAIN recommendations rather than asking about them again.

Example application:
- If platform.primary = "ios" → Recommend SwiftUI vs React Native vs Flutter, not "should this be a web app?"
- If project_context.type = "brownfield" with existing_system.tech_stack = "React/Express" → Recommend compatible additions
- If integrations.required includes a specific API → Factor into backend framework choice
- If scale.profile = "personal" → Don't over-engineer, recommend simpler solutions
</architecture-context-integration>
```

---

### Location: Phase 2 (consider-user-context) - default-project-profile section

**FIND:**
```xml
<default-project-profile>
Unless the user explicitly states otherwise, assume:
- Single user application (not multi-tenant)
- Private access (authenticated, no public exposure desired)
- Web-based deployment (browser accessible)
```

**REPLACE WITH:**
```xml
<default-project-profile>
If architecture-context.json exists, use its values instead of defaults:
- scale.profile → "personal" = single user, "team"/"organization"/"public" = multi-user
- platform.primary → Use specified platform instead of assuming web
- constraints → May specify access model

If architecture-context.json is missing, assume:
- Single user application (not multi-tenant)
- Private access (authenticated, no public exposure desired)
- Web-based deployment (browser accessible)
```

---

### Location: workflow-status section

**FIND:**
```xml
<workflow-status>
Phase 1 of 7: Technology Stack Selection

Status:
  Phase 0: Project Brief (project-brief-writer)
  Phase 1: Tech Stack Advisor (you are here)
  Phase 2: Deployment Strategy (deployment-advisor)
  Phase 3: Project Foundation (project-spinup) <- TERMINATION POINT
  Phase 4: Test Strategy (test-orchestrator) - optional
  Phase 5: Deployment (deploy-guide) <- TERMINATION POINT
  Phase 6: CI/CD (ci-cd-implement) <- TERMINATION POINT
</workflow-status>
```

**REPLACE WITH:**
```xml
<workflow-status>
Phase 2 of 8: Technology Stack Selection

Status:
  Phase 0: Project Brief (project-brief-writer)
  Phase 1: Architecture Context (solution-architect) [recommended]
  Phase 2: Tech Stack Advisor (you are here)
  Phase 3: Deployment Strategy (deployment-advisor)
  Phase 4: Project Foundation (project-spinup) <- TERMINATION POINT
  Phase 5: Test Strategy (test-orchestrator) - optional
  Phase 6: Deployment (deploy-guide) <- TERMINATION POINT
  Phase 7: CI/CD (ci-cd-implement) <- TERMINATION POINT
</workflow-status>
```

---

### Location: integration-notes section - workflow-position

**FIND:**
```xml
<workflow-position>
Phase 1 of 7 in the Skills workflow chain.
Expected input: .docs/PROJECT-MODE.md, .docs/brief.json (gathered conversationally if missing)
Produces: .docs/tech-stack-decision.json for deployment-advisor
</workflow-position>
```

**REPLACE WITH:**
```xml
<workflow-position>
Phase 2 of 8 in the Skills workflow chain.
Expected input:
- .docs/PROJECT-MODE.md (mode declaration)
- .docs/brief.json (project requirements)
- .docs/architecture-context.json (architectural constraints) [optional but recommended]

If architecture-context.json exists, architectural decisions (platform, integrations, scale, constraints) are LOCKED and constrain recommendations.

Produces: .docs/tech-stack-decision.json for deployment-advisor and project-spinup
</workflow-position>
```

---

## Patch 2: deployment-advisor/SKILL.md

### Location: Phase 0 (initialize) - expected-documents section

**FIND:**
```xml
<expected-documents>
- .docs/PROJECT-MODE.md (workflow mode declaration)
- .docs/tech-stack-decision.json (structured technology stack selection)
</expected-documents>
```

**REPLACE WITH:**
```xml
<expected-documents>
- .docs/PROJECT-MODE.md (workflow mode declaration)
- .docs/architecture-context.json (architectural decisions) [optional]
- .docs/tech-stack-decision.json (structured technology stack selection)
</expected-documents>
```

---

### Location: Phase 0 (initialize) - check-process section

**FIND:**
```xml
<check-process>
1. Load environment context (if available)
2. Scan .docs/ for expected handoff documents
3. If found: Load context and summarize conversationally
4. If missing: Gather equivalent information through questions
5. **Check for native platform**: If tech-stack-decision.json contains `platform.primary` of "ios", "macos", or "tauri" → skip to native-platform-handoff phase
6. Proceed with skill workflow regardless
</check-process>
```

**REPLACE WITH:**
```xml
<check-process>
1. Load environment context (if available)
2. Scan .docs/ for expected handoff documents
3. If architecture-context.json exists:
   - Load scale profile (affects hosting recommendations)
   - Load integration requirements (may need specific hosting capabilities)
   - Load constraints (budget, compliance, data residency)
   - Platform decision may also be here (check both files)
4. If found: Load context and summarize conversationally
5. If missing: Gather equivalent information through questions
6. **Check for native platform**: If architecture-context.json OR tech-stack-decision.json contains platform.primary of "ios", "macos", or "tauri" → skip to native-platform-handoff phase
7. Proceed with skill workflow regardless
</check-process>
```

---

### Location: Phase 1 (gather-information) - required-information section

**ADD after the existing required-information list:**
```xml
<architecture-context-integration>
If .docs/architecture-context.json exists, the following are ALREADY DECIDED:

- scale.profile → Informs hosting size requirements
- scale.concurrent_users → Affects scaling recommendations
- scale.availability → Affects redundancy recommendations
- integrations.required → May require specific hosting capabilities (e.g., WebSocket support)
- constraints.hard → Budget, compliance, data residency constraints

Use these to CONSTRAIN hosting recommendations rather than asking about them again.

Example application:
- If scale.profile = "personal" → Recommend localhost or minimal hosting, don't suggest enterprise solutions
- If constraints includes budget "$0" → Focus on free tier options or localhost
- If constraints includes compliance "HIPAA" → Rule out certain hosting options
- If integrations.required includes real-time → Ensure hosting supports WebSockets
</architecture-context-integration>
```

---

### Location: workflow-status section

**FIND:**
```xml
<workflow-status>
Phase 2 of 7: Deployment Strategy

Status:
  Phase 0: Project Brief (project-brief-writer)
  Phase 1: Tech Stack (tech-stack-advisor)
  Phase 2: Deployment Strategy (you are here)
  Phase 3: Project Foundation (project-spinup) <- TERMINATION POINT (localhost)
  Phase 4: Test Strategy (test-orchestrator) - optional
  Phase 5: Deployment (deploy-guide) <- TERMINATION POINT (manual deploy)
  Phase 6: CI/CD (ci-cd-implement) <- TERMINATION POINT (full automation)
</workflow-status>
```

**REPLACE WITH:**
```xml
<workflow-status>
Phase 3 of 8: Deployment Strategy

Status:
  Phase 0: Project Brief (project-brief-writer)
  Phase 1: Architecture Context (solution-architect) [recommended]
  Phase 2: Tech Stack (tech-stack-advisor)
  Phase 3: Deployment Strategy (you are here)
  Phase 4: Project Foundation (project-spinup) <- TERMINATION POINT (localhost)
  Phase 5: Test Strategy (test-orchestrator) - optional
  Phase 6: Deployment (deploy-guide) <- TERMINATION POINT (manual deploy)
  Phase 7: CI/CD (ci-cd-implement) <- TERMINATION POINT (full automation)
</workflow-status>
```

---

### Location: integration-notes section - workflow-position

**FIND:**
```xml
<workflow-position>
Phase 2 of 7 in the Skills workflow chain.
Expected input: .docs/tech-stack-decision.json (gathered conversationally if missing)
Produces: .docs/deployment-strategy.json for project-spinup, deploy-guide, ci-cd-implement
</workflow-position>
```

**REPLACE WITH:**
```xml
<workflow-position>
Phase 3 of 8 in the Skills workflow chain.
Expected input:
- .docs/architecture-context.json (scale, constraints) [optional]
- .docs/tech-stack-decision.json (tech stack decisions)

If architecture-context.json exists, scale and constraint decisions are already resolved.

Produces: .docs/deployment-strategy.json for project-spinup, deploy-guide, ci-cd-implement
</workflow-position>
```

---

## Summary of Changes

### tech-stack-advisor
1. Added architecture-context.json to expected documents
2. Updated check-process to load architectural constraints
3. Updated when-prerequisites-exist to acknowledge architecture context
4. Added architecture-context-integration section explaining how to use the data
5. Updated default-project-profile to defer to architecture context when present
6. Updated workflow-status to show solution-architect as Phase 1
7. Updated workflow-position to reflect new phase numbering

### deployment-advisor
1. Added architecture-context.json to expected documents
2. Updated check-process to load scale/constraint data from architecture context
3. Added architecture-context-integration section for Phase 1
4. Updated workflow-status to show solution-architect as Phase 1
5. Updated workflow-position to reflect new phase numbering

### Key Principle
Both skills continue to work WITHOUT architecture-context.json (flexible entry). When it exists, they use it to CONSTRAIN recommendations and SKIP questions that have already been answered.
