# Docker Setup for Minerva Development

This setup provides a complete local development environment for the Minerva project using Docker and Docker Compose.

## Services Included

1. **Oracle Database** - Oracle Free 23c database
2. **Redis** - Redis cache service
3. **Minerva API** - Go backend service
4. **Minerva Scraper** - Node.js Yahoo Finance scraper

## Quick Start

### 1. Prerequisites
- Docker (v20.10+)
- Docker Compose (v2.0+)
- Docker Compose V2 (if using .env files)

### 2. Environment Setup

Copy and update the environment files:
```bash
cp .env.development .env.local
```

Edit `.env.local` with your Google OAuth credentials:
```env
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
```

### 3. Start the Services

#### Production Mode
```bash
docker-compose up -d
```

#### Development Mode (with hot-reload)
```bash
# Start database and Redis first
docker-compose up -d oracle-db redis

# Start services in development mode
docker-compose --profile dev up minerva-api-dev minerva-scraper-dev
```

### 4. Access Services

- **Minerva API**: http://localhost:8080
- **API Health Check**: http://localhost:8080/api/health
- **Minerva Scraper**: http://localhost:3001
- **Oracle Database**: localhost:1521
- **Redis**: localhost:6379

## Configuration Details

### Oracle Database
- Default username: `system`
- Default password: `Minerva2024!`
- Service name: `XE`
- Connection string: `oracle-xdb://system:Minerva2024!@localhost:1521/XE`

### API Service
- Default port: 8080
- Environment variables configured in `.env.development`
- Health check endpoint: `/api/health`

### Scraper Service
- Default port: 3001
- Built with hot-reload support for development

## Development Workflow

### Hot-Reload Development
For better development experience with code changes:

1. Start the database and Redis:
```bash
docker-compose up -d oracle-db redis
```

2. Start the services with hot-reload:
```bash
# For API service (requires air)
cd minerva-api
docker-compose --profile dev up minerva-api-dev

# For scraper service
cd minerva-scraper
docker-compose --profile dev up minerva-scraper-dev
```

### Manual Build and Start
```bash
# Build all services
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f minerva-api
docker-compose logs -f minerva-scraper
```

### Stop Services
```bash
# Stop all services
docker-compose down

# Stop and remove volumes (to reset data)
docker-compose down -v

# Stop development services only
docker-compose --profile dev down
```

## Database Initialization

The Oracle database is pre-configured with:
- Character set: AL32UTF8
- Tablespaces: USERS, SYSAUX, SYSTEM
- Initialization scripts from `minerva-infra/docker-oracle/init/`

### Reset Database
```bash
# Stop and remove Oracle volume
docker-compose down -v

# Start Oracle again (will recreate from scratch)
docker-compose up -d oracle-db
```

## Volume Management

The following Docker volumes are created:
- `oracle-data`: Oracle database files
- `redis-data`: Redis data persistence

### View Volume Usage
```bash
docker system df -v
```

### Backup Volumes
```bash
# Backup Oracle data
docker run --rm -v oracle-data:/data -v $(pwd):/backup alpine tar cvf /backup/oracle-backup.tar -C /data .

# Backup Redis data
docker run --rm -v redis-data:/data -v $(pwd):/backup alpine tar cvf /backup/redis-backup.tar -C /data .
```

## Network Configuration

All services are connected to the `minerva-network` bridge network with:
- Subnet: 172.20.0.0/16
- Internal service communication via service names

## Troubleshooting

### Oracle Database Not Starting
- Oracle takes 2-5 minutes to initialize
- Check logs: `docker-compose logs -f oracle-db`
- Look for "DATABASE IS READY TO USE" message

### Port Conflicts
- If ports are already in use, change them in docker-compose.yml
- Common ports: 1521 (Oracle), 6379 (Redis), 8080 (API), 3001 (Scraper)

### Connection Issues
- Verify all services are running: `docker-compose ps`
- Check health: `docker-compose ps --filter "status=running"`
- Test connections manually:
```bash
# Test Oracle
echo "SELECT * FROM DUAL;" | sqlplus system/Minerva2024!@localhost:1521/XE

# Test Redis
redis-cli ping
```

### Build Failures
- Clear Docker cache: `docker system prune -a`
- Rebuild: `docker-compose build --no-cache`
- Check Dockerfile syntax

## Additional Configuration

### Environment Variables
Modify `.env.development` for different configurations:
- Development vs production settings
- Database connection strings
- OAuth credentials
- Logging levels

### Debug Mode
Add debug flags to Dockerfile:
```dockerfile
ENV DEBUG=true
ENV LOG_LEVEL=debug
```

### Resource Limits
Add resource limits to docker-compose.yml:
```yaml
services:
  oracle-db:
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G
```