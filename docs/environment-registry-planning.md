# Environment Registry Planning Document

**Created:** 2025-12-05
**Purpose:** Design a centralized environment configuration system for the Skills workflow
**Status:** Planning - Awaiting Review
**Related:** [workflow-refinement.md](workflow-refinement.md)

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Goals](#goals)
3. [Proposed Solution](#proposed-solution)
4. [Schema Design](#schema-design)
5. [Integration with Skills](#integration-with-skills)
6. [Alignment with Anthropic's Guidance](#alignment-with-anthropics-guidance)
7. [Open Questions](#open-questions)
8. [Implementation Considerations](#implementation-considerations)
9. [Next Steps](#next-steps)

---

## Problem Statement

### Current Pain Points

The existing skills workflow has infrastructure knowledge scattered throughout:
- User context embedded directly in skill definitions (e.g., VPS specs in deployment-advisor)
- Repetitive questions about established infrastructure ("Do you have SSH access?")
- No single source of truth for available deployment targets
- Risk of skills recommending options outside the user's actual environment
- Credentials and endpoints re-explained in every session

### What Gets Asked Repeatedly

| Question Type | Example | Should Be Known |
|--------------|---------|-----------------|
| Access | "Do you have SSH access to your VPS?" | Yes - always true |
| Infrastructure | "What proxy server are you using?" | Yes - always Caddy |
| Capabilities | "Can you create Backblaze buckets?" | Yes - on demand |
| Preferences | "Should we compare costs?" | Yes - user manages separately |
| Services | "Where is your Pocketbase instance?" | Yes - documented location |

### The Cost of Repetition

- **Context waste**: Questions and answers consume tokens
- **Session friction**: Slows down the workflow
- **Error risk**: Inconsistent answers across sessions
- **Decision drift**: Skills might assume different infrastructure states

---

## Goals

### Primary Goals

1. **Eliminate redundant questions** about established infrastructure
2. **Constrain recommendations** to actual available options
3. **Provide accurate configuration** so skills don't guess
4. **Support evolution** as infrastructure changes over time
5. **Maintain security** by separating sensitive from non-sensitive data

### Non-Goals

- Storing highly sensitive credentials (use `.env` or secrets managers)
- Replacing project-specific configuration
- Automating infrastructure provisioning
- Cost tracking or billing integration

---

## Proposed Solution

### The Environment Registry

A structured JSON file that serves as the authoritative source for:
- Available infrastructure (VPS, services, accounts)
- Deployment options (what's allowed, what's preferred)
- Storage options (Backblaze, Tigris, etc.)
- Established choices (proxy, hardening status, etc.)
- Skill guidance (what to skip, what to always offer)

### Why JSON

| Reason | Benefit |
|--------|---------|
| Machine-readable | Skills can parse and query specific fields |
| Schema-validatable | Can ensure required fields exist |
| Diffable | Git tracks changes clearly |
| Editable | Direct editing in any text editor |
| Extensible | Add new sections without restructuring |
| Aligned with workflow | Matches the JSON-for-data pattern in workflow-refinement.md |

### File Location Options

**Option A: User-level (recommended)**
```
~/.claude/environment.json
```
- Shared across all projects
- Represents YOUR infrastructure, not project-specific
- Single place to update when infrastructure changes

**Option B: Project-level**
```
.claude/environment.json
```
- Per-project customization possible
- Could override user-level defaults
- More files to maintain

**Recommendation:** User-level as primary, with optional project-level overrides for special cases.

---

## Schema Design

### Complete Proposed Schema

```json
{
  "$schema": "./environment-schema.json",
  "version": "1.0",
  "last_updated": "2025-12-05",
  "owner": "john",

  "infrastructure": {
    "vps": {
      "vps8": {
        "provider": "Hostinger",
        "role": "homelab",
        "hostname": "vps8.yourdomain.com",
        "ip": "xxx.xxx.xxx.xxx",
        "specs": {
          "ram": "16GB",
          "cpu": "4 vCPU",
          "storage": "200GB",
          "os": "Ubuntu 24.04"
        },
        "proxy": "caddy",
        "ssh_access": true,
        "hardened": true,
        "docker_installed": true,
        "notes": "Primary homelab server"
      },
      "vps1": {
        "provider": "Hostinger",
        "role": "application_hosting",
        "hostname": "vps1.yourdomain.com",
        "ip": "xxx.xxx.xxx.xxx",
        "specs": {
          "ram": "4GB",
          "cpu": "2 vCPU",
          "storage": "50GB",
          "os": "Ubuntu 24.04"
        },
        "proxy": "caddy",
        "ssh_access": true,
        "hardened": true,
        "docker_installed": true,
        "notes": "Smaller VPS for container deployments"
      },
      "vps2": {
        "provider": "Hostinger",
        "role": "application_hosting",
        "hostname": "vps2.yourdomain.com",
        "ip": "xxx.xxx.xxx.xxx",
        "specs": {
          "ram": "4GB",
          "cpu": "2 vCPU",
          "storage": "50GB",
          "os": "Ubuntu 24.04"
        },
        "proxy": "caddy",
        "ssh_access": true,
        "hardened": true,
        "docker_installed": true,
        "notes": "Smaller VPS for container deployments"
      }
    },

    "services": {
      "supabase_selfhosted": {
        "location": "vps8",
        "type": "supabase",
        "dashboard_url": "https://supabase.yourdomain.com",
        "api_url": "https://supabase-api.yourdomain.com",
        "status": "running",
        "notes": "Self-hosted Supabase instance"
      },
      "pocketbase_homelab": {
        "location": "vps8",
        "type": "pocketbase",
        "url": "https://pb.yourdomain.com",
        "admin_url": "https://pb.yourdomain.com/_/",
        "status": "running",
        "purpose": "shared backend for multiple projects",
        "notes": "Primary Pocketbase instance"
      },
      "n8n": {
        "location": "vps8",
        "type": "n8n",
        "url": "https://n8n.yourdomain.com",
        "status": "running",
        "notes": "Workflow automation"
      },
      "wikijs": {
        "location": "vps8",
        "type": "wiki",
        "url": "https://wiki.yourdomain.com",
        "status": "running",
        "notes": "Documentation wiki"
      },
      "ollama": {
        "location": "vps8",
        "type": "llm",
        "api_endpoint": "http://localhost:11434",
        "external_endpoint": "https://ollama.yourdomain.com",
        "models_available": ["llama3.2", "codellama"],
        "status": "running",
        "notes": "Local LLM inference"
      }
    }
  },

  "deployment_options": {
    "allowed": [
      {
        "id": "vps_container_vps1",
        "name": "VPS Container (vps1)",
        "type": "docker_container",
        "target": "vps1",
        "suitable_for": ["web_apps", "apis", "background_services"]
      },
      {
        "id": "vps_container_vps2",
        "name": "VPS Container (vps2)",
        "type": "docker_container",
        "target": "vps2",
        "suitable_for": ["web_apps", "apis", "background_services"]
      },
      {
        "id": "fly_io",
        "name": "Fly.io",
        "type": "paas",
        "suitable_for": ["web_apps", "apis", "globally_distributed"],
        "notes": "Good for apps needing edge deployment"
      },
      {
        "id": "cloudflare_pages",
        "name": "Cloudflare Pages",
        "type": "static_jamstack",
        "suitable_for": ["static_sites", "jamstack", "spa"],
        "notes": "Static sites and Jamstack only"
      },
      {
        "id": "hostinger_shared",
        "name": "Hostinger Shared Hosting",
        "type": "shared_hosting",
        "suitable_for": ["php", "wordpress", "simple_static"],
        "notes": "Traditional shared hosting"
      },
      {
        "id": "localhost",
        "name": "Localhost",
        "type": "local",
        "suitable_for": ["development", "learning", "limited_production"],
        "notes": "Valid for dev AND limited production for small-scale, short-term projects"
      }
    ],
    "not_using": [
      "AWS",
      "GCP",
      "Azure",
      "Vercel",
      "Netlify",
      "Heroku",
      "DigitalOcean"
    ],
    "selection_notes": "Deployment target should match project requirements. Don't suggest options outside this list."
  },

  "storage_options": {
    "backblaze_b2": {
      "available": true,
      "account_exists": true,
      "bucket_creation": "on_demand",
      "key_provisioning": "per_project",
      "compatible_with": ["vps_container_vps1", "vps_container_vps2", "fly_io", "localhost"],
      "notes": "User will provide bucket-specific keys for each project"
    },
    "tigris": {
      "available": true,
      "preferred_with": ["fly_io"],
      "notes": "Consider when deploying to Fly.io"
    },
    "local_storage": {
      "available": true,
      "use_cases": ["development", "small_projects"],
      "notes": "Docker volumes on VPS or local filesystem"
    }
  },

  "external_accounts": {
    "fly_io": {
      "account_exists": true,
      "can_create_apps": true,
      "can_create_postgres": true,
      "can_create_pocketbase": true,
      "notes": "Full Fly.io access"
    },
    "cloudflare": {
      "account_exists": true,
      "services": ["pages", "workers", "dns"],
      "notes": "Domain DNS managed here"
    },
    "github": {
      "account_exists": true,
      "can_create_repos": true,
      "actions_available": true
    },
    "backblaze": {
      "account_exists": true,
      "services": ["b2_storage"],
      "notes": "See storage_options.backblaze_b2"
    }
  },

  "database_options": {
    "pocketbase": {
      "instances": {
        "homelab": {
          "reference": "infrastructure.services.pocketbase_homelab",
          "use_when": "shared backend, non-isolated projects"
        },
        "fly_io": {
          "type": "on_demand",
          "use_when": "project needs isolation or fly.io deployment"
        }
      },
      "default": "homelab",
      "notes": "Confirm which instance for each project"
    },
    "supabase": {
      "instances": {
        "selfhosted": {
          "reference": "infrastructure.services.supabase_selfhosted",
          "use_when": "full Supabase features needed"
        }
      }
    },
    "sqlite": {
      "available": true,
      "use_when": "simple persistence, single-user apps, development"
    },
    "postgres": {
      "options": ["supabase_selfhosted", "fly_io_managed"],
      "notes": "Via Supabase or Fly.io managed postgres"
    }
  },

  "established_choices": {
    "proxy_server": {
      "choice": "caddy",
      "locked": true,
      "reason": "Installed on all VPS, handles SSL automatically"
    },
    "ssh_security": {
      "choice": "hardened",
      "locked": true,
      "details": ["key_based_auth", "password_disabled", "fail2ban"]
    },
    "containerization": {
      "choice": "docker",
      "locked": true,
      "reason": "Docker installed on all VPS"
    },
    "version_control": {
      "choice": "git",
      "remote": "github",
      "locked": true
    },
    "cost_assessment": {
      "choice": "user_managed",
      "locked": true,
      "reason": "User maintains separate cost assessments"
    }
  },

  "skill_guidance": {
    "skip_questions": [
      "ssh_access_available",
      "proxy_server_choice",
      "cost_comparison_needed",
      "storage_cost_analysis",
      "can_create_buckets",
      "docker_available",
      "git_available",
      "github_access"
    ],
    "always_offer": [
      "backblaze_b2_storage",
      "tigris_with_fly_io"
    ],
    "always_ask": [
      "pocketbase_instance_choice"
    ],
    "never_suggest": [
      "nginx_instead_of_caddy",
      "deployment_outside_allowed_list",
      "cost_optimization_alternatives"
    ],
    "preferences": {
      "explanations": "concise",
      "alternatives_shown": 2,
      "learning_mode_default": true
    }
  },

  "credentials": {
    "_note": "Only non-sensitive identifiers here. Actual secrets go in .env files.",
    "domains": {
      "primary": "yourdomain.com",
      "managed_by": "cloudflare"
    },
    "api_endpoints": {
      "pocketbase_homelab": "https://pb.yourdomain.com",
      "supabase_api": "https://supabase-api.yourdomain.com",
      "ollama": "https://ollama.yourdomain.com"
    }
  }
}
```

### Schema Sections Explained

| Section | Purpose |
|---------|---------|
| `infrastructure.vps` | Physical/virtual servers you control |
| `infrastructure.services` | Running services on those servers |
| `deployment_options` | Where projects CAN be deployed |
| `storage_options` | Object storage choices |
| `external_accounts` | Third-party service accounts |
| `database_options` | Database choices with instance disambiguation |
| `established_choices` | Decisions that are locked/unchanging |
| `skill_guidance` | Direct instructions for skill behavior |
| `credentials` | Non-sensitive identifiers and endpoints |

---

## Integration with Skills

### How Skills Would Use the Registry

Each skill would include an environment loading phase:

```xml
<phase id="0" name="load-environment">
  <description>Load and apply environment constraints</description>

  <actions>
    1. Read ~/.claude/environment.json (or .claude/environment.json if exists)
    2. Parse deployment_options.allowed - ONLY recommend from this list
    3. Check skill_guidance.skip_questions - DO NOT ask these
    4. Check skill_guidance.always_ask - MUST ask these
    5. Check skill_guidance.never_suggest - AVOID these recommendations
    6. Load established_choices - treat as immutable facts
  </actions>

  <validation>
    If environment.json not found:
    - Warn user: "No environment registry found. I'll ask more questions."
    - Proceed with standard questioning
  </validation>
</phase>
```

### Example: deployment-advisor Using the Registry

**Before (current behavior):**
```
"What VPS do you have available?"
"Do you have SSH access?"
"What proxy server are you using?"
"Should we compare costs between options?"
"Do you have a Backblaze account?"
```

**After (with registry):**
```
Based on your environment, here are your deployment options for this project:

1. **VPS Container on vps1** - Good fit for [reasons]
2. **Fly.io** - Good fit for [reasons]
3. **Localhost** - Suitable for development/learning

Which direction interests you?
```

### Per-Skill Integration Points

| Skill | Uses From Registry |
|-------|-------------------|
| **project-brief-writer** | Nothing (stays pure problem/solution) |
| **tech-stack-advisor** | `database_options`, `skill_guidance.preferences` |
| **deployment-advisor** | `deployment_options`, `infrastructure`, `storage_options` |
| **project-spinup** | `infrastructure.services`, `credentials.api_endpoints` |
| **test-orchestrator** | `established_choices.containerization` |
| **ci-cd-implement** | `external_accounts.github`, `deployment_options` |
| **deploy-guide** | `infrastructure.vps`, `credentials`, `established_choices` |

---

## Alignment with Anthropic's Guidance

### From "Effective Harnesses for Long-Running Agents"

Anthropic recommends:
- **External memory artifacts** for state that persists across sessions
- **JSON over markdown** for machine-managed state
- **Reducing context waste** by not re-explaining known information

The Environment Registry directly implements these principles:
- It's an external memory artifact
- It's JSON-formatted
- It eliminates repetitive Q&A

### From Workflow Refinement Goals

This addresses several identified problems:
- **Problem: User context embedded in skills** → Solution: Externalized to registry
- **Question 4: User Context Handling** → Answer: Option B (separate file), expanded scope
- **Goal: Neutral handoffs** → Registry provides facts, not biases

---

## Open Questions

### Question 1: File Location

**Where should the environment registry live?**

| Option | Location | Pros | Cons |
|--------|----------|------|------|
| A | `~/.claude/environment.json` | Single source, shared across projects | Need to ensure Claude Code can access |
| B | `.claude/environment.json` per project | Project-specific overrides | Duplication, maintenance burden |
| C | Both (user-level + project overrides) | Flexibility | Complexity, merge logic needed |

**My recommendation:** Start with Option A (user-level). Add project-level overrides only if a clear need emerges. The registry represents YOUR infrastructure, which doesn't change per-project.

---

### Question 2: Sensitive Data Handling

**How should we handle semi-sensitive information?**

| Data Type | Example | Recommendation |
|-----------|---------|----------------|
| Public URLs | `https://pb.yourdomain.com` | In registry |
| IP addresses | `123.45.67.89` | In registry (not truly secret) |
| API endpoints | Supabase API URL | In registry |
| API keys | `sb_xxxxx` | NOT in registry → `.env` |
| Passwords | Admin passwords | NOT in registry → `.env` or secrets manager |

**Options:**
- **A:** Everything non-secret in main registry
- **B:** Split into `environment.json` (public) + `environment.local.json` (gitignored, semi-sensitive)
- **C:** Registry contains references only; actual values always in `.env`

**My recommendation:** Option A for now. IP addresses and URLs aren't truly secrets. Actual credentials (API keys, passwords) should always be in `.env` files per-project. The registry can note "API key required" without containing the key.

---

### Question 3: Pocketbase Instance Disambiguation

**How should skills know which Pocketbase instance to use?**

Your homelab Pocketbase vs. a fresh Fly.io Pocketbase are different use cases:

| Scenario | Which Instance |
|----------|---------------|
| Shared backend, multiple projects | Homelab |
| Isolated project, needs own data | Fly.io (new) |
| Deploying to Fly.io anyway | Could be either |

**Options:**
- **A:** Always ask (one question is worth it for clarity)
- **B:** Default to homelab, only ask if deploying to Fly.io
- **C:** Specify in project brief ("this project uses isolated Pocketbase")
- **D:** Add to DECISIONS.json during tech-stack-advisor phase

**My recommendation:** Option A (always ask) during deployment-advisor phase. This is a meaningful architectural decision that affects data isolation, and the cost of asking once is low compared to the cost of getting it wrong.

---

### Question 4: Update Mechanism

**How should the registry be updated when infrastructure changes?**

| Option | Method | Complexity | User Experience |
|--------|--------|------------|-----------------|
| A | Direct JSON editing | Low | Technical but full control |
| B | CLI helper commands | Medium | `claude env add-vps vps3 --ip x.x.x.x` |
| C | Interactive skill | Medium | Conversational updates |
| D | Web UI | High | Most user-friendly |

**My recommendation:** Start with Option A (direct editing). The JSON structure is clear, and you're comfortable with it. If friction emerges, we can add a simple skill that walks through updates conversationally (Option C). Avoid over-engineering with CLI tools or UIs until there's a clear need.

---

### Question 5: Validation

**Should we validate the registry schema?**

| Option | Approach | Benefit |
|--------|----------|---------|
| A | No validation | Simple, flexible |
| B | JSON Schema file | Catches typos, ensures required fields |
| C | Runtime validation in skills | Skills verify what they need |

**My recommendation:** Option B (JSON Schema). Create a simple `environment-schema.json` that validates structure. This catches errors early without adding runtime complexity. The schema can be loose initially and tighten as patterns stabilize.

---

### Question 6: Versioning

**How should we handle registry schema evolution?**

As your infrastructure grows, the schema may need new sections.

| Option | Approach |
|--------|----------|
| A | Additive only (never break existing) |
| B | Version field with migration notes |
| C | Formal migrations |

**My recommendation:** Option A (additive only) with a version field for documentation. Skills should handle missing optional fields gracefully. This keeps things simple while allowing growth.

---

### Question 7: Registry Scope

**What else might belong in the registry?**

Potential additions to consider:

| Category | Examples | Include? |
|----------|----------|----------|
| Domain names | Primary domain, subdomains available | Yes |
| SSL/TLS | Cert management approach | Maybe (Caddy handles this) |
| Monitoring | Uptime tools, logging | Future consideration |
| Backup strategy | Where backups go | Future consideration |
| Team/collaboration | Other users with access | Not needed (solo) |

**My recommendation:** Start minimal. Add categories only when a skill actually needs the information. The current schema covers the immediate needs well.

---

### Question 8: Skill Guidance Granularity

**How specific should skill_guidance be?**

The current schema has:
- `skip_questions`: Don't ask these
- `always_offer`: Present these options
- `always_ask`: Must ask these
- `never_suggest`: Avoid these

**Should we add:**
- Per-skill guidance sections?
- Conditional logic ("if deploying to fly.io, always offer tigris")?
- Priority ordering for recommendations?

**My recommendation:** Keep it flat for now. Per-skill sections add complexity. If specific skills need special guidance, we can add it. The current structure handles 90% of cases.

---

## Implementation Considerations

### Migration Path

1. **Create the registry** with your current infrastructure
2. **Add environment loading** to one skill (deployment-advisor is ideal)
3. **Test the pattern** with a real project
4. **Roll out** to remaining skills
5. **Iterate** based on friction points

### Graceful Degradation

Skills should work without the registry:
```xml
<if condition="environment.json not found">
  Proceed with standard questioning.
  Note: "Consider creating ~/.claude/environment.json for streamlined sessions."
</if>
```

### Registry Maintenance

- **When to update:** After any infrastructure change (new VPS, new service, account changes)
- **How to update:** Edit JSON directly, commit to dotfiles repo
- **Validation:** Optional JSON Schema check before committing

---

## Next Steps

### Immediate Actions

1. **Review this document** - Note any disagreements or additions
2. **Answer open questions** - Your preferences will shape the implementation
3. **Draft your actual registry** - Fill in real values (IPs, URLs, etc.)

### When Ready to Implement

1. Create `~/.claude/environment.json` with your infrastructure
2. Create `environment-schema.json` for validation (optional)
3. Modify one skill to load and use the registry
4. Test with a real project
5. Roll out to remaining skills

### Integration with Workflow Refinement

This document complements [workflow-refinement.md](workflow-refinement.md):
- Registry handles **environment/infrastructure** state
- DECISIONS.json handles **project-specific** decisions
- Together they eliminate ambiguity and reduce questioning

---

## Appendix: Quick Reference

### Files in the System

| File | Location | Purpose |
|------|----------|---------|
| `environment.json` | `~/.claude/` | Your infrastructure registry |
| `environment-schema.json` | `~/.claude/` | Optional validation schema |
| `DECISIONS.json` | `.docs/` (per project) | Project-specific decisions |
| `handoff-*.json` | `.docs/` (per project) | Skill-to-skill handoffs |

### Key Principles

1. **JSON for machine state** - Skills read structured data, not prose
2. **Single source of truth** - One place for infrastructure facts
3. **Graceful degradation** - Works without registry, better with it
4. **Minimal questioning** - Ask only what's not already known
5. **Evolution-friendly** - Easy to add, hard to break

---

*Document generated by Claude Code session on 2025-12-05*
