# PR 1.2: Docker Development Environment

**Branch:** `1.2-docker-setup`  
**Description:** Configure Docker Compose for local PostgreSQL 16 + pgvector development environment

## Goal

Provide a containerized local development environment with PostgreSQL 16 and pgvector extension that developers can start with a single command. This ensures consistent database setup across all development machines and eliminates manual PostgreSQL installation requirements.

## Why This Approach

Docker Compose provides zero-configuration database setup for local development. By containerizing PostgreSQL with the pgvector extension, we ensure:
- Consistent PostgreSQL version (16) across all developers
- Automatic pgvector extension availability for RAG implementation
- Data persistence across container restarts via Docker volumes
- No conflicts with existing local PostgreSQL installations
- Easy teardown and rebuild for testing

This mirrors the production Neon PostgreSQL environment while giving developers full local control.

## Implementation Steps

### Step 1: Docker Compose Configuration
**Folder:** `1-docker-compose-setup/`
**Files:**
- `docker-compose.yml`
- `.dockerignore`

**What:** Create Docker Compose configuration using official `pgvector/pgvector:pg16` image. Configure PostgreSQL with environment variables for database name, user, and password. Set up named volume for data persistence. Expose port 5432 for backend connectivity. Add healthcheck to ensure database readiness. Create `.dockerignore` to exclude unnecessary files from Docker context.

**Testing:**
- Run `docker compose up -d` and verify container starts
- Check logs: `docker compose logs postgres`
- Verify healthcheck: `docker compose ps` (should show "healthy")
- Connect with `psql` or database client to confirm accessibility

### Step 2: Database Initialization Scripts
**Folder:** `2-database-init/`
**Files:**
- `docker/postgres/init-scripts/01-enable-pgvector.sql`

**What:** Create PostgreSQL initialization script that automatically enables the pgvector extension when the database container first starts. Scripts in `/docker-entrypoint-initdb.d/` are executed alphabetically on initial database creation. This ensures pgvector is available immediately for RAG development.

**Testing:**
- Start fresh database: `docker compose down -v && docker compose up -d`
- Wait for healthcheck to pass
- Connect to database and verify: `SELECT * FROM pg_extension WHERE extname = 'vector';`
- Should return one row showing pgvector extension is installed

### Step 3: Backend Environment Configuration
**Folder:** `3-backend-env-config/`
**Files:**
- `src/backend/.env.example` (update)

**What:** Update backend environment template to include Docker-specific `DATABASE_URL` examples. Add clear comments documenting the difference between local Docker connection string vs. production Neon connection string. Include examples for both development and production configurations.

**Testing:**
- Copy `.env.example` to `.env`
- Update `DATABASE_URL` with Docker connection string
- Verify backend can connect (will be tested in PR 1.4 when backend is scaffolded)

### Step 4: Root Package Scripts
**Folder:** `4-docker-npm-scripts/`
**Files:**
- `package.json` (root - update scripts)

**What:** Add npm convenience scripts for Docker Compose management. Scripts include: `docker:up` (start containers in detached mode), `docker:down` (stop and remove containers), `docker:logs` (follow postgres logs), `docker:restart` (restart containers), `docker:clean` (remove volumes and reset database).

**Testing:**
- Run `npm run docker:up` and verify containers start
- Run `npm run docker:logs` and see PostgreSQL startup logs
- Run `npm run docker:down` and verify containers stop
- Run each script to ensure they work correctly

### Step 5: Documentation Updates
**Folder:** `5-documentation/`
**Files:**
- `README.md` (update)

**What:** Add comprehensive "Local Development Setup" section to root README. Document Docker prerequisites (Docker Desktop 4.0+ for Mac/Windows, Docker Engine 20.10+ for Linux). Provide step-by-step setup instructions including Docker installation verification, starting database, connecting from backend, and stopping containers. Include troubleshooting section for common issues: port 5432 conflicts, volume permissions on Linux, M1/M2 Mac compatibility, connection refused errors.

**Testing:**
- Follow README instructions from scratch on clean machine
- Verify all commands work as documented
- Ensure troubleshooting section covers actual issues encountered

## Success Criteria

