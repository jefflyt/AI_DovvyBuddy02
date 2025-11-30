# DovvyDive - Development Plan

## Project Overview

DovvyDive is an AI-powered dive trip planning platform featuring a conversational chatbot that helps scuba divers discover, plan, and organize dive trips across Malaysia. The MVP leverages Google ADK multi-agent RAG system with Gemini 2.5 Flash, bilingual support (English/Chinese), and focuses on 50+ Malaysian dive sites with guest-only access.

---

## Architecture & Technology Stack

### Recommended Approach

**Monorepo with clear separation of concerns**: Frontend (Next.js SSR), Backend (Express API), and shared types/utilities in a single repository. This approach supports:
- Type sharing between frontend/backend
- Unified deployment pipeline
- Consistent development environment
- RAG-powered multi-agent system with dedicated orchestration layer

### Key Technologies

**Frontend Stack:**
- **Next.js 14+ (App Router)**: Server-side rendering for SEO on dive site pages, client components for chat interface
- **TypeScript**: Type safety across the stack
- **Tailwind CSS**: Rapid UI development with consistent design system
- **Zustand**: Lightweight state management for chat session/conversation state
- **next-i18next**: Bilingual support with automatic language detection

**Backend Stack:**
- **Node.js 20+ with Express**: REST API server with middleware for validation, rate limiting
- **Prisma ORM**: Type-safe database access with native pgvector support
- **Zod**: Runtime validation for API requests
- **Google ADK**: Multi-agent orchestration framework

**AI & Data:**
- **Google Gemini 2.5 Flash**: LLM for all agent responses
- **Neon PostgreSQL + pgvector**: Unified database for relational data + vector embeddings
- **Google text-embedding-004**: Embedding model for RAG pipeline

**Infrastructure:**
- **Docker Compose**: Local development environment
- **Vercel**: Frontend hosting with edge caching
- **Render**: Backend API hosting with auto-scaling

### High-Level Architecture

```
┌─────────────┐
│   Browser   │
└──────┬──────┘
       │ HTTPS/JSON
       ▼
┌─────────────────────┐      ┌──────────────────┐
│  Next.js Frontend   │◄────►│  Cloudflare CDN  │
│  (SSR + Client)     │      └──────────────────┘
└──────┬──────────────┘
       │ Internal API
       ▼
┌─────────────────────┐
│  Express Backend    │
│  (REST API)         │
└──┬────────┬─────────┘
   │        │
   │        ▼
   │   ┌─────────────────┐
   │   │  Google ADK     │
   │   │  Multi-Agent    │◄──► Google Gemini 2.5 Flash
   │   │  Orchestration  │
   │   └────────┬────────┘
   │            │
   ▼            ▼
┌──────────────────────────┐
│  Neon PostgreSQL         │
│  (Relational + pgvector) │
└──────────────────────────┘
```

---

## Project Phases & PR Breakdown

### Phase 1: Foundation & Infrastructure Setup
**Goal**: Establish development environment, database schema, and project scaffolding

#### PR 1.1: Monorepo Initialization & Configuration
**Branch:** `1.1-monorepo-init`  
**Description:** Initialize monorepo with TypeScript, linting, and shared configuration  
**Goal:** Create foundational project structure with development tooling  
**Key Components/Files:**
- Root `package.json` with workspace configuration
- `tsconfig.json` (root + per-workspace)
- ESLint + Prettier configuration
- `.env.example` files for environment variables
- `src/shared/` types and utilities setup

**Dependencies:** None

#### PR 1.2: Docker Development Environment
**Branch:** `1.2-docker-setup`  
**Description:** Configure Docker Compose for local Postgres + pgvector  
**Goal:** Enable local development with containerized database  
**Key Components/Files:**
- `docker-compose.yml` (Postgres 16 + pgvector extension)
- `docker-compose.dev.yml` (with volume mounts)
- Database initialization scripts
- README updates for local setup

**Dependencies:** PR 1.1

