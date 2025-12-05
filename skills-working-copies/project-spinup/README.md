# project-spinup Skill

## Overview

Generate complete project foundations with personalized claude.md, Docker setup, and project structure. Supports Guided Setup (step-by-step learning) or Quick Start (full scaffolding).

**Use when:** After tech-stack-advisor and deployment-advisor have completed, to initialize new projects with your workflow, infrastructure, and best practices.

**Output:** Complete project foundation including claude.md, docker-compose.yml, directory structure, configuration files, and either guided learning prompts or full scaffolding.

---

## How It Works

When invoked, this skill will:

1. **Load context from handoff documents** - PROJECT-MODE.md, tech-stack-decision.md, deployment-strategy.md
2. **Ask about spinup approach** - Guided Setup vs Quick Start with MODE-informed suggestion
3. **Gather any missing inputs** - Project name, description, special features
4. **Generate core files** - claude.md, Docker, directory structure, configs
5. **Apply spinup approach** - Step-by-step prompts or complete scaffolding
6. **Provide next steps** - Summary and git initialization commands

---

## Skills Workflow Integration

This skill is **Phase 3 of 3** (final phase) in the learning-focused project workflow:

```
project-brief-writer (Phase 0)
    ↓
tech-stack-advisor (Phase 1)
    ↓
deployment-advisor (Phase 2)
    ↓
project-spinup ← YOU ARE HERE (Final)
```

After completion: Skills workflow complete, ready for development.

---

## Prerequisites

Before invoking, ensure these exist:
1. **PROJECT-MODE.md** (from project-brief-writer)
2. **tech-stack-decision.md** (from tech-stack-advisor)
3. **deployment-strategy.md** (from deployment-advisor)

If documents don't exist, the skill will ask for information interactively.

---

## Spinup Approach vs PROJECT-MODE

**Important distinction:**

- **PROJECT-MODE** (LEARNING/BALANCED/DELIVERY): Governs advisory skills - exploration, discussion, checkpoints. This is **strategic learning** (understanding trade-offs).

- **spinup_approach** (Guided/Quick Start): Governs scaffolding style - step-by-step or all-at-once. This is **tactical implementation** (familiarity with tech stack).

**Key insight:** You can be in LEARNING mode but use Quick Start if you're already familiar with the tech stack. Or be in DELIVERY mode but use Guided Setup for a new framework.

---

## Guided Setup vs Quick Start

### Guided Setup
Foundation + step-by-step prompts:
- Learn how each layer works
- Verification checkpoints
- Learning objectives for each step
- Recommended if new to the tech stack

### Quick Start
Complete scaffolding immediately:
- All configuration files
- Starter/example code with comments
- Sample tests
- Recommended if familiar with the tech stack

---

## What Gets Generated

### Always Created (Both Approaches)
- claude.md - Comprehensive project context for AI assistant
- docker-compose.yml - Local development environment
- Directory structure - src/, tests/, docs/
- .gitignore - Tech-stack-appropriate
- README.md - Setup and development instructions
- .env.example - Required environment variables

### Guided Setup Adds
- Detailed "Next Steps" section with copy-paste prompts
- Learning objectives and estimated times
- Verification commands

### Quick Start Adds
- Complete source file structure
- Tech-stack-specific configurations
- Starter code with comments
- Sample tests
- All middleware/utilities

---

## Supported Tech Stacks

### Next.js + Supabase
- Next.js 15 (App Router), TypeScript, Tailwind CSS v4
- Supabase client, shadcn/ui, React Query, Zod
- Jest or Vitest

### PHP + MySQL
- PHP 8.2+, MySQL 8.0+, Composer
- Simple MVC structure, PDO, PHPUnit

### FastAPI + PostgreSQL
- FastAPI, PostgreSQL 15, SQLAlchemy
- Alembic migrations, Pydantic, pytest

---

## User Context (Hardcoded)

### Developer Profile
- Name: John
- Experience: Hobbyist, beginner-to-intermediate
- Learning Goal: Deep understanding of full-stack, professional practices
- Heavy reliance on Claude Code

### Development Environment
- Computers: MacBook Pro and Mac Mini
- IDE: VS Code
- Git: main + dev branches, conventional commits
- Directory: src/, docs/, tests/

### Infrastructure
- Hostinger VPS8 with Supabase, PocketBase, n8n, Ollama, Wiki.js, Caddy
- Backblaze B2, Cloudflare DNS
- Deployment: Localhost, Shared Hosting, Cloudflare Pages, Fly.io, VPS Docker

---

## Related Skills

- **project-brief-writer** - Creates PROJECT-MODE.md (Phase 0)
- **tech-stack-advisor** - Provides technology decisions (Phase 1)
- **deployment-advisor** - Provides deployment strategy (Phase 2)

---

## Version History

### v1.3 (2025-11-17)
- Updated infrastructure list with VPS8 specs, Caddy, PocketBase
- Expanded deployment options reference

### v1.2 (2025-11-12)
- Added support for reading handoff documents
- Integrated complete context loading

### v1.1 (2025-11-12)
- Added workflow state visibility
- Separated spinup approach from PROJECT-MODE
- Implemented MODE-informed suggestions

### v1.0 (Initial)
Initial release with claude.md generation and dual-mode scaffolding.

---

**Version:** 1.3
**Last Updated:** 2025-11-17