- ✅ `docker compose up -d` starts PostgreSQL 16 container with pgvector extension
- ✅ Database container shows "healthy" status in `docker compose ps`
- ✅ pgvector extension is enabled and queryable
- ✅ Data persists across `docker compose restart`
- ✅ Backend can connect using `DATABASE_URL` from `.env` file
- ✅ npm scripts (`docker:up`, `docker:down`, `docker:logs`) work correctly
- ✅ README provides clear setup instructions with troubleshooting
- ✅ `.dockerignore` excludes unnecessary files from Docker context

## Commit Message

`feat(docker): add PostgreSQL 16 + pgvector development environment`

**Extended description:**
- Configure Docker Compose with pgvector/pgvector:pg16 image
- Add database initialization script to enable pgvector extension
- Set up volume persistence for local development data
- Add npm scripts for Docker container management
- Update README with Docker setup and troubleshooting guide

---

## Architecture & Technology Stack

### Recommended Approach

**Docker Compose** for local development database orchestration. This provides:
- Consistent PostgreSQL 16 environment matching production version
- Pre-configured pgvector extension for RAG vector search
- Zero manual installation (developers only need Docker Desktop)
- Volume-based persistence for development data
- Easy reset and cleanup for testing scenarios
- No port conflicts with existing PostgreSQL installations (configurable port mapping)

### Key Technologies

- **PostgreSQL 16**: Latest stable PostgreSQL version with improved performance
- **pgvector 0.5+**: Vector similarity search extension for RAG implementation
- **Docker Compose V2**: Modern container orchestration (bundled with Docker Desktop)
- **Docker volumes**: Named volume (`dovvydive-postgres-data`) for data persistence

### High-Level Architecture

```
Developer Machine
├── Docker Desktop / Docker Engine
│   └── Docker Compose
│       └── PostgreSQL 16 Container
│           ├── pgvector extension (enabled)
│           ├── dovvydive database
│           ├── Volume: dovvydive-postgres-data (persistence)
│           └── Port 5432 → Host 5432
│
├── Backend (Node.js/Express)
│   └── Connects via DATABASE_URL=postgresql://localhost:5432/dovvydive
│
└── Local Tools (Optional)
    ├── psql CLI
    ├── pgAdmin / Adminer (web GUI)
    └── TablePlus / DBeaver (desktop GUI)
```

---

## Implementation Plan

### Scope & Objectives

**Primary Goal**: Provide a one-command local database setup that mirrors production PostgreSQL + pgvector configuration.

**User Impact**: Developers can start backend development immediately without PostgreSQL installation, version management, or pgvector compilation. Eliminates "works on my machine" database issues.

**Success Criteria**:
- ✅ Single command (`docker compose up`) starts fully-configured database
- ✅ pgvector extension pre-enabled and ready for vector operations
- ✅ Data persists between coding sessions
- ✅ Clear documentation for setup and troubleshooting

### Key Work Items

1. **Docker Compose Configuration**
   - Create `docker-compose.yml` with PostgreSQL 16 + pgvector service
   - Use official `pgvector/pgvector:pg16` Docker image
   - Configure environment variables: `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`
   - Set up named volume for data persistence
   - Add healthcheck for database readiness detection

2. **Database Initialization**
   - Create `docker/postgres/init-scripts/01-enable-pgvector.sql`
   - Enable pgvector extension: `CREATE EXTENSION IF NOT EXISTS vector;`
   - Scripts auto-execute on first container startup

3. **Environment Configuration**
   - Update `src/backend/.env.example` with Docker connection string
   - Document local vs. production `DATABASE_URL` format
   - Add comments explaining each environment variable

4. **Developer Convenience Scripts**
   - Add npm scripts to root `package.json`:
     - `docker:up`: Start containers (`docker compose up -d`)
     - `docker:down`: Stop containers (`docker compose down`)
     - `docker:logs`: View postgres logs (`docker compose logs -f postgres`)
     - `docker:restart`: Restart containers
     - `docker:clean`: Remove volumes and reset database

5. **Documentation**
   - Update root `README.md` with "Docker Setup" section
   - Prerequisites: Docker Desktop 4.0+ or Docker Engine 20.10+
   - Step-by-step setup instructions
   - Troubleshooting: port conflicts, permissions, connection issues
   - Document how to add optional services (Adminer, Redis) manually

