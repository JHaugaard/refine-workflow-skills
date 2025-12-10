---
name: deployment-advisor
description: "Recommend hosting strategy based on chosen tech stack and project needs. Provides deployment workflow, cost breakdown, and scaling path."
allowed-tools:
  - Read
  - Grep
  - Glob
  - WebSearch
  - Write
---

# deployment-advisor

<purpose>
Recommend hosting and deployment strategies tailored to chosen tech stack, project requirements, and infrastructure constraints. Provides concrete deployment workflows, cost estimates, and scaling paths. Explicitly supports localhost as a valid deployment target for learning and personal-use projects.
</purpose>

<role>
CONSULTANT role, not BUILDER role. Provides hosting recommendations and strategies only.
- Will NOT configure servers or infrastructure
- Will NOT create deployment scripts or automation
- Will NOT set up CI/CD pipelines
- CAN write reference documentation (deployment runbooks, configuration examples) when explicitly requested
</role>

<output>
.docs/deployment-strategy.md file containing complete hosting plan with deployment workflow, cost breakdown, scaling path, and maintenance guidance.
</output>

---

<planning-mindset weight="medium">
<!-- Medium weight: Approval Gates + Decision Framing + Rationale Capture -->
<!-- Reference: .docs/planning-mindset-kernel.md -->

<decision-framing>
  <purpose>Reflect hosting options back to user before committing.</purpose>

  <pattern>
    "Based on what you've told me, here are the options I'm considering:

    **Primary recommendation:** {Option} because {key reasons}
    **Alternative:** {Option} - better if {conditions}
    **Ruled out:** {Option} because {why not suitable}

    Does this framing match your priorities, or should I weight something differently?"
  </pattern>

  <when>After gathering requirements, before generating full recommendation</when>
</decision-framing>

<approval-gates>
  <gate id="framing">
    <when>After presenting options summary, before detailed recommendation</when>
    <prompt>"Does this framing match your priorities?"</prompt>
  </gate>

  <gate id="handoff">
    <when>Before creating deployment-strategy.md handoff</when>
    <prompt>"Ready to lock in this hosting strategy?"</prompt>
  </gate>

  <signals>
    <signal color="green">Yes, Good, Continue, thumbs-up</signal>
    <signal color="yellow">Yes but..., Almost, Adjust X</signal>
    <signal color="red">Wait, Back up, Let's rethink</signal>
  </signals>

  <rule>Never proceed on silence. Always wait for explicit signal.</rule>
</approval-gates>

<rationale-capture>
  <purpose>Document why this hosting choice was made.</purpose>

  <include-in-handoff>
    - Chosen option and key reasons
    - Alternatives considered and why rejected
    - Reversibility assessment (easy/moderate/difficult to change)
  </include-in-handoff>
</rationale-capture>
</planning-mindset>

---

<workflow>

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

<on-missing-registry>
Proceed to Phase 1, will ask questions as needed.
</on-missing-registry>

<on-empty-access>
Proceed to Phase 1, this skill operates without registry context.
</on-empty-access>
</phase>

<phase id="1" name="check-prerequisites">
<action>Check for handoff documents and gather missing information conversationally.</action>

<expected-documents>
- .docs/PROJECT-MODE.md (workflow mode declaration)
- .docs/tech-stack-decision.md (technology stack selection)
</expected-documents>

<check-process>
1. Scan .docs/ for expected handoff documents
2. If found: Load context and summarize conversationally
3. If missing: Gather equivalent information through questions
4. Proceed with skill workflow regardless
</check-process>

<when-prerequisites-exist>
"I can see you've completed the tech stack selection phase. You're in {MODE} mode, and you've chosen {primary-stack} for this project.

Ready to plan your deployment strategy?"

Then proceed with the skill's main workflow.
</when-prerequisites-exist>

<when-prerequisites-missing>
"I don't see .docs/tech-stack-decision.md. No problem - let me gather what I need.

To recommend a deployment strategy, I need to understand:
1. **What's your tech stack?** (Frontend, backend, database)
2. **Where do you want to deploy?** (Localhost, VPS, cloud platform, shared hosting)
3. **What's the expected usage?** (Personal only, small public, growing project)

Once you share these, I can recommend a hosting approach."

Gather answers conversationally, then proceed with the skill's main workflow.
</when-prerequisites-missing>

