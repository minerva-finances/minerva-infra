# minerva-infra

Infrastructure configuration for Minerva project.

## Overview

Docker Compose configuration for local development and production deployment.

## Components

- **Oracle 26ai Autonomous DB** - Primary database (hosted on OCI)
- **Redis** - Caching layer

## Services

- `minerva-api` - Backend API (Go) - Port 8080
- `minerva-ui` - Frontend (React) - Port 5173
- `minerva-scraper` - Market data scraper (Node.js) - Port 3001
- `minerva-worker` - Background jobs (Go)
- `redis` - Redis Cache - Port 6379

## Prerequisites

Before starting the services, ensure you have:
1. Oracle Autonomous DB credentials and connection string
2. Set `DATABASE_URL` in docker-compose.yml with your OCI DB connection details

## Quick Start

```bash
# Edit DATABASE_URL with your OCI credentials first!
# Then start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

## Environment Variables

Update these in docker-compose.yml before starting:

```yaml
DATABASE_URL: oracle-xdb://user:password@host:port/service_name
```

Get your connection string from Oracle Cloud Console:
1. Go to Autonomous Database → Connection
2. Copy the connection string for your wallet
3. Format: `oracle-xdb://ADMIN:password@hostname:1521/service_name`

## Status

**Current Version:** v0.3.0
**Status:** Development environment

## Changes

- v0.3.0: Removed Oracle DB container (use OCI Autonomous DB)
- v0.2.0: Added minerva-ui and minerva-worker services
- v0.1.0: Initial version

## Documentation

See [minerva-docs](https://github.com/minerva-finances/minerva-docs) for complete documentation.
