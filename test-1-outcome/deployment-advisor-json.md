---
name: deployment-advisor-json
description: "[TEST] JSON-consuming version of deployment-advisor. Recommend hosting strategy based on tech-stack-decision.json."
allowed-tools:
  - Read
  - Grep
  - Glob
  - WebSearch
  - Write
---

# deployment-advisor (JSON Input Test)

<purpose>
Recommend hosting and deployment strategies tailored to chosen tech stack, project requirements, and infrastructure constraints. Consumes structured JSON input from tech-stack-advisor for reliable data extraction.
</purpose>

<role>
CONSULTANT role, not BUILDER role. Provides hosting recommendations and strategies only.
</role>

<output>
.docs/deployment-strategy.md (human-readable) AND .docs/deployment-strategy.json (machine-readable handoff)
</output>

---

<workflow>

<phase id="0" name="load-json-input">
<action>Load and parse tech-stack-decision.json as primary input.</action>

<input-file>.docs/tech-stack-decision.json</input-file>

<json-schema>
The input JSON contains:
- meta: document metadata (version, date, status)
- project: name, mode, brief reference
- stack: structured tech choices (frontend, backend, database, auth, storage, embeddings)
- services: array of services with name, type, port, memory requirements
- schema: database tables and indexes
- decisions: key technology decisions with rationale
- alternatives_ruled_out: rejected options with reasons
- learning_outcomes: educational goals
- resource_estimate: memory/compute requirements
- deployment_hints: existing infrastructure and questions for this phase
</json-schema>

<extraction>
From JSON, directly extract:
- stack.frontend.framework → frontend framework
- stack.database.engine → database system
- stack.database.extensions → special requirements (e.g., pgvector)
- services[] → services to deploy with resource needs
- deployment_hints.existing_infrastructure → available infra
- deployment_hints.services_to_deploy → what needs deployment
- deployment_hints.questions_for_deployment → specific questions to address
- project.mode → LEARNING/BALANCED/DELIVERY
- resource_estimate.total_memory_mb → resource requirements
</extraction>

<when-json-found>
Parse JSON and confirm:
"Loaded tech stack decision for {project.name}:
- Stack: {stack.frontend.framework} + {stack.database.engine}
- Services: {services[].name joined}
- Mode: {project.mode}

Proceeding to deployment analysis."
</when-json-found>

<when-json-missing>
Fall back to markdown parsing or conversational gathering:
"No tech-stack-decision.json found. Checking for .md file or gathering requirements conversationally."
</when-json-missing>
</phase>

<phase id="1" name="analyze-from-json">
<action>Use structured data to drive recommendation logic.</action>

<direct-mappings>
| JSON Field | Deployment Decision |
|------------|---------------------|
| services[].memory_mb | Resource allocation per service |
| services[].port | Port mapping in docker-compose |
| deployment_hints.existing_infrastructure | What to reuse vs deploy new |
| stack.database.provider | Managed vs self-hosted database |
| stack.auth.provider | Auth service requirements |
| resource_estimate.total_memory_mb | VPS sizing |
</direct-mappings>

<automated-decisions>
Based on JSON data, automatically determine:
1. VPS feasibility: sum(services[].memory_mb) < available_vps_ram
2. Database approach: deployment_hints.existing_infrastructure contains "supabase" → reuse
3. Service count: len(services[]) → docker-compose complexity
4. Special requirements: stack.database.extensions contains "pgvector" → needs PostgreSQL 15+
</automated-decisions>
</phase>

<phase id="2" name="generate-recommendation">
<action>Generate hosting recommendation using JSON-derived data.</action>

<template-variables>
All template fields populated directly from JSON:
- {project.name} → project name
- {stack.frontend.framework} → frontend
- {stack.database.engine} → database
- {services} → docker-compose services block
- {deployment_hints.existing_infrastructure} → infra to leverage
</template-variables>
</phase>

<phase id="3" name="create-outputs">
<action>Create both markdown (human) and JSON (machine) output.</action>

<markdown-output>
.docs/deployment-strategy.md - Human-readable deployment plan
</markdown-output>

<json-output>
.docs/deployment-strategy.json - Structured handoff for deploy-guide and ci-cd-implement
</json-output>

<json-output-schema>
{
  "meta": {
    "document": "deployment-strategy",
    "version": "1.0",
    "date": "{date}",
    "status": "approved",
    "input_ref": ".docs/tech-stack-decision.json"
  },
  "project": {
    "name": "{from input}",
    "mode": "{from input}"
  },
  "recommendation": {
    "type": "vps-docker | fly-io | cloudflare-pages | localhost",
    "primary_host": {
      "name": "{vps name or service}",
      "ip": "{if applicable}",
      "specs": { "cores": N, "ram_gb": N, "storage_gb": N }
    },
    "secondary_hosts": []
  },
  "services": [
    {
      "name": "{service name}",
      "location": "{host}",
      "container": true|false,
      "port": N,
      "memory_mb": N,
      "image": "{docker image if applicable}",
      "preexisting": true|false
    }
  ],
  "networking": {
    "reverse_proxy": "caddy | nginx | none",
    "ssl": "caddy-auto | letsencrypt | cloudflare",
    "domains": ["{domain}"]
  },
  "database": {
    "host": "{location}",
    "engine": "{postgresql | mysql | sqlite}",
    "connection_method": "direct | supabase-api | local",
    "port": N
  },
  "storage": {
    "provider": "{backblaze-b2 | local | s3}",
    "bucket": "{bucket name}"
  },
  "environment_variables": [
    { "name": "DATABASE_URL", "source": "{description}" },
    { "name": "SUPABASE_URL", "value": "{if known}" }
  ],
  "costs": {
    "setup_usd": N,
    "monthly_usd": N,
    "notes": "{cost assumptions}"
  },
  "pre_deployment_tasks": [
    { "task": "{description}", "location": "{where}", "command": "{if applicable}" }
  ],
  "deployment_workflow": {
    "initial": ["{step 1}", "{step 2}"],
    "regular": ["{step 1}", "{step 2}"],
    "rollback": ["{step 1}", "{step 2}"]
  },
  "next_phase": {
    "skill": "project-spinup | deploy-guide",
    "input": ".docs/deployment-strategy.json"
  }
}
</json-output-schema>
</phase>

</workflow>

---

<json-benefits>

<for-deployment-advisor>
- No markdown parsing ambiguity
- Direct field access: json.stack.frontend.framework
- Guaranteed structure: missing fields are explicit errors
- Numerical data preserved: memory_mb is a number, not "~2 GB"
</for-deployment-advisor>

<for-downstream-skills>
- deploy-guide can iterate services[] directly
- ci-cd-implement can generate docker-compose from services[]
- project-spinup can scaffold from schema.tables[]
</for-downstream-skills>

<for-validation>
- Schema validation possible before processing
- Required fields enforceable
- Version tracking for format changes
</for-validation>

</json-benefits>

---

<guardrails>

<must-do>
- Parse JSON input first, fall back to markdown/conversation if missing
- Output both .md and .json files
- Preserve all numerical data as numbers in JSON output
- Include input_ref in output JSON for traceability
</must-do>

<must-not-do>
- Lose precision converting numbers to prose ("~2 GB" instead of 2000)
- Skip JSON output even if markdown is primary user-facing doc
- Hardcode values that exist in input JSON
</must-not-do>

</guardrails>
