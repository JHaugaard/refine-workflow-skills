# Hosting Templates Reference

> **Purpose**: Detailed hosting options, deployment patterns, and output templates for deployment-advisor.
> Referenced by: SKILL.md Phase 3 (Analyze & Recommend)

---

## Hosting Options

### Localhost

| Attribute | Value |
|-----------|-------|
| **Description** | Running application on personal computer, not publicly accessible |
| **Best For** | Learning, personal utility apps, testing before deployment, development tools |
| **Cost** | $0 |
| **Tech Stacks** | Any |
| **Pros** | No infrastructure to manage, immediate feedback, full control, any database locally, maximum privacy |
| **Cons** | Not accessible outside network, dependent on computer being on, no global distribution |
| **When to Recommend** | Personal-use only apps, learning projects, development utilities, testing environments |
| **Vendor Lock-in** | NONE |
| **Workflow Note** | TERMINATION POINT - workflow complete after project-spinup |

---

### Hostinger Shared Hosting

| Attribute | Value |
|-----------|-------|
| **Description** | Traditional web hosting with cPanel, PHP/MySQL stack |
| **Best For** | Simple PHP sites, WordPress, static websites |
| **Cost** | $0 marginal if account exists, otherwise $3-5/month |
| **Tech Stacks** | PHP + MySQL only |
| **Pros** | $0 marginal cost, managed infrastructure, cPanel interface, minimal maintenance |
| **Cons** | Limited to PHP + MySQL, no Docker, shared resources, single region |
| **When to Recommend** | Simple PHP apps, WordPress, existing shared hosting account |
| **Vendor Lock-in** | LOW - standard PHP |

---

### Cloudflare Pages

| Attribute | Value |
|-----------|-------|
| **Description** | Serverless platform for static sites and JAMstack on global edge network |
| **Best For** | Static sites, SPAs, JAMstack, frontend-only projects |
| **Cost** | $0 (genuinely free, unlimited bandwidth/requests/builds) |
| **Tech Stacks** | Any static generator, React, Vue, Next.js (static), Svelte, Astro |
| **Pros** | Zero-config deployment, automatic HTTPS, global CDN, git-connected auto-deploy |
| **Cons** | Static/frontend only, no persistent backend, 100MB project limit |
| **When to Recommend** | Frontend-only app, need global performance, want zero cost, already using Cloudflare DNS |
| **Vendor Lock-in** | LOW - vanilla HTML/CSS/JS, portable |

---

### Fly.io

| Attribute | Value |
|-----------|-------|
| **Description** | Platform for containerized applications globally across 35 regions with managed PostgreSQL |
| **Best For** | Full-stack with database, global distribution, managed infrastructure |
| **Cost** | $20-50/month for multiple small projects |
| **Tech Stacks** | Any Docker-based |
| **Pros** | Docker-based, 35 global regions, auto-scaling, managed PostgreSQL, CLI-first |
| **Cons** | Requires Docker knowledge, pay-per-use pricing, data egress charges |
| **When to Recommend** | Full-stack needing database, global distribution, comfortable with Docker, $20-50/month acceptable |
| **Vendor Lock-in** | LOW - standard Docker containers portable anywhere |

---

### VPS with Docker

| Attribute | Value |
|-----------|-------|
| **Description** | Virtual Private Server running Docker containers for complete control |
| **Best For** | Full-stack with custom requirements, self-hosted infrastructure, learning DevOps |
| **Cost** | $40-60/month VPS (marginal cost $0 if already have) |
| **Tech Stacks** | Any |
| **Pros** | Full control, use existing infrastructure, predictable costs, maximum learning, run any database |
| **Cons** | More maintenance, manage security updates, single point of failure, slower deployment |
| **When to Recommend** | Already have VPS with capacity, tech stack runs in Docker, traffic <10k daily, learning is goal, need maximum control |
| **Vendor Lock-in** | NONE - standard Docker containers, move anywhere |

---

## Decision Logic

### Recommend Localhost When

- Personal utility app for own use only
- Learning project with no need for public access
- Development tools and scripts
- Testing before public deployment
- Privacy-sensitive personal data

### Recommend Shared Hosting When

- Simple PHP-based application
- Traditional LAMP stack preferred
- Minimal maintenance desired
- Want to use existing shared hosting ($0 marginal)

### Recommend Cloudflare Pages When

- Frontend-only application
- Static or JAMstack architecture
- Want zero cost
- Need global CDN performance

### Recommend Fly.io When

- Full-stack app with database
- Need global distribution
- Want managed infrastructure
- Comfortable with Docker
- Budget $20-50/month acceptable

### Recommend VPS Docker When

- Tech stack runs well in Docker
- Traffic low-to-moderate (<10k daily)
- Want to learn DevOps
- Budget-conscious (marginal cost $0)
- Need maximum control
- Want to self-host tools

---

## Deployment Patterns

### Pattern: Localhost Development

```
Option: Localhost
Stack: Any
Database: Local PostgreSQL/SQLite/MySQL
Deployment: docker compose up (or native)
Cost: $0/month
Termination: Workflow complete after project-spinup
```

### Pattern: Static JAMstack

```
Option: Cloudflare Pages
Stack: React, Vue, Svelte, Astro, Next.js (static)
Database: External API or none
Deployment: Git push (auto-deploy)
Cost: $0/month
```

### Pattern: Full-Stack Self-Hosted

```
Option: VPS with Docker
Stack: Next.js, FastAPI, PHP, Node.js
Database: Self-hosted PostgreSQL/MySQL
Deployment: Git push -> SSH -> docker compose up
Cost: $0/month marginal
```

### Pattern: Full-Stack Global

```
Option: Fly.io
Stack: Next.js, FastAPI, Node.js, Go
Database: Fly.io managed PostgreSQL
Deployment: fly deploy
Cost: $20-50/month
```

### Pattern: Hybrid Frontend/Backend

```
Frontend: Cloudflare Pages
Backend API: VPS with Docker
Database: Self-hosted PostgreSQL
Cost: $0/month marginal
Best for: Fast frontend, controlled backend
```

### Pattern: Simple PHP

```
Option: Hostinger Shared Hosting
Stack: PHP + MySQL
Deployment: FTP or Git
Cost: $3-5/month
```

---

## Scoring Criteria

When evaluating options, rate each 1-5:

| Criterion | What to Evaluate |
|-----------|------------------|
| **Tech Stack Compatibility** | Can handle chosen stack? Required services available? |
| **Performance Requirements** | Handle expected traffic? Acceptable latency? |
| **Cost Efficiency** | Fits budget? Predictable costs? Cost-effective as grows? |
| **Deployment Ease** | Complexity? Can user manage it? Automated or manual? |
| **Maintenance Burden** | Weekly maintenance time? Monitoring complexity? |
| **Learning Value** | Teaches useful skills? Industry-relevant? |

---

## Output Templates

### Standard Recommendation Template

```markdown
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
```

### Localhost-Specific Template

```markdown
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
docker compose up -d  # or native start commands

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
```

---

*Reference document for deployment-advisor skill*
*Created: 2025-12-09*