#### PR 1.3: Prisma Schema & Database Models
**Branch:** `1.3-prisma-schema`  
**Description:** Implement complete database schema with Prisma  
**Goal:** Define data models for sessions, dive sites, operators, conversations  
**Key Components/Files:**
- `src/backend/prisma/schema.prisma`
- Session, Conversation, Message models
- DiveSite model with pgvector columns
- DiveOperator model
- Initial migration files

**Dependencies:** PR 1.2

#### PR 1.4: Backend API Scaffolding
**Branch:** `1.4-backend-scaffold`  
**Description:** Set up Express server with middleware and health endpoint  
**Goal:** Create backend foundation with logging, validation, error handling  
**Key Components/Files:**
- `src/backend/src/server.ts` (Express app)
- Middleware: CORS, rate limiting, request validation
- `src/backend/src/routes/health.ts`
- Error handling utilities
- Logger configuration (Winston or Pino)

**Dependencies:** PR 1.3

#### PR 1.5: Frontend Next.js Setup
**Branch:** `1.5-frontend-scaffold`  
**Description:** Initialize Next.js 14 with App Router, Tailwind, and i18n  
**Goal:** Create frontend foundation with bilingual support  
**Key Components/Files:**
- `src/frontend/` Next.js project initialization
- Tailwind CSS configuration
- `next-i18next` setup with English/Chinese locale files
- Root layout with language detection
- Basic homepage placeholder

**Dependencies:** PR 1.1

---

### Phase 2: Data Pipeline & RAG Foundation
**Goal**: Ingest dive site data and implement vector search capabilities

#### PR 2.1: Dive Site Data Schema & Seed Data
**Branch:** `2.1-dive-site-data`  
**Description:** Create structured dive site data in markdown/JSON format  
**Goal:** Prepare initial 10 Malaysian dive sites with bilingual content  
**Key Components/Files:**
- `src/backend/data/dive-sites/` (markdown files per site)
- YAML frontmatter with structured metadata
- Bilingual descriptions (English + Chinese)
- Site metadata: location, depth, difficulty, marine life

**Dependencies:** PR 1.3

#### PR 2.2: Data Ingestion Pipeline
**Branch:** `2.2-ingestion-pipeline`  
**Description:** Build scripts to parse markdown and generate embeddings  
**Goal:** Transform dive site data into database records with vector embeddings  
**Key Components/Files:**
- `src/backend/scripts/ingest.ts`
- Markdown parser with frontmatter extraction
- Google text-embedding-004 integration
- Chunking strategy (500 tokens per chunk)
- Prisma seed script

**Dependencies:** PR 2.1

#### PR 2.3: Vector Search API
**Branch:** `2.3-vector-search`  
**Description:** Implement semantic search endpoint over dive sites  
**Goal:** Enable similarity-based retrieval for RAG pipeline  
**Key Components/Files:**
- `src/backend/src/services/vectorSearch.ts`
- SQL queries with pgvector operators (`<->`, `<#>`)
- HNSW index creation migration
- `POST /api/v1/search/semantic` endpoint
- Language-aware filtering

**Dependencies:** PR 2.2

#### PR 2.4: Dive Site REST API
**Branch:** `2.4-dive-site-api`  
**Description:** Create endpoints for dive site retrieval and filtering  
**Goal:** Support frontend dive site browsing and SEO pages  
**Key Components/Files:**
- `GET /api/v1/sites` (list with filters)
- `GET /api/v1/sites/:slug` (detail page)
- Query parameters: difficulty, depth, region
- Response pagination

**Dependencies:** PR 2.3

---

### Phase 3: Multi-Agent AI System
**Goal**: Implement Google ADK agents and chat orchestration

#### PR 3.1: Google ADK Integration & Base Agent
**Branch:** `3.1-adk-foundation`  
**Description:** Set up Google ADK framework and base agent configuration  
**Goal:** Establish connection to Gemini 2.5 Flash with ADK  
**Key Components/Files:**
- `src/backend/src/agents/base/` (base agent class)
- Google ADK SDK initialization
- Gemini API configuration
- Agent prompt template system
- Conversation context management