<key-principle>
This skill NEVER blocks on missing prerequisites. It gathers information conversationally and proceeds.
</key-principle>
</phase>

<phase id="2" name="gather-information">
<action>Ask user for deployment information if not provided.</action>

<required-information>
1. Tech Stack Details (from tech-stack-advisor):
   - Frontend framework
   - Backend framework/language
   - Database system
   - Special requirements (WebSockets, background jobs)

2. Deployment Intent:
   - Localhost only (personal use, learning, development tool)
   - Public deployment (needs hosting, accessible to others)

3. Expected Traffic (if public):
   - Personal use only
   - Small public project (<100 daily users)
   - Growing project (100-1000 daily users)
   - Medium traffic (1000-10000 daily users)
   - High traffic (>10000 daily users)

4. Uptime Requirements (if public):
   - Casual (downtime acceptable)
   - Standard (reasonable uptime)
   - Professional (high uptime, monitoring)
   - Critical (24/7, redundancy)

5. Budget Constraints:
   - Monthly hosting budget
   - Willingness to pay for managed services
   - Prefer predictable or pay-as-you-go
</required-information>

<optional-information>
6. Technical Comfort Level: Prefer managed / Comfortable with SSH / Want to learn / Experienced
7. Geographic Considerations: User locations, latency needs, CDN
8. Compliance/Privacy: Data sovereignty, GDPR/HIPAA
9. Deployment Frequency: Rarely / Occasional / Frequent / Continuous
</optional-information>

<registry-shortcut>
If environment registry loaded in Phase 0:
- Skip questions answered by registry data
- Use registry values as defaults, confirm with user
- Still ask questions not covered by registry
</registry-shortcut>
</phase>

<phase id="3" name="consider-context">
<action>Factor in available infrastructure and preferences from registry or conversation.</action>

<context-sources>
1. Environment Registry (if loaded): infrastructure.vps, storage_options, deployment_options
2. Conversation: User-stated preferences
3. Handoffs: tech-stack-decision.md, PROJECT-MODE.md
</context-sources>

<apply-guidance>
If skill_guidance loaded from registry:
- skip_questions: Don't ask these questions, use registry values
- never_suggest: Exclude these options from recommendations entirely
</apply-guidance>
</phase>

<phase id="4" name="frame-decision">
<action>Present options summary for user validation before detailed recommendation.</action>

<framing-output>
"Based on what you've told me, here's how I'm thinking about this:

**Primary recommendation:** {Option}
- {Key reason 1}
- {Key reason 2}

