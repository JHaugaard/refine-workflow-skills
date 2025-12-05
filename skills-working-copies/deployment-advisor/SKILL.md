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

<workflow>

<phase id="0" name="check-prerequisites">
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

<phase id="1" name="gather-information">
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
</phase>

<phase id="2" name="consider-user-context">
<action>Factor in available infrastructure and preferences.</action>

<available-infrastructure>
- Hostinger VPS8: 8 cores, 32GB RAM, 400GB storage
  - Supabase (self-hosted), PocketBase, n8n, Ollama, Wiki.js, Caddy
  - Docker/Docker Compose, SSH as user "john"
- Cloudflare DNS
- Backblaze B2 storage
</available-infrastructure>

<user-preferences>
- Prefers self-hosting when practical (learning + cost control)
- Budget-conscious but willing to pay for right solution
- Values learning over convenience
- Comfortable with SSH and terminal basics
- Learning Docker, server management
</user-preferences>
</phase>

<phase id="3" name="analyze-recommend">
<action>Generate comprehensive hosting recommendation.</action>

<recommendation-components>
1. Primary Hosting Recommendation
2. Deployment Workflow (initial setup + regular deploys + rollback)
3. Cost Breakdown (setup + monthly ongoing + scaling)
4. Scaling Path (phases with triggers)
5. Alternative Options with trade-offs
6. Monitoring & Maintenance schedule
7. Backup Strategy
8. Security Considerations
9. Next Steps (invoke project-spinup, or note termination point)
</recommendation-components>

<localhost-recommendation>
When localhost is the chosen deployment target:
- Acknowledge as valid choice (learning, personal use, utility apps)
- Focus on local development environment setup
- Note this is a workflow termination point after project-spinup
- Include notes for future public deployment if needed
</localhost-recommendation>
</phase>

<phase id="4" name="create-handoff">
<action>Create .docs/deployment-strategy.md handoff document.</action>

<purpose>
- Handoff artifact for project-spinup
- Session bridge for fresh sessions
- Deployment record for future reference
</purpose>

<location>.docs/deployment-strategy.md</location>

<timing>After collaborative refinement and user convergence on recommendation.</timing>

<ensure-directory>
Create .docs/ directory if it doesn't exist before writing handoff document.
</ensure-directory>
</phase>

<phase id="5" name="checkpoint">
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

<hosting-options>

<option name="localhost">
<description>Running application on personal computer, not publicly accessible.</description>
<best-for>Learning, personal utility apps, testing before deployment, development tools</best-for>
<cost>$0</cost>
<tech-stacks>Any</tech-stacks>
<pros>No infrastructure to manage, immediate feedback, full control, any database locally, maximum privacy</pros>
<cons>Not accessible outside network, dependent on computer being on, no global distribution</cons>
<when-recommend>Personal-use only apps, learning projects, development utilities, testing environments</when-recommend>
<workflow-note>This is a TERMINATION POINT - workflow complete after project-spinup</workflow-note>
</option>

<option name="hostinger-shared">
<description>Traditional web hosting with cPanel, PHP/MySQL stack.</description>
<best-for>Simple PHP sites, WordPress, static websites</best-for>
<cost>$0 marginal if account exists, otherwise $3-5/month</cost>
<tech-stacks>PHP + MySQL only</tech-stacks>
<pros>$0 marginal cost, managed infrastructure, cPanel interface, minimal maintenance</pros>
<cons>Limited to PHP + MySQL, no Docker, shared resources, single region</cons>
<when-recommend>Simple PHP apps, WordPress, existing shared hosting account</when-recommend>
</option>

<option name="cloudflare-pages">
<description>Serverless platform for static sites and JAMstack on global edge network.</description>
<best-for>Static sites, SPAs, JAMstack, frontend-only projects</best-for>
<cost>$0 (genuinely free, unlimited bandwidth/requests/builds)</cost>
<tech-stacks>Any static generator, React, Vue, Next.js (static), Svelte, Astro</tech-stacks>
<pros>Zero-config deployment, automatic HTTPS, global CDN, git-connected auto-deploy</pros>
<cons>Static/frontend only, no persistent backend, 100MB project limit</cons>
<vendor-lock-in>LOW - vanilla HTML/CSS/JS, portable</vendor-lock-in>
<when-recommend>Frontend-only app, need global performance, want zero cost, already using Cloudflare DNS</when-recommend>
</option>