**Dependencies:** PR 1.4

#### PR 3.2: Router Agent
**Branch:** `3.2-router-agent`  
**Description:** Implement intent classification and routing logic  
**Goal:** Classify user queries and route to specialized agents  
**Key Components/Files:**
- `src/backend/src/agents/router.ts`
- Intent classification prompts (DISCOVERY, PLANNING, SAFETY, WEATHER)
- Routing decision tree
- Agent registry/factory pattern

**Dependencies:** PR 3.1

#### PR 3.3: Dive Site Expert Agent (RAG)
**Branch:** `3.3-dive-site-agent`  
**Description:** Build specialized agent for dive site recommendations  
**Goal:** Combine vector search with LLM generation for site recommendations  
**Key Components/Files:**
- `src/backend/src/agents/diveSiteAgent.ts`
- RAG pipeline: query → embed → search → context assembly
- System prompts for dive expertise persona
- Certification-aware filtering logic
- Citation formatting for retrieved sources

**Dependencies:** PR 3.2, PR 2.3

#### PR 3.4: Safety Validation Agent
**Branch:** `3.4-safety-agent`  
**Description:** Implement certification validation and safety warnings  
**Goal:** Enforce safety requirements and provide warnings  
**Key Components/Files:**
- `src/backend/src/agents/safetyAgent.ts`
- Certification level validation (OW, AOW, Rescue, etc.)
- Depth limit checks
- Safety disclaimer templates
- Medical advice refusal logic

**Dependencies:** PR 3.2

#### PR 3.5: Weather & Logistics Agent
**Branch:** `3.5-weather-agent`  
**Description:** Create agent for seasonal insights and weather data  
**Goal:** Provide weather patterns and best diving seasons  
**Key Components/Files:**
- `src/backend/src/agents/weatherAgent.ts`
- Seasonal patterns database (stored in dive site metadata)
- External weather API integration (OpenWeather or similar)
- Monsoon season warnings

**Dependencies:** PR 3.2

#### PR 3.6: Agent Orchestration Service
**Branch:** `3.6-orchestration`  
**Description:** Coordinate multi-agent responses and handoffs  
**Goal:** Enable complex queries requiring multiple agents  
**Key Components/Files:**
- `src/backend/src/services/orchestrator.ts`
- Multi-agent coordination logic
- Context passing between agents
- Response merging and formatting

**Dependencies:** PR 3.2, PR 3.3, PR 3.4, PR 3.5

---

### Phase 4: Chat Interface & Session Management
**Goal**: Build conversational UI and session handling

#### PR 4.1: Session Management Backend
**Branch:** `4.1-session-backend`  
**Description:** Implement guest session creation and persistence  
**Goal:** Track conversations without user accounts  
**Key Components/Files:**
- `POST /api/v1/sessions` (create session)
- `GET /api/v1/sessions/:id` (retrieve session)
- Session expiry logic (24-hour TTL)
- Browser session token generation (UUID)

**Dependencies:** PR 1.4

#### PR 4.2: Chat API Endpoint
**Branch:** `4.2-chat-api`  
**Description:** Create main chat endpoint connecting frontend to agents  
**Goal:** Handle user messages and return AI responses  
**Key Components/Files:**
- `POST /api/v1/chat/message`
- Request validation (Zod schema)
- Session validation middleware
- Integration with orchestration service
- Conversation history storage
- Rate limiting (100 msg/hour)

**Dependencies:** PR 4.1, PR 3.6

#### PR 4.3: Chat UI Component
**Branch:** `4.3-chat-ui`  
**Description:** Build React chat interface with message display  
**Goal:** Create responsive, accessible chat UI  
**Key Components/Files:**
- `src/frontend/components/Chat/ChatWindow.tsx`
- `src/frontend/components/Chat/MessageList.tsx`
- `src/frontend/components/Chat/MessageInput.tsx`
- Zustand store for chat state
- Typing indicators
- Message timestamp rendering

**Dependencies:** PR 1.5