6. **`.dockerignore` File**
   - Exclude: `node_modules/`, `.git/`, `.env*`, `dist/`, `.next/`, `build/`
   - Optimize Docker context for future Dockerfile builds (PR 7.6)

### Single PR Summary

**PR Name:** Docker Development Environment  
**Branch:** `1.2-docker-setup`  
**Description:** Configure Docker Compose for local PostgreSQL 16 + pgvector development environment

**Goal:** Enable developers to run local database with pgvector extension using a single command

**Key Components/Files:**
- `docker-compose.yml` (PostgreSQL 16 service with pgvector)
- `docker/postgres/init-scripts/01-enable-pgvector.sql` (database initialization)
- `.dockerignore` (optimize Docker build context)
- `src/backend/.env.example` (updated with Docker connection string)
- `package.json` (root - Docker management scripts)
- `README.md` (Docker setup documentation)

**Dependencies:** PR 1.1 (monorepo structure must exist)

**Tech Details:**
- **Docker Image**: `pgvector/pgvector:pg16` (official image with pgvector pre-compiled)
- **Database Name**: `dovvydive`
- **Port Mapping**: `5432:5432` (configurable if conflicts exist)
- **Volume**: `dovvydive-postgres-data` (named volume for persistence)
- **Healthcheck**: `pg_isready -U dovvydive` every 10 seconds
- **Connection String**: `postgresql://dovvydive:dev_password@localhost:5432/dovvydive`

**Testing Approach:**
- Start containers: `npm run docker:up`
- Verify healthcheck: `docker compose ps` (status should be "Up (healthy)")
- Connect with psql: `psql -h localhost -U dovvydive -d dovvydive`
- Check pgvector: `SELECT * FROM pg_extension WHERE extname = 'vector';`
- Test persistence: Insert data, restart containers, verify data remains
- Test cleanup: `npm run docker:clean` removes all data and volumes

---

## Implementation Sequence

1. **Create `docker-compose.yml`** with PostgreSQL 16 + pgvector service definition
2. **Create `.dockerignore`** to exclude unnecessary files
3. **Create database initialization script** (`01-enable-pgvector.sql`)
4. **Update `src/backend/.env.example`** with Docker-specific `DATABASE_URL`
5. **Add Docker management scripts** to root `package.json`
6. **Update root `README.md`** with Docker setup section
7. **Test the setup**:
   - Start containers: `npm run docker:up`
   - Check health: `docker compose ps`
   - Connect to database and verify pgvector extension
   - Test data persistence across restarts
   - Test cleanup: `npm run docker:clean`

---

## Testing Strategy

### Manual Validation

**Fresh Setup Test:**
1. Clone repository on clean machine
2. Run `npm run docker:up`
3. Verify container starts and becomes healthy within 30 seconds
4. Connect with psql or database GUI
5. Verify pgvector extension is enabled

**Persistence Test:**
1. Start containers: `npm run docker:up`
2. Create test table and insert data
3. Stop containers: `npm run docker:down`
4. Restart containers: `npm run docker:up`
5. Verify data still exists

**Cleanup Test:**
1. Run `npm run docker:clean`
2. Verify volumes are removed: `docker volume ls`
3. Restart containers: `npm run docker:up`
4. Verify fresh database (no previous data)

**Troubleshooting Tests:**
- Simulate port 5432 conflict (start local PostgreSQL first)
- Test on different OS (Mac, Windows, Linux)
- Test M1/M2 Mac compatibility (ARM64 architecture)

### Automated Tests

Not applicable for this PR (infrastructure only). Backend integration tests will validate database connectivity in PR 1.4.

### Verification Checklist

- [ ] `docker compose up -d` starts PostgreSQL container successfully
- [ ] Container healthcheck passes within 30 seconds
- [ ] pgvector extension is enabled: `\dx` shows `vector` extension
- [ ] Can connect from host using `psql` or database GUI
- [ ] Data persists across container restarts
- [ ] `docker compose down` cleanly stops containers
- [ ] `docker compose down -v` removes volumes
- [ ] npm scripts work: `docker:up`, `docker:down`, `docker:logs`, `docker:clean`
- [ ] README instructions are accurate and complete
- [ ] Troubleshooting section covers common issues

