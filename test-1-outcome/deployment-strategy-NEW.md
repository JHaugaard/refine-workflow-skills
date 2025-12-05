# Deployment Strategy: kline-martin-photos

## Meta

| Field | Value |
|-------|-------|
| Document | deployment-strategy |
| Version | 1.0 |
| Date | 2025-12-03 |
| Status | recommended |
| Mode | BALANCED |
| Upstream | tech-stack-decision.json |

---

## PRIMARY HOSTING RECOMMENDATION: VPS Docker (Hostinger VPS8)

Deploy Next.js and the Python embedding service as Docker containers on your existing VPS8, leveraging the already-running Supabase (PostgreSQL + Auth) and Ollama instances. Caddy handles reverse proxy and automatic HTTPS.

### Why This Fits Your Project

- **Tech stack compatibility:** Next.js and FastAPI both containerize cleanly; pgvector already available in your Supabase PostgreSQL
- **Traffic match:** Small public use (<100 daily) is trivial for VPS8's 8 cores / 32GB RAM
- **Budget alignment:** $0 marginal cost - infrastructure already exists and has capacity

### Why This Fits Your Infrastructure

- **Existing services:** Supabase and Ollama are already running on VPS8 - no new database or inference setup
- **Caddy integration:** Add two reverse proxy entries for the new services
- **Docker Compose:** Extend existing compose setup or add new stack alongside

### Hosting Details