#### PR 4.4: Session Persistence (Frontend)
**Branch:** `4.4-frontend-session`  
**Description:** Implement browser session storage and restoration  
**Goal:** Persist conversations across page refreshes  
**Key Components/Files:**
- `src/frontend/lib/sessionStorage.ts`
- LocalStorage wrapper for session ID
- Conversation history caching
- Session restoration on page load

**Dependencies:** PR 4.3

#### PR 4.5: Bilingual Chat Interface
**Branch:** `4.5-i18n-chat`  
**Description:** Add language switching and localized UI  
**Goal:** Support seamless English/Chinese switching  
**Key Components/Files:**
- Language toggle component
- Localized system messages
- Chat placeholder text translations
- Language detection from user input
- Locale persistence

**Dependencies:** PR 4.3

---

### Phase 5: Dive Site Pages & SEO
**Goal**: Server-rendered dive site detail pages for organic discovery

#### PR 5.1: Dive Site Detail Page (SSR)
**Branch:** `5.1-site-detail-page`  
**Description:** Create server-rendered detail pages for each dive site  
**Goal:** Enable SEO-friendly dive site information pages  
**Key Components/Files:**
- `src/frontend/app/[locale]/sites/[slug]/page.tsx`
- Next.js dynamic route with `generateStaticParams`
- Server-side data fetching from backend API
- Structured data (JSON-LD) for rich snippets

**Dependencies:** PR 2.4, PR 1.5

#### PR 5.2: Dive Site List & Filtering Page
**Branch:** `5.2-site-list-page`  
**Description:** Build browsable dive site directory with filters  
**Goal:** Allow users to explore sites before starting chat  
**Key Components/Files:**
- `src/frontend/app/[locale]/sites/page.tsx`
- Filter UI: difficulty, region, depth range
- Pagination component
- Grid/list view toggle

**Dependencies:** PR 5.1

#### PR 5.3: Google Maps Integration
**Branch:** `5.3-maps-integration`  
**Description:** Embed interactive map showing dive site locations  
**Goal:** Visual location context on detail pages  
**Key Components/Files:**
- `src/frontend/components/Map/DiveSiteMap.tsx`
- Google Maps JavaScript API integration
- Custom markers for dive sites
- Info window with site preview

**Dependencies:** PR 5.1

#### PR 5.4: SEO Optimization (Metadata & Sitemap)
**Branch:** `5.4-seo-meta`  
**Description:** Generate dynamic metadata and sitemap.xml  
**Goal:** Maximize organic search visibility  
**Key Components/Files:**
- `src/frontend/app/[locale]/sites/[slug]/metadata.ts`
- OpenGraph tags generation
- Twitter Card metadata
- `src/frontend/app/sitemap.ts` (dynamic sitemap)
- `robots.txt` configuration

**Dependencies:** PR 5.1, PR 5.2

---

### Phase 6: Trip Planning & Export Features
**Goal**: Itinerary generation and export capabilities

#### PR 6.1: Itinerary Generation Agent
**Branch:** `6.1-itinerary-agent`  
**Description:** Create agent that builds multi-day trip plans  
**Goal:** Generate detailed day-by-day dive itineraries  
**Key Components/Files:**
- `src/backend/src/agents/itineraryAgent.ts`
- Prompts for itinerary structuring
- Dive safety constraints (max 3-4 dives/day, surface intervals)
- Cost estimation logic
- Accommodation/logistics suggestions

**Dependencies:** PR 3.6

#### PR 6.2: Trip Export API (PDF)
**Branch:** `6.2-export-api`  
**Description:** Generate PDF exports of trip itineraries  
**Goal:** Allow users to download/print trip plans  
**Key Components/Files:**
- `POST /api/v1/export/pdf`
- PDF generation library (Puppeteer or PDFKit)
- Trip itinerary template (HTML/CSS)
- Bilingual PDF support

**Dependencies:** PR 6.1

