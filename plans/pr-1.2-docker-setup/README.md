# PR 1.2: Docker Development Environment - Quick Reference

**Status:** Ready for Implementation  
**Branch:** `1.2-docker-setup`  
**Dependencies:** PR 1.1 (Monorepo Initialization)

## Summary

Set up Docker Compose with PostgreSQL 16 + pgvector for local development. Enables developers to run the complete database stack with a single `npm run docker:up` command.

## Key Deliverables

- Docker Compose configuration with PostgreSQL 16 + pgvector
- Database initialization script to enable pgvector extension
- Docker management npm scripts
- Updated README with setup and troubleshooting guide
- `.dockerignore` for optimized Docker contexts

## Quick Commands

```bash
# Start database
npm run docker:up

# View logs
npm run docker:logs

# Stop database
npm run docker:down

# Clean database (remove all data)
npm run docker:clean
```

## Files Created/Modified

### New Files
- `docker-compose.yml` - PostgreSQL service configuration
- `docker/postgres/init-scripts/01-enable-pgvector.sql` - Extension initialization
- `.dockerignore` - Exclude unnecessary files from Docker context

### Modified Files
- `package.json` - Add Docker management scripts
- `src/backend/.env.example` - Add Docker DATABASE_URL example
- `README.md` - Add Docker setup documentation

## Implementation Steps

1. **Step 1: Docker Compose Setup** - Create `docker-compose.yml` with pgvector image
2. **Step 2: Database Init Scripts** - Enable pgvector extension automatically
3. **Step 3: Backend Env Config** - Update `.env.example` with Docker connection string
4. **Step 4: Docker npm Scripts** - Add convenience commands to `package.json`
5. **Step 5: Documentation** - Update README with setup guide

## Success Criteria

- ✅ `docker compose up` starts PostgreSQL 16 with pgvector
- ✅ Container shows "healthy" status
- ✅ Backend can connect using `DATABASE_URL`
- ✅ Data persists across container restarts
- ✅ Clear documentation and troubleshooting guide

## Connection Details

**Local Development:**
```
Host: localhost
Port: 5432
Database: dovvydive
User: dovvydive
Password: dev_password
```

**Connection String:**
```
postgresql://dovvydive:dev_password@localhost:5432/dovvydive
```

## Testing Checklist

- [ ] Start containers: `npm run docker:up`
- [ ] Verify health: `docker compose ps` (should show "Up (healthy)")
- [ ] Connect with psql: `psql -h localhost -U dovvydive -d dovvydive`
- [ ] Check pgvector: `SELECT * FROM pg_extension WHERE extname = 'vector';`
- [ ] Test persistence: Insert data, restart containers, verify data exists
- [ ] Test cleanup: `npm run docker:clean` removes all data

## Next PR

**PR 1.3:** Prisma Schema & Database Models
