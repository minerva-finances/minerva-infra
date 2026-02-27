# minerva-infra

Infrastructure configuration for Minerva project.

## Overview

Docker Compose configuration for local development and production deployment.

## Components

- **Oracle 26ai Autonomous DB** - Primary database
- **Redis** - Caching layer
- **Nginx** - Reverse proxy

## Services

- `minerva-api` - Backend API (Go)
- `minerva-ui` - Frontend (React)
- `minerva-scraper` - Market data scraper (Node.js)
- `minerva-worker` - Background jobs (Go)

## Status

**Current Version:** v0.1.0
**Status:** Development environment

## Documentation

See [minerva-docs](https://github.com/minerva-finances/minerva-docs) for complete documentation.