#### PR 6.3: Export UI Component
**Branch:** `6.3-export-ui`  
**Description:** Add export buttons to chat interface  
**Goal:** Enable one-click trip plan downloads  
**Key Components/Files:**
- `src/frontend/components/Chat/ExportButton.tsx`
- "Export Chat" (JSON/text)
- "Export Trip Plan" (PDF)
- Download trigger and file naming

**Dependencies:** PR 4.3, PR 6.2

---

### Phase 7: Polish, Testing & Deployment
**Goal**: Production readiness, monitoring, and launch preparation

#### PR 7.1: Error Handling & User Feedback
**Branch:** `7.1-error-handling`  
**Description:** Comprehensive error states and fallback UI  
**Goal:** Graceful degradation when services fail  
**Key Components/Files:**
- Global error boundaries (React)
- API error response standardization
- User-friendly error messages
- Retry logic for transient failures
- Offline state detection

**Dependencies:** All previous PRs

#### PR 7.2: Response Time Optimization & Timeout Mitigation
**Branch:** `7.2-performance-optimization`  
**Description:** Optimize RAG pipeline and API calls to stay under Vercel timeout limits  
**Goal:** Ensure 95%+ of chat responses complete within 10s (free tier) or 60s (Pro tier)  
**Key Components/Files:**
- Parallel execution of vector search + external API calls
- Neon connection pooling optimization (Prisma configuration)
- Response time middleware for logging slow queries
- Redis caching layer for frequent queries (optional)
- Vector search query optimization (limit chunks, tune HNSW params)
- Documentation of Vercel Pro plan requirement

**Dependencies:** PR 4.2, PR 3.6

#### PR 7.3: Rate Limiting & Security Hardening
**Branch:** `7.3-security`  
**Description:** Implement production security measures  
**Goal:** Prevent abuse and secure API endpoints  
**Key Components/Files:**
- Rate limiting middleware (express-rate-limit + Redis)
- Input sanitization (XSS prevention)
- CORS configuration
- Helmet.js security headers
- API key rotation strategy

**Dependencies:** PR 4.2

#### PR 7.4: Monitoring & Logging
**Branch:** `7.4-observability`  
**Description:** Add application monitoring and structured logging  
**Goal:** Track system health and debug production issues  
**Key Components/Files:**
- Structured logging (Winston/Pino)
- Error tracking (Sentry or similar)
- Performance metrics collection (response times, token usage)
- Health check endpoints (`/api/v1/health`)
- Database query performance logging
- Vercel Analytics integration

**Dependencies:** PR 1.4, PR 7.2

#### PR 7.5: End-to-End Testing
**Branch:** `7.5-e2e-tests`  
**Description:** Automated tests for critical user flows  
**Goal:** Ensure core functionality works end-to-end  
**Key Components/Files:**
- `src/tests/e2e/` (Playwright or Cypress)
- Test scenarios: chat flow, site browsing, export
- Bilingual testing (English + Chinese)
- Mobile responsiveness tests
- Timeout edge case testing

**Dependencies:** All Phase 4-6 PRs

#### PR 7.6: Production Deployment Configuration
**Branch:** `7.6-deployment`  
**Description:** Configure Vercel + Render deployment pipelines  
**Goal:** Automated staging and production deployments  
**Key Components/Files:**
- `vercel.json` (frontend deployment config with function timeout settings)
- Render YAML configuration (backend)
- Environment variable documentation
- Deployment runbook/checklist (includes Vercel Pro plan upgrade)
- Database migration strategy
- Performance budget documentation

**Dependencies:** PR 7.4

#### PR 7.7: Initial Dive Site Content (50 Sites)
**Branch:** `7.7-content-expansion`  
**Description:** Expand from 10 to 50 Malaysian dive sites  
**Goal:** Meet MVP requirement of 50+ dive sites  
**Key Components/Files:**
- Additional markdown files in `src/backend/data/dive-sites/`
- Sipadan, Mabul, Kapalai, Redang, Perhentian, etc.
- Dive operator contact information
- Photography and video assets

**Dependencies:** PR 2.1

---

## Implementation Sequence