| Aspect | Value |
|--------|-------|
| Provider | Hostinger VPS8 |
| Server Type | VPS (8 cores, 32GB RAM, 400GB storage) |
| Container | Docker Compose |
| Database | Self-hosted Supabase PostgreSQL (existing) |
| File Storage | Backblaze B2 (S3-compatible) |
| CDN | Cloudflare (DNS + proxy) |
| SSL | Caddy (automatic HTTPS via Let's Encrypt) |
| Domain | kline-martin-photos.com |

---

## Architecture Overview

```
                    Internet
                        │
                        ▼
                   Cloudflare
                   (DNS + Proxy)
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                      VPS8 (Hostinger)                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                     Caddy                            │   │
│  │            (Reverse Proxy + HTTPS)                   │   │
│  └──────────┬──────────────────┬───────────────────────┘   │
│             │                  │                            │
│             ▼                  ▼                            │
│  ┌──────────────────┐  ┌──────────────────┐                │
│  │     Next.js      │  │ Embedding Service │                │
│  │    (port 3000)   │  │   (port 8001)     │                │
│  │    NEW DEPLOY    │  │   NEW DEPLOY      │                │
│  └────────┬─────────┘  └────────┬─────────┘                │
│           │                     │                           │
│           ▼                     ▼                           │
│  ┌──────────────────────────────────────────┐              │
│  │              Supabase (existing)          │              │
│  │   PostgreSQL + pgvector + Auth            │              │
│  │              (port 5432)                  │              │
│  └──────────────────────────────────────────┘              │
│                        │                                    │
│                        ▼                                    │
│  ┌──────────────────────────────────────────┐              │
│  │           Ollama (existing)               │              │
│  │         nomic-embed-text model            │              │
│  │            (port 11434)                   │              │
│  └──────────────────────────────────────────┘              │
└─────────────────────────────────────────────────────────────┘
                        │
                        ▼
              ┌──────────────────┐
              │   Backblaze B2   │
              │  (Image Storage) │
              └──────────────────┘
```

---

## Services Summary

| Service | Status | Port | Memory | Notes |
|---------|--------|------|--------|-------|
| nextjs | **TO DEPLOY** | 3000 | 500MB | App Router, SSR for share links |
| embedding-service | **TO DEPLOY** | 8001 | 2GB | FastAPI + SigLIP model |
| supabase | existing | 5432 | 4GB | PostgreSQL + pgvector + Auth |
| ollama | existing | 11434 | 1GB | nomic-embed-text for text embeddings |
| caddy | existing | 80/443 | - | Reverse proxy + HTTPS |

**Total New Memory:** ~2.5GB
**VPS8 Capacity:** 32GB RAM (plenty of headroom)

---

## Deployment Workflow

### Initial Setup (One-time)

1. **DNS Configuration (Cloudflare)**
   ```
   A    kline-martin-photos.com      → VPS8 IP
   A    api.kline-martin-photos.com  → VPS8 IP (optional, for embedding service)
   ```

2. **Create Project Directory on VPS**
   ```bash
   ssh john@vps8
   mkdir -p ~/apps/kline-martin-photos
   cd ~/apps/kline-martin-photos
   ```

3. **Create Docker Compose Stack**
   - Next.js container (from built image or multi-stage build)
   - Embedding service container (FastAPI + SigLIP)
   - Shared network with existing Supabase/Ollama

4. **Configure Caddy**
   Add to Caddyfile:
   ```
   kline-martin-photos.com {
       reverse_proxy localhost:3000
   }

   # Optional: separate subdomain for embedding API
   api.kline-martin-photos.com {
       reverse_proxy localhost:8001
   }
   ```

5. **Environment Variables**
   ```bash
   # .env on VPS
   SUPABASE_URL=http://localhost:8000  # or internal Supabase URL
   SUPABASE_ANON_KEY=your-anon-key
   SUPABASE_SERVICE_KEY=your-service-key
   B2_KEY_ID=your-b2-key
   B2_APP_KEY=your-b2-app-key
   B2_BUCKET=kline-martin-photos
   OLLAMA_URL=http://localhost:11434
   EMBEDDING_SERVICE_URL=http://localhost:8001
   ```

6. **Initialize Database Schema**
   Run migrations via Supabase CLI or direct SQL for:
   - `images` table with vector columns
   - `share_links` table
   - `profiles` table
   - RLS policies

7. **Pull Ollama Model (if not present)**
   ```bash
   ollama pull nomic-embed-text
   ```

### Regular Deployment (Updates)

**Option A: Git-based (Recommended)**
```bash
# On VPS
cd ~/apps/kline-martin-photos
git pull origin main
docker compose build
docker compose up -d
```

**Option B: Local Build + Push**
```bash
# Local machine
docker build -t kline-martin-photos-web .
docker save kline-martin-photos-web | ssh john@vps8 'docker load'
ssh john@vps8 'cd ~/apps/kline-martin-photos && docker compose up -d'
```

### Rollback Procedure

```bash
# On VPS
cd ~/apps/kline-martin-photos
git checkout HEAD~1  # or specific commit
docker compose build
docker compose up -d
```

Or with tagged images:
```bash
docker compose down
# Edit compose to use previous image tag
docker compose up -d
```

**Deployment Speed:** Moderate (2-5 minutes for build + restart)
**Maintenance Burden:** Low-Medium (Docker updates, occasional Supabase maintenance)

---

## Cost Breakdown

### Setup Costs
| Item | Cost |
|------|------|
| VPS8 (existing) | $0 |
| Domain (kline-martin-photos.com) | ~$12/year |
| Cloudflare (free tier) | $0 |
| Backblaze B2 (first 10GB free) | $0 |
| **Total Setup** | **~$12/year** |

### Monthly Ongoing
| Item | Cost |
|------|------|
| VPS8 (marginal) | $0 |
| Backblaze B2 (~10GB images) | ~$0.50 |
| Cloudflare | $0 |
| **Total Monthly** | **~$0.50/month** |

### Cost Scaling
| Traffic | Cost Impact |
|---------|-------------|
| Current (<100 daily) | $0.50/month |
| 10x (1,000 daily) | $0.50/month (same) |
| 100x (10,000 daily) | Consider CDN for images ($5-10/month) |

---

## Scaling Path

### Current State (Phase 0)
- Single VPS deployment
- All services on VPS8
- Suitable for: <1,000 daily users

### Phase 1: Optimization (When Needed)
**Trigger:** Response times >500ms or embedding queue backing up

Actions:
- Add Redis for caching (search results, session data)
- Optimize database indexes
- Pre-generate common embeddings

### Phase 2: Separation (If Growth Continues)
**Trigger:** >5,000 daily users OR need high availability

Actions:
- Move Next.js to separate VPS or Fly.io
- Consider managed PostgreSQL (Supabase Cloud)
- Add CDN for image delivery (Cloudflare R2 or keep B2 + CDN)

**Realistic Assessment:** For a personal photo collection with ~1,100 images shared with family/friends, Phase 0 will likely serve you indefinitely.

---

## Alternative Hosting Options

### Alternative 1: Fly.io (Managed Platform)

| Aspect | Details |
|--------|---------|
| **Approach** | Deploy Next.js + embedding service on Fly.io |
| **Database** | Fly Postgres or keep self-hosted Supabase |
| **Cost** | ~$25-40/month |

**Pros:**
- Zero server maintenance
- Automatic scaling
- Global edge deployment
- Easy rollbacks

**Cons:**
- Monthly cost vs $0 marginal
- Less learning (DevOps)
- Need to configure connection to VPS8 Supabase/Ollama (or migrate those too)

**When to Choose:** If VPS maintenance becomes burdensome or you need multi-region.

### Alternative 2: Hybrid (Cloudflare Pages + VPS API)

| Aspect | Details |
|--------|---------|
| **Approach** | Static Next.js export on Cloudflare Pages, API on VPS |
| **Cost** | $0/month |

**Pros:**
- Global CDN for frontend
- Reduced VPS load

**Cons:**
- Loses SSR for share links (critical feature)
- More complex architecture
- CORS configuration needed

**When to Choose:** Not recommended for this project due to SSR requirement.

---

## Monitoring & Maintenance

### Daily
- None required (set up alerts for automation)

### Weekly
- Check VPS resource usage: `docker stats`
- Review Caddy access logs for anomalies
- Verify backups completed

### Monthly
- Apply security updates: `apt update && apt upgrade`
- Review and rotate logs
- Check Backblaze B2 storage usage
- Update Docker images for security patches

### Recommended Monitoring Stack
- **Uptime:** UptimeRobot (free tier) or similar
- **Metrics:** Docker stats + Caddy logs (start simple)
- **Alerts:** Email notifications for downtime

---

## Backup Strategy

### Database Backups
| Aspect | Value |
|--------|-------|
| Frequency | Daily |
| Method | Supabase built-in or `pg_dump` cron |
| Storage | Backblaze B2 (separate bucket) |
| Retention | 30 days |

```bash
# Example cron (add to VPS)
0 3 * * * pg_dump -h localhost -U postgres kline_martin | gzip > /backups/db-$(date +\%Y\%m\%d).sql.gz
```

### File Storage Backups
| Aspect | Value |
|--------|-------|
| Provider | Backblaze B2 |
| Redundancy | B2 has built-in redundancy |
| Additional | Consider B2 versioning for accidental deletes |

### Configuration Backups
- Docker Compose files in git repository
- Environment variables documented (secrets in password manager)
- Caddyfile in git or backed up

### Disaster Recovery
| Scenario | Recovery Time |
|----------|---------------|
| Container crash | ~1 minute (auto-restart) |
| VPS reboot | ~5 minutes |
| Full VPS rebuild | ~2 hours (restore from backups) |

---

## Security Considerations

### Implemented (via existing infrastructure)
- HTTPS via Caddy (automatic Let's Encrypt)
- Cloudflare proxy (DDoS protection, IP hiding)
- Supabase RLS (row-level security)
- Magic link auth (no passwords to steal)

### Recommended Additions
- [ ] Firewall rules: Only expose 80/443, block direct access to app ports
- [ ] Fail2ban for SSH protection
- [ ] Regular security updates (automated via unattended-upgrades)
- [ ] Rate limiting on embedding service API
- [ ] Validate/sanitize image uploads
- [ ] Secure environment variable management

### Backblaze B2 Security
- Use application keys with minimal permissions
- Enable bucket encryption
- Private bucket (signed URLs for access)

---

## Environment Configuration Reference

### Next.js (.env.local)
```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-supabase-instance
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-key

# Backblaze B2
B2_KEY_ID=your-key-id
B2_APP_KEY=your-app-key
B2_BUCKET_NAME=kline-martin-photos
B2_ENDPOINT=https://s3.us-west-004.backblazeb2.com

# Embedding Service
EMBEDDING_SERVICE_URL=http://embedding-service:8001

# Ollama (for text embeddings)
OLLAMA_URL=http://ollama:11434
```

### Embedding Service (.env)
```bash
# Model configuration
SIGLIP_MODEL=ViT-B-16-SigLIP
EMBEDDING_DIM=512

# Service
HOST=0.0.0.0
PORT=8001
```

---

## Docker Compose Reference

```yaml
# docker-compose.yml (new services only)
version: '3.8'

services:
  nextjs:
    build:
      context: ./web
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    env_file:
      - .env
    restart: unless-stopped
    networks:
      - app-network

  embedding-service:
    build:
      context: ./embedding-service
      dockerfile: Dockerfile
    ports:
      - "8001:8001"
    environment:
      - HOST=0.0.0.0
      - PORT=8001
    volumes:
      - model-cache:/root/.cache
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 2G
    networks:
      - app-network

volumes:
  model-cache:

networks:
  app-network:
    external: true  # Connect to existing network with Supabase/Ollama
```

---

## Next Steps

**Handoff document created:** `.docs/deployment-strategy.md`

1. Review this strategy and ask any questions
2. If agreed, invoke **project-spinup** skill to generate project foundation
3. Build your features (Next.js app, embedding service, database schema)
4. When ready to deploy, use **deploy-guide** skill for step-by-step deployment
5. Optional: Use **ci-cd-implement** skill for automated deployments

---

## BALANCED Mode Checkpoint

Before proceeding, confirm your understanding:

- [ ] I understand why VPS Docker fits this project (existing infrastructure, low traffic, learning value)
- [ ] I understand the deployment workflow (git pull, docker compose build, docker compose up)
- [ ] I've considered the cost (~$0.50/month) and maintenance trade-offs (weekly checks, monthly updates)
- [ ] I'm ready to initialize my project with project-spinup

**Ready to proceed to project-spinup?**