<option name="fly-io">
<description>Platform for containerized applications globally across 35 regions with managed PostgreSQL.</description>
<best-for>Full-stack with database, global distribution, managed infrastructure</best-for>
<cost>$20-50/month for multiple small projects</cost>
<tech-stacks>Any Docker-based</tech-stacks>
<pros>Docker-based, 35 global regions, auto-scaling, managed PostgreSQL, CLI-first</pros>
<cons>Requires Docker knowledge, pay-per-use pricing, data egress charges</cons>
<vendor-lock-in>LOW - standard Docker containers portable anywhere</vendor-lock-in>
<when-recommend>Full-stack needing database, global distribution, comfortable with Docker, $20-50/month acceptable</when-recommend>
</option>

<option name="vps-docker">
<description>Virtual Private Server running Docker containers for complete control.</description>
<best-for>Full-stack with custom requirements, self-hosted infrastructure, learning DevOps</best-for>
<cost>$40-60/month VPS (marginal cost $0 if already have)</cost>
<tech-stacks>Any</tech-stacks>
<pros>Full control, use existing infrastructure, predictable costs, maximum learning, run any database</pros>
<cons>More maintenance, manage security updates, single point of failure, slower deployment</cons>
<vendor-lock-in>NONE - standard Docker containers, move anywhere</vendor-lock-in>
<when-recommend>Already have VPS with capacity, tech stack runs in Docker, traffic <10k daily, learning is goal, need maximum control</when-recommend>
</option>

</hosting-options>

---

<decision-framework>

<scoring-criteria>
Rate each 1-5:
- Tech Stack Compatibility: Can handle chosen stack? Required services available?
- Performance Requirements: Handle expected traffic? Acceptable latency?
- Cost Efficiency: Fits budget? Predictable costs? Cost-effective as grows?
- Deployment Ease: Complexity? Can user manage it? Automated or manual?
- Maintenance Burden: Weekly maintenance time? Monitoring complexity?
- Learning Value: Teaches useful skills? Industry-relevant?
</scoring-criteria>

<recommendation-logic>

<localhost-when>
- Personal utility app for own use only
- Learning project with no need for public access
- Development tools and scripts
- Testing before public deployment
- Privacy-sensitive personal data
</localhost-when>

<shared-hosting-when>
- Simple PHP-based application
- Traditional LAMP stack preferred
- Minimal maintenance desired
- Want to use existing shared hosting ($0 marginal)
</shared-hosting-when>

<cloudflare-pages-when>
- Frontend-only application
- Static or JAMstack architecture
- Want zero cost
- Need global CDN performance
</cloudflare-pages-when>

<fly-io-when>
- Full-stack app with database
- Need global distribution
- Want managed infrastructure
- Comfortable with Docker
- Budget $20-50/month acceptable
</fly-io-when>

<vps-docker-when>
- Tech stack runs well in Docker
- Traffic low-to-moderate (<10k daily)
- Want to learn DevOps
- Budget-conscious (marginal cost $0)
- Need maximum control
- Want to self-host tools
</vps-docker-when>

</recommendation-logic>

</decision-framework>

---

<deployment-patterns>

<pattern name="localhost-development">
Option: Localhost
Stack: Any
Database: Local PostgreSQL/SQLite/MySQL
Deployment: docker compose up (or native)
Cost: $0/month
Termination: Workflow complete after project-spinup
</pattern>

<pattern name="static-jamstack">
Option: Cloudflare Pages
Stack: React, Vue, Svelte, Astro, Next.js (static)
Database: External API or none
Deployment: Git push (auto-deploy)
Cost: $0/month
</pattern>

<pattern name="full-stack-self-hosted">
Option: VPS with Docker
Stack: Next.js, FastAPI, PHP, Node.js
Database: Self-hosted PostgreSQL/MySQL
Deployment: Git push -> SSH -> docker compose up
Cost: $0/month marginal
</pattern>

<pattern name="full-stack-global">
Option: Fly.io
Stack: Next.js, FastAPI, Node.js, Go
Database: Fly.io managed PostgreSQL
Deployment: fly deploy
Cost: $20-50/month
</pattern>

<pattern name="hybrid-frontend-backend">
Frontend: Cloudflare Pages
Backend API: VPS with Docker
Database: Self-hosted PostgreSQL
Cost: $0/month marginal
Best for: Fast frontend, controlled backend
</pattern>

<pattern name="simple-php">
Option: Hostinger Shared Hosting
Stack: PHP + MySQL
Deployment: FTP or Git
Cost: $3-5/month
</pattern>

</deployment-patterns>

---

<output-format>

<template>
## PRIMARY HOSTING RECOMMENDATION: {Hosting Approach}

{2-3 sentence summary}

### Why This Fits Your Project
- {Reason 1 - tech stack compatibility}
- {Reason 2 - traffic/performance match}
- {Reason 3 - budget alignment}

### Why This Fits Your Infrastructure
- {How it uses existing VPS}
- {Integration with current setup}