**Sequential Phases** (must complete in order):
1. **Phase 1** → **Phase 2** → **Phase 3** → **Phase 4** → **Phase 5** → **Phase 6** → **Phase 7**

**Within each phase:**
- PRs can be parallelized if no dependencies exist
- Example: In Phase 3, PR 3.3, 3.4, 3.5 can be developed simultaneously after PR 3.2 is merged

**Critical Path**:
- Phase 1 (all PRs) → Phase 2 (all PRs) → Phase 3.1 → 3.2 → 3.6 → Phase 4.2 → 4.3 → Phase 7 (testing & deploy)

---

## Testing Strategy

**Per-PR Testing:**
- **Unit tests**: Each PR includes Jest tests for business logic (services, utilities)
- **Integration tests**: API endpoints tested with Supertest
- **Type safety**: TypeScript compilation must pass without errors

**Phase-Level Testing:**
- **Phase 2**: Verify vector search accuracy with sample queries
- **Phase 3**: Test agent responses with predefined user scenarios
- **Phase 4**: Manual QA of chat UX in browser
- **Phase 5**: SEO validation (Lighthouse, schema.org validator)
- **Phase 6**: PDF export rendering on multiple devices

**Pre-Launch:**
- **E2E tests** (PR 7.5): Cover 10+ critical user journeys
- **Load testing**: 100 concurrent users on chat endpoint
- **Bilingual validation**: Native speakers review both English and Chinese content

---

## Success Criteria

**Technical Completion:**
- ✅ All 7 phases merged to `main`
- ✅ 50+ Malaysian dive sites with bilingual content ingested
- ✅ Multi-agent RAG system responding within 5 seconds (90th percentile)
- ✅ Zero critical security vulnerabilities (Snyk/npm audit)
- ✅ 80%+ code coverage on backend services

**User Experience:**
- ✅ Chat interface loads in <2 seconds on 4G
- ✅ Users can complete first query within 30 seconds
- ✅ Session persistence works across page refreshes
- ✅ Bilingual switching works without context loss

**Business Readiness:**
- ✅ SEO: At least 20 dive site pages indexed by Google
- ✅ Legal: Safety disclaimers displayed and acknowledged
- ✅ Monitoring: Error tracking and logging operational
- ✅ Documentation: README, deployment guide, and API docs complete

---

## Known Constraints & Considerations

**Technical Constraints:**
- **No streaming**: MVP uses batch responses (no WebSocket/SSE) to simplify architecture
- **No authentication**: Guest-only access means limited personalization and no cross-device sync
- **Single database**: Neon Postgres handles both OLTP and vector search (may need separation at scale)
- **Google ADK maturity**: Framework is relatively new; may encounter undocumented edge cases

**Data Constraints:**
- **Manual curation**: 50 dive sites require significant content creation effort (bilingual)
- **Embedding costs**: Google text-embedding-004 API costs scale with content volume
- **Seasonal data staleness**: Weather patterns are historical averages, not real-time forecasts

**Business Considerations:**
- **No monetization**: MVP is free; future revenue model (ads, operator commissions) TBD
- **Operator partnerships**: No formal partnerships yet; contact info displayed as-is
- **Legal liability**: Disclaimers required but may not fully protect against dive accident claims

**Development Risks:**
- **9-week timeline**: Aggressive; assumes 2-3 developers working full-time
- **Google ADK learning curve**: Team may need ramp-up time on new framework
- **Bilingual QA**: Requires native Chinese speakers for content validation

**Deployment Gotchas:**
- **Vercel function timeouts**: Free tier has 10s limits, Pro tier ($20/mo) provides 60s; recommend upgrading to Pro for production
- **Performance optimization required**: Parallel API calls and response caching critical to stay under timeout limits (addressed in PR 7.2)
- **Neon cold starts**: Free tier may have connection latency on first request
- **pgvector index size**: HNSW indexes can grow large; monitor storage costs

---

**Plan Status:** ✅ Finalized  
**Created:** 30 November 2025  
**Next Steps:** Begin Phase 1 implementation with PR 1.1
