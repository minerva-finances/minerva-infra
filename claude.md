# Minerva Infrastructure - Claude Instructions

> Instructions for Claude Code AI assistant when working on this service.

---

## Agent Behavior

### How to Approach Tasks

1. **Infrastructure as Code**
   - All infrastructure defined in code
   - Docker Compose for local and production
   - Nginx as reverse proxy only

2. **Security First**
   - Only Nginx exposed (ports 80/443)
   - All services on private network
   - No secrets in code

3. **Think in Services**
   - Each service is a container
   - Services communicate via internal network
   - External access only through Nginx

### Development Workflow

1. Read existing docker-compose and nginx configs
2. Add/modify service definition
3. Update Nginx routes if needed
4. Test locally with `docker-compose up`
5. **Update documentation in minerva-docs**

---

## Code Generation Rules

### File Organization

```
minerva-infra/
├── docker-compose.yml         # Main orchestration
├── nginx/
│   ├── nginx.conf              # Main configuration
│   ├── conf.d/                 # Route-specific configs
│   └── ssl/                    # SSL certificates (gitignored)
├── scripts/
│   ├── deploy.sh               # Deployment
│   ├── backup.sh               # Database backup
│   └── restore.sh              # Database restore
└── README.md
```

### Code Patterns - DO's

```yaml
# ✅ CORRECT: Service definition
version: '3.8'

services:
  backend:
    image: ghcr.io/minerva/minerva-api:latest
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=redis://redis:6379
    expose:
      - "8080"  # Internal only
    networks:
      - private
    restart: unless-stopped

# ✅ CORRECT: Health check
  backend:
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:8080/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

```nginx
# ✅ CORRECT: Nginx route with rate limiting
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

location /api/ {
    limit_req zone=api burst=20 nodelay;
    proxy_pass http://backend:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

### Code Patterns - DON'Ts

```yaml
# ❌ WRONG: Exposing service ports directly
services:
  backend:
    ports:
      - "8080:8080"  # Only Nginx should be exposed

# ❌ WRONG: Secrets in code
services:
  backend:
    environment:
      - JWT_SECRET=super-secret  # Use .env instead

# ❌ WRONG: No restart policy
services:
  backend:
    # Missing: restart: unless-stopped
```

### Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Exposing internal ports | Use `expose:` instead of `ports:` |
| Secrets in docker-compose | Use environment variables from `.env` |
| Missing health checks | Add healthcheck to all services |
| No restart policy | Add `restart: unless-stopped` |
| Not using internal network | Create private network for services |

---

## Testing Rules (CRITICAL)

### When to Test

- After any docker-compose change
- After Nginx config change
- Before deploying to production

### Test Structure

Infrastructure tests are manual integration tests:

```bash
# Test scripts/
scripts/test-infrastructure.sh
```

### Test Patterns

```bash
#!/bin/bash
# scripts/test-infrastructure.sh

echo "Testing infrastructure..."

# Test services are healthy
docker-compose ps | grep -q "Up" || exit 1

# Test API health
curl -f http://localhost/api/health || exit 1

# Test frontend is served
curl -f http://localhost/ || exit 1

# Test backend not directly accessible
curl -f http://localhost:8080/ && exit 1  # Should fail

echo "All tests passed!"
```

### Running Tests

```bash
# Start services
docker-compose up -d

# Run tests
./scripts/test-infrastructure.sh

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

---

## Documentation Rules (CRITICAL)

### What to Document

| Change Type | Documentation Location |
|-------------|----------------------|
| New service | `minerva-docs/docs/architecture/deployment.md` |
| Network change | `minerva-docs/docs/architecture/network-topology.md` |
| Environment variable | `minerva-docs/docs/architecture/deployment.md` |
| Deployment change | `minerva-docs/docs/architecture/deployment.md` |

### How to Document

**DO:**
- Document new services and their purpose
- Note any new environment variables
- Update deployment instructions
- Document network topology changes

**DON'T:**
- Copy entire docker-compose.yml
| Document internal implementation details

### Documentation Workflow

When modifying infrastructure:

1. **Before coding** - Read deployment docs
2. **While coding** - Note changes needed to docs
3. **After coding** - Update architecture docs

### Example: Adding Redis Cache

If you add Redis to docker-compose:

Update `minerva-docs/docs/architecture/deployment.md`
```markdown
### Redis Cache

**Purpose:** Caching API responses and rate limiting

**Configuration:**
- Image: redis:alpine
- Port: 6379 (internal only)
- Persistence: AOF enabled
- TTL: 30 minutes for API responses
```

Update `minerva-docs/docs/architecture/network-topology.md`
```markdown
### Private Network

The following services communicate on a private Docker network:
- Backend (Go)
- Scraper (Node.js)
- Worker (Go)
- Redis
```

---

## Tool Usage

### When to Use Each Tool

| Situation | Tool |
|-----------|------|
| Read config files | `Read` tool |
| Modify config | `Edit` tool |
| Create new config | `Write` tool |
| Run docker commands | `Bash` tool |
| Deploy | `Bash` tool |

### Common Commands

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs -f [service]

# Restart service
docker-compose restart [service]

# Pull latest images
docker-compose pull

# Rebuild and start
docker-compose up -d --build

# Check status
docker-compose ps
```

---

## Context References

### Files to Read Before Modifying

| Task | Read First |
|------|------------|
| Add new service | `docker-compose.yml` |
| Add route | `nginx/conf.d/` |
| Modify SSL | `nginx/nginx.conf` |
| Add script | `scripts/` |

### Key Technologies

- Docker Compose - Orchestration
- Nginx - Reverse proxy, static files
- Let's Encrypt - SSL certificates (certbot)
- Oracle 26ai - Database (external, but configured here)
- Redis - Cache

---

## Quick Reference

| Area | Standard |
|------|----------|
| Purpose | Docker Compose orchestration |
| Cloud Provider | Oracle Cloud Infrastructure (OCI) - Always Free |
| Services | Nginx, Backend, Frontend, Scraper, Worker, Redis |
| Databases | Oracle 26ai Autonomous DB, Redis |
| Deployment | Docker Compose on OCI ARM VM |
| Docs Location | `minerva-docs/docs/architecture/` |

---

## Global Rules (from Minerva Constitution)

1. **English for code** - Comments, variable names in English
2. **Commit convention** - `<type>(<scope>): <description>`
3. **Security** - No secrets in code, use environment variables
4. **Docs REQUIRED** - Document infrastructure changes