### Hosting Details

| Aspect | Value |
|--------|-------|
| Provider | {Hostinger VPS / Shared / Localhost / Other} |
| Server Type | {VPS / Shared / PaaS / Serverless / Local} |
| Container | {Docker / Native / Platform-managed} |
| Database | {Self-hosted / Managed / Local} |
| File Storage | {Local VPS / Backblaze B2 / Local disk} |
| CDN | {Cloudflare / Provider CDN / None} |
| SSL | {Caddy / Let's Encrypt / Cloudflare / None} |

### Deployment Workflow

**Initial Setup (One-time):**
{steps}

**Regular Deployment (Updates):**
{steps}

**Rollback Procedure:**
{steps}

**Deployment Speed:** Fast / Moderate / Slow
**Maintenance Burden:** Low / Medium / High

---

## Cost Breakdown

**Setup Costs:** ~${X}/month amortized
**Monthly Ongoing:** ${Total}/month
**Cost Scaling:** Current -> 10x traffic -> 100x traffic

---

## Scaling Path

**Current State:** {recommendation}
**Phase 1 Optimization:** {when and what}
**Phase 2 Scaling:** {when and what}

**When to Scale:**
- Phase 1: {specific metric}
- Phase 2: {specific metric}

---

## Alternative Hosting Options

### Alternative 1: {approach}

**Pros:** {list}
**Cons:** {list}
**Cost:** ${X}/month
**When to Choose:** {conditions}

---

## Monitoring & Maintenance

**Daily:** {tasks}
**Weekly:** {tasks}
**Monthly:** {tasks}

---

## Backup Strategy

**Database Backups:** {frequency, method, storage, retention}
**File Storage Backups:** {frequency, method}
**Configuration Backups:** {method}
**Disaster Recovery:** {procedure, estimated time}

---

## Security Considerations

**Implemented:** {list}
**Recommended Additions:** {list}

---

## Next Steps

**Handoff document created:** .docs/deployment-strategy.md

[If Localhost:]
1. Proceed to project-spinup to generate your project foundation
2. After project-spinup, your localhost project is ready for development
3. **WORKFLOW TERMINATION POINT** - no further phases needed
4. If you later decide to deploy publicly, re-run deployment-advisor

[If Public Deployment:]
1. Review and ask questions
2. If agreed -> Invoke project-spinup skill
3. Build your features
4. When ready -> Use deploy-guide to deploy
5. Optional -> Use ci-cd-implement for automation
</template>

<localhost-template>
## PRIMARY HOSTING RECOMMENDATION: Localhost

This project will run locally on your development machine.

### Why Localhost
- Personal utility / learning project - no public access needed
- Full control over environment
- Zero hosting costs
- Privacy for personal data
- Immediate development feedback

### Local Development Setup

| Aspect | Value |
|--------|-------|
| Environment | Local machine (Mac) |
| Container | Docker Compose (recommended) or native |
| Database | Local {database type} |
| Storage | Local filesystem |
| Access | localhost only |

### Development Workflow

**Start Development:**
```bash
docker compose up -d  # or native start commands
```

**Access Application:**
- Frontend: http://localhost:3000
- Backend: http://localhost:8000
- Database: localhost:5432

### Cost
$0/month - runs on your existing computer

### Future Public Deployment
If you later decide to deploy publicly:
1. Re-run deployment-advisor
2. Choose a public hosting option
3. Continue with deploy-guide and ci-cd-implement

---

## Next Steps

**Handoff document created:** .docs/deployment-strategy.md

1. Proceed to project-spinup to generate your project foundation
2. After project-spinup, start building your features
3. **WORKFLOW TERMINATION POINT** - no hosting phases needed

Your localhost project workflow is: project-brief-writer -> tech-stack-advisor -> deployment-advisor -> project-spinup -> DONE
</localhost-template>

</output-format>

---

<guardrails>

<must-do>
- Gather tech stack details (can't recommend hosting without knowing what runs)
- Consider user's VPS (leverage when appropriate)
- Budget-conscious recommendations
- Learning-first when appropriate
- Provide concrete steps (actual commands/workflow)
- Cost transparency (setup + monthly, compare alternatives)
- Include backup, monitoring, security considerations
- Create .docs/deployment-strategy.md handoff document
- Recognize localhost as valid deployment target
- Indicate termination points clearly
- Gather missing prerequisites conversationally (never block)
</must-do>

<must-not-do>
- Skip handoff document creation
- Configure servers (CONSULTANT role)
- Push to next phase without checkpoint validation
- Dismiss localhost as "not a real deployment"
- Block on missing prerequisites (gather info instead)
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