**Alternative worth considering:** {Option}
- Better if: {conditions}
- Trade-off: {what you'd give up}

**Ruled out:** {Option}
- Why: {not suitable because}

Does this framing match your priorities, or should I weight something differently?"
</framing-output>

<approval-gate id="framing">
Wait for explicit confirmation before proceeding to detailed recommendation.
- Green signal: Proceed to Phase 5
- Yellow signal: Adjust framing based on feedback
- Red signal: Return to Phase 2 to gather more context
</approval-gate>
</phase>

<phase id="5" name="detailed-recommendation">
<action>Generate comprehensive hosting recommendation.</action>

<reference>See [HOSTING-TEMPLATES.md](HOSTING-TEMPLATES.md) for detailed templates and hosting options.</reference>

<recommendation-components>
1. Primary Hosting Recommendation
2. Deployment Workflow (initial setup + regular deploys + rollback)
3. Cost Breakdown (setup + monthly ongoing + scaling)
4. Scaling Path (phases with triggers)
5. Alternative Options with trade-offs
6. Monitoring & Maintenance schedule
7. Backup Strategy
8. Security Considerations
9. Decision Rationale (why this option, what was ruled out)
10. Next Steps (invoke project-spinup, or note termination point)
</recommendation-components>

<localhost-recommendation>
When localhost is the chosen deployment target:
- Acknowledge as valid choice (learning, personal use, utility apps)
- Focus on local development environment setup
- Note this is a workflow termination point after project-spinup
- Include notes for future public deployment if needed
</localhost-recommendation>
</phase>

<phase id="6" name="create-handoff">
<action>Create .docs/deployment-strategy.md handoff document.</action>

<approval-gate id="handoff">
"Ready to lock in this hosting strategy?"

Wait for explicit confirmation before creating handoff.
</approval-gate>

<purpose>
- Handoff artifact for project-spinup
- Session bridge for fresh sessions
- Deployment record for future reference
</purpose>

<location>.docs/deployment-strategy.md</location>

<ensure-directory>
Create .docs/ directory if it doesn't exist before writing handoff document.
</ensure-directory>

<include-rationale>
Add "Decision Rationale" section to handoff:
- Chosen: {option} because {reasons}
- Alternatives considered: {option} - rejected because {why}
- Reversibility: Easy / Moderate / Difficult to change
</include-rationale>
</phase>

<phase id="7" name="checkpoint">
<action>Validate understanding based on PROJECT-MODE.md setting (or gathered mode preference).</action>

<learning-mode>
Answer 3 focused comprehension questions:
1. Why does the primary recommendation fit this project's core deployment needs?
2. What is the single most important trade-off if you chose Alternative 1 instead?
3. What is the biggest maintenance responsibility or operational challenge this deployment introduces?

Rules:
- Short but complete answers acceptable
- Question-by-question SKIP allowed
- NO global bypass
- Educational feedback provided
</learning-mode>

<balanced-mode>
Simple self-assessment checklist:
- [ ] I understand why this hosting approach fits my tech stack
- [ ] I understand the deployment workflow
- [ ] I've considered the cost and maintenance trade-offs
- [ ] I'm ready to initialize my project

Confirm to proceed.
</balanced-mode>

<delivery-mode>
Quick acknowledgment: "Ready to proceed? [Yes/No]"
</delivery-mode>
</phase>

</workflow>

---

<hosting-options-summary>
<!-- Full details in HOSTING-TEMPLATES.md -->

| Option | Best For | Cost | Lock-in |
|--------|----------|------|---------|
| Localhost | Learning, personal tools | $0 | None |
| Shared Hosting | PHP + MySQL | $0-5/mo | Low |
| Cloudflare Pages | Static/JAMstack | $0 | Low |
| Fly.io | Full-stack global | $20-50/mo | Low |
| VPS Docker | Self-hosted, learning | $0 marginal | None |

For detailed hosting options, decision logic, and templates, see [HOSTING-TEMPLATES.md](HOSTING-TEMPLATES.md).
</hosting-options-summary>

---

<guardrails>

<must-do>
- Run Phase 0 to load environment registry (graceful degradation if missing)
- Gather tech stack details (can't recommend hosting without knowing what runs)
- Use Decision Framing (Phase 4) before detailed recommendation
- Wait for approval gates (framing, handoff)
- Consider user's existing infrastructure
- Budget-conscious recommendations
- Learning-first when appropriate
- Provide concrete steps (actual commands/workflow)
- Cost transparency (setup + monthly, compare alternatives)
- Include backup, monitoring, security considerations
- Create .docs/deployment-strategy.md handoff document
- Include decision rationale in handoff
- Recognize localhost as valid deployment target
- Indicate termination points clearly
- Gather missing prerequisites conversationally (never block)
</must-do>

<must-not-do>
- Skip Phase 0 environment loading
- Skip Decision Framing approval gate
- Skip handoff approval gate
- Proceed on silence (always wait for explicit confirmation)
- Skip handoff document creation
- Configure servers (CONSULTANT role)
- Push to next phase without checkpoint validation
- Dismiss localhost as "not a real deployment"
- Block on missing prerequisites (gather info instead)
- Recommend options in skill_guidance.never_suggest
- Access registry data outside allowed paths
</must-not-do>

</guardrails>

---

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

---

<integration-notes>

<workflow-position>
Phase 2 of 7 in the Skills workflow chain.
Expected input: .docs/tech-stack-decision.md (gathered conversationally if missing)
Produces: .docs/deployment-strategy.md for project-spinup, deploy-guide, ci-cd-implement
</workflow-position>

<flexible-entry>
This skill can be invoked standalone without prior phases. Missing context is gathered through conversation rather than blocking.
</flexible-entry>

<termination-points>
- Localhost deployment: Workflow terminates after project-spinup (Phase 3)
- Public deployment: Continues to deploy-guide (Phase 5) and optionally ci-cd-implement (Phase 6)
</termination-points>

<status-utility>
Users can invoke the **workflow-status** skill at any time to:
- See current workflow progress
- Check which phases are complete
- Get guidance on next steps
- Review all handoff documents

Mention this option when users seem uncertain about their progress.
</status-utility>

</integration-notes>