---

## Success Criteria

**Technical Completion:**
- ✅ PostgreSQL 16 container runs with pgvector extension enabled
- ✅ Database accessible from host machine on port 5432
- ✅ Data persists across container restarts via Docker volumes
- ✅ Healthcheck confirms database readiness before backend connections
- ✅ Initialization scripts automatically enable pgvector on first startup

**Developer Experience:**
- ✅ Developers can start database with single command: `npm run docker:up`
- ✅ No manual PostgreSQL installation or configuration required
- ✅ Clear error messages if Docker is not installed
- ✅ npm scripts provide convenient database management
- ✅ README explains setup clearly for all experience levels

**Documentation:**
- ✅ README includes:
  - Docker prerequisites and installation links
  - Step-by-step setup instructions
  - Database connection string examples
  - Troubleshooting guide for common issues
  - Commands for viewing logs and managing containers
- ✅ Environment variable documentation updated
- ✅ Comments in `docker-compose.yml` explain each configuration option

---

## Known Constraints & Considerations

**Docker Requirements:**
- **Mac/Windows**: Docker Desktop 4.0+ required (includes Docker Compose V2)
- **Linux**: Docker Engine 20.10+ and Docker Compose V2 plugin required
- **M1/M2 Macs**: `pgvector/pgvector:pg16` supports ARM64 natively (no Rosetta needed)

**Port Conflicts:**
- Default PostgreSQL port 5432 may conflict with existing installations
- **Solution**: Change port mapping in `docker-compose.yml` to `5433:5432` and update `DATABASE_URL`
- Document port customization in README

**Volume Persistence:**
- Data survives `docker compose down` but NOT `docker compose down -v`
- **Solution**: Document difference between `down` and `down -v` clearly
- Recommend `docker:clean` script for intentional data wipes

**Performance Considerations:**
- Docker Desktop uses VM on Mac/Windows (slight overhead vs. native)
- Production uses Neon (serverless PostgreSQL) - different performance characteristics
- **Solution**: Document that Docker is for development only; performance testing should use staging/production

**Data Seeding:**
- No seed data included in this PR
- **Rationale**: Seed data belongs in PR 2.2 (Data Ingestion Pipeline) with proper markdown parsing and embedding generation
- **Workaround**: Developers can manually insert test data if needed before PR 2.2

**Environment Variables:**
- Development uses simple password in `.env` (not suitable for production)
- **Solution**: Document that production uses Neon connection string with SSL and proper credentials

**Linux-Specific Issues:**
- Volume permissions may require `chown` on mounted directories
- **Solution**: README includes Linux-specific troubleshooting:
  ```bash
  # Fix volume permissions on Linux
  sudo chown -R $USER:$USER ./docker/postgres/data
  ```

**Optional Services:**
- No Redis, Adminer, or other services included in base `docker-compose.yml`
- **Rationale**: Keep MVP minimal; Redis added in PR 4.1 when sessions are implemented
- **Workaround**: README documents how to add optional services manually

**Database Migrations:**
- No Prisma migrations in this PR (schema defined in PR 1.3)
- Initialization script only enables pgvector extension
- **Future**: PR 1.3 will add Prisma schema and migration commands

**Connection String Format:**
Local (Docker):
```
DATABASE_URL=postgresql://dovvydive:dev_password@localhost:5432/dovvydive
```

Production (Neon):
```
DATABASE_URL=postgresql://user:password@ep-cool-name-123456.us-east-2.aws.neon.tech/dovvydive?sslmode=require
```

**Docker Compose V2 vs V1:**
- This PR uses Docker Compose V2 (`docker compose` without hyphen)
- V1 (`docker-compose` with hyphen) is deprecated as of July 2023
- **Migration**: All scripts and documentation use V2 syntax

---

**Plan Status:** ✅ Finalized  
**Created:** 1 December 2025  
**Next Steps:** Begin implementation of PR 1.2 on branch `1.2-docker-setup`
