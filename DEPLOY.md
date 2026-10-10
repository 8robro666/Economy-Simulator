# Railway Deployment Guide

This guide explains how to deploy the Economy Simulator to Railway.

## Prerequisites

1. A Railway account (https://railway.app)
2. Railway CLI installed (`npm install -g @railway/cli`)
3. Docker installed locally for testing
4. Git repository with the project code

## Project Structure

The project consists of 7 services that need to be deployed:

1. **API** (Node.js/TypeScript) - Main REST API server
2. **Game Server** (Node.js/TypeScript) - Game server communication & WebSocket
3. **Admin** (Svelte/Webpack) - Admin dashboard
4. **Asset Validation** (Go) - Asset validation service
5. **Website** (.NET 6) - Main website application
6. **RCC Service** (Ubuntu + Xvfb) - Roblox rendering service
7. **Frontend** (Next.js) - 2016-roblox-main frontend

## Quick Deploy

### 1. Login to Railway

```bash
railway login
```

### 2. Initialize Project

```bash
railway init
```

Select "Empty Project" or connect your GitHub repository.

### 3. Add Databases

```bash
# Add PostgreSQL
railway add postgresql

# Add Redis
railway add redis
```

### 4. Configure Environment Variables

Set the following variables in Railway dashboard or via CLI:

```bash
# Required secrets (generate secure random strings)
railway variables set JWT_SECRET="$(openssl rand -base64 32)"
railway variables set SESSION_SECRET="$(openssl rand -base64 32)"
railway variables set GAME_SERVER_AUTH="$(openssl rand -base64 32)"
railway variables set WEBSITE_AUTH="$(openssl rand -base64 32)"
railway variables set WEBSITE_BOT_AUTH="$(openssl rand -base64 32)"
railway variables set ASSET_VALIDATION_AUTH="$(openssl rand -base64 32)"
railway variables set RCC_AUTH="$(openssl rand -base64 32)"

# Base URL (will be your Railway app URL)
railway variables set BASE_URL="https://your-app.railway.app"

# Owner user ID
railway variables set OWNER_USER_ID="1"

# hCaptcha (use test keys for development)
railway variables set HCAPTCHA_PUBLIC="10000000-ffff-ffff-ffff-000000000001"
railway variables set HCAPTCHA_PRIVATE="0x0000000000000000000000000000000000000000"
```

### 5. Deploy Services

Deploy each service using the railway.toml configuration:

```bash
# Deploy all services defined in railway.toml
railway up
```

Or deploy individual services:

```bash
railway up --service api
railway up --service game-server
railway up --service admin
railway up --service asset-validation
railway up --service website
railway up --service rcc-service
railway up --service frontend
```

## Configuration Files

Each service requires a configuration file. Copy the example files and fill in the values:

```bash
# API service
cp services/api/config.example.json services/api/config.json

# Game Server
cp services/game-server/config.example.json services/game-server/config.json

# Frontend
cp services/2016-roblox-main/config.example.json services/2016-roblox-main/config.json
```

The config.json files are gitignored and should be created at deploy time using environment variables.

## Local Development with Docker Compose

For local testing before deploying to Railway:

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down
```

## Service Details

### API Service
- **Port**: 3000
- **Health Check**: `/health`
- **Dependencies**: PostgreSQL, Redis
- **Build**: Multi-stage Docker build (builder + runner)

### Game Server
- **Ports**: 3000 (HTTP), 3040 (WebSocket)
- **Health Check**: `/health`
- **Dependencies**: API service
- **Build**: Multi-stage Docker build

### Admin Dashboard
- **Port**: 80 (nginx)
- **Health Check**: `/`
- **Dependencies**: API service
- **Build**: Multi-stage (webpack build + nginx)

### Asset Validation Service
- **Port**: 4300
- **Health Check**: `/`
- **Dependencies**: None
- **Build**: Go multi-stage build

### Website (.NET 6)
- **Port**: 80
- **Health Check**: `/health`
- **Dependencies**: PostgreSQL, Redis, Asset Validation, Game Server
- **Build**: .NET SDK build + ASP.NET runtime

### RCC Service
- **Port**: 3189
- **Health Check**: `/health` (if implemented)
- **Dependencies**: Game Server
- **Build**: Ubuntu with Xvfb
- **Note**: Requires Linux build of RCCService (rcc binary)

### Frontend (Next.js)
- **Port**: 3000
- **Health Check**: `/`
- **Dependencies**: API service
- **Build**: Next.js standalone build

## Important Notes

### RCC Service Linux Binary
The RCC Service requires a Linux build of the rendering engine. The current repository only contains Windows binaries (RCCService.exe). You will need to:

1. Compile the RCC Service for Linux (Ubuntu 22.04)
2. Place the `rcc` binary in `services/RCCService/`
3. Ensure all required Linux shared libraries are available

### Database Migrations
Run database migrations after deploying the API service:

```bash
railway run --service api npx knex migrate:latest
```

### HTTPS
Railway automatically provides HTTPS. Update `BASE_URL` to use `https://` once deployed.

### Scaling
Each service can be scaled independently in the Railway dashboard based on load.

## Troubleshooting

### Service Fails to Start
1. Check logs: `railway logs --service <service-name>`
2. Verify environment variables are set correctly
3. Check health check endpoint is implemented

### Database Connection Issues
1. Ensure PostgreSQL service is running
2. Verify DATABASE_URL format
3. Check firewall/network policies

### RCC Service Issues
1. Ensure Linux binary exists at `services/RCCService/rcc`
2. Check Xvfb is starting correctly
3. Verify GPU access if needed (configure in railway.toml)

## Environment Variable Reference

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string | Yes (auto-set by Railway) |
| `REDIS_URL` | Redis connection string | Yes (auto-set by Railway) |
| `JWT_SECRET` | JWT signing secret | Yes |
| `SESSION_SECRET` | Session encryption secret | Yes |
| `GAME_SERVER_AUTH` | Game server auth token | Yes |
| `WEBSITE_AUTH` | Website auth token | Yes |
| `WEBSITE_BOT_AUTH` | Bot auth token | Yes |
| `ASSET_VALIDATION_AUTH` | Asset validation auth | Yes |
| `RCC_AUTH` | RCC service auth | Yes |
| `BASE_URL` | Public base URL | Yes |
| `OWNER_USER_ID` | Admin user ID | Yes |
| `HCAPTCHA_PUBLIC` | hCaptcha public key | Yes |
| `HCAPTCHA_PRIVATE` | hCaptcha private key | Yes |

## Support

For issues with Railway deployment, check:
- Railway documentation: https://docs.railway.app
- Railway Discord: https://discord.gg/railway
- Project issues: GitHub Issues