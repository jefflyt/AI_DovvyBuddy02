# PR 1.1: Monorepo Initialization & Configuration

**Branch:** `1.1-monorepo-init`
**Description:** Set up TypeScript monorepo with npm workspaces, ESLint, Prettier, environment variable templates, and VS Code workspace settings

## Goal

Establish foundational monorepo structure for DovvyDive with three workspaces (frontend, backend, shared) using npm workspaces. Configure TypeScript strict mode, ESLint/Prettier tooling, environment variable templates, and VS Code settings to enable type-safe, consistent development across the entire codebase.

## Why This Approach

npm workspaces (native to Node.js 20+) provides zero-config monorepo support with seamless type sharing, eliminating the need for external tools like Turborepo or Lerna. This approach maintains simplicity while enabling cross-workspace imports via TypeScript path aliases, ensuring frontend and backend can share types from the `shared` workspace without tight coupling.

## Implementation Steps

### Step 1: Root Configuration & Workspace Setup
**Folder:** `1-root-workspace-setup/`
**Files:** 
- `package.json` (root)
- `tsconfig.json` (root)
- `.gitignore`
- `.nvmrc`

**What:** Initialize root package.json with npm workspaces configuration pointing to `src/frontend`, `src/backend`, and `src/shared`. Create base TypeScript configuration with strict mode and path aliases (`@shared/*` mapping). Install shared dev dependencies (TypeScript, ESLint, Prettier, and related plugins) at root level. Add `.nvmrc` to enforce Node 20+ requirement.

**Testing:** 
- Run `npm install` and verify no errors
- Check that `node_modules` is created at root
- Verify TypeScript is installed: `npx tsc --version`

### Step 2: Linting & Formatting Configuration
**Folder:** `2-linting-formatting/`
**Files:**
- `.eslintrc.js`
- `.prettierrc.js`
- `package.json` (updated with lint/format scripts)

**What:** Configure ESLint with TypeScript parser, Next.js plugin for frontend overrides, and Node.js rules for backend. Set up Prettier with consistent formatting rules (2-space indent, single quotes, trailing commas). Integrate Prettier with ESLint via `eslint-config-prettier` to prevent conflicts. Add workspace-wide npm scripts: `lint`, `format`, `format:check`, `type-check`.

**Testing:**
- Run `npm run lint` (should pass with no files yet)
- Run `npm run format:check` (should pass)
- Verify no ESLint/Prettier conflicts by formatting a test file

### Step 3: Workspace Package Configuration
**Folder:** `3-workspace-packages/`
**Files:**
- `src/frontend/package.json`
- `src/frontend/tsconfig.json`
- `src/frontend/.env.local.example`
- `src/backend/package.json`
- `src/backend/tsconfig.json`
- `src/backend/.env.example`
- `src/shared/package.json`
- `src/shared/tsconfig.json`
- `src/shared/index.ts`
- `src/shared/types/index.ts`
- `src/shared/utils/index.ts`

**What:** Create individual package.json files for each workspace with appropriate names (`@dovvydive/frontend`, `@dovvydive/backend`, `@dovvydive/shared`). Create workspace-specific tsconfig.json files that extend the root config. Add environment variable templates for frontend (`.env.local.example`) and backend (`.env.example`) with documented placeholder values. Scaffold shared workspace with `types/` and `utils/` directories and barrel export files.

**Testing:**
- Run `npm install` and verify workspace linking works
- Run `npm run type-check` in each workspace (should pass)
- Create test type in `src/shared/types/example.ts` and import in `src/frontend` to verify cross-workspace imports
- Verify environment variable templates contain all required fields

### Step 4: VS Code Workspace Settings & Documentation
**Folder:** `4-vscode-docs/`
**Files:**
- `.vscode/settings.json`
- `.vscode/extensions.json`
- `README.md` (updated)

**What:** Create VS Code workspace settings with TypeScript IntelliSense, ESLint auto-fix on save, Prettier format on save, and workspace TypeScript version. Add recommended extensions file for ESLint and Prettier. Update root README.md with comprehensive setup instructions including Node version requirements, installation steps, available npm scripts, workspace structure explanation, and environment variable setup guide.

**Testing:**
- Open project in VS Code and verify settings apply
- Create a TypeScript file with intentional formatting issues, save, and verify auto-format works
- Create a file with ESLint errors and verify auto-fix on save
- Verify TypeScript IntelliSense shows cross-workspace type completion
- Follow README instructions from scratch to verify completeness

## Success Criteria

- ✅ `npm install` successfully installs all workspace dependencies with proper hoisting
- ✅ `npm run lint` passes across all workspaces
- ✅ `npm run format:check` passes
- ✅ `npm run type-check` compiles without errors in all three workspaces
- ✅ Cross-workspace imports work: frontend/backend can import from `@shared/*`
- ✅ Environment variable templates documented with all required fields
- ✅ VS Code auto-formats on save and shows TypeScript errors inline
- ✅ README provides clear setup instructions for new developers

## Commit Message

`feat(monorepo): initialize TypeScript monorepo with npm workspaces`

**Extended description:**
- Set up npm workspaces for frontend, backend, and shared packages
- Configure TypeScript strict mode with path aliases
- Add ESLint + Prettier with auto-fix and format-on-save
- Create environment variable templates
- Add VS Code workspace settings for consistent dev experience

---

## Architecture & Technology Stack

### Recommended Approach

**npm workspaces** for monorepo management (native to Node.js 20+, zero external dependencies). This provides:
- Native TypeScript support across workspaces
- Simplified dependency management
- Cross-workspace type sharing
- Compatible with Vercel and Render deployment
- Zero configuration overhead compared to pnpm/yarn/Turborepo
- Built-in to Node.js 20+ (no additional tooling required)

### Key Technologies

- **npm workspaces**: Native monorepo tooling (recommended for simplicity and native Node.js integration)
- **TypeScript 5.x**: Shared base configuration with workspace-specific overrides
- **ESLint 8.x**: Unified linting rules with Next.js and Node.js presets
- **Prettier 3.x**: Consistent code formatting across all workspaces

### High-Level Architecture

```
DovvyDive/
├── package.json (root - workspace orchestrator)
├── tsconfig.json (base config)
├── .eslintrc.js (shared rules)
├── .prettierrc.js (formatting)
├── .vscode/
│   └── settings.json (workspace settings)
├── src/
│   ├── frontend/
│   │   ├── package.json
│   │   ├── tsconfig.json (extends ../../tsconfig.json)
│   │   └── .env.local.example
│   ├── backend/
│   │   ├── package.json
│   │   ├── tsconfig.json (extends ../../tsconfig.json)
│   │   └── .env.example
│   └── shared/
│       ├── package.json
│       └── tsconfig.json (extends ../../tsconfig.json)
```

---

## Implementation Plan

### Scope & Objectives

**Primary Goal**: Create a working TypeScript monorepo with proper workspace isolation, shared tooling configuration, environment variable templates, and VS Code workspace settings.

**User Impact**: Enables developers to run type-safe code across frontend/backend with consistent linting and formatting. Establishes patterns for cross-workspace imports. VS Code will automatically apply correct settings for all developers.

**Success Criteria**:
- ✅ `npm install` successfully installs all workspace dependencies
- ✅ TypeScript compiles without errors in all workspaces
- ✅ ESLint runs without errors
- ✅ Prettier formatting is consistent
- ✅ Cross-workspace imports work (frontend can import from shared)
- ✅ Environment variable templates exist for both frontend and backend
- ✅ VS Code workspace settings configure TypeScript, ESLint, and Prettier integration

### Key Work Items

1. **Root package.json with workspace configuration**
   - Define workspaces: `src/frontend`, `src/backend`, `src/shared`
   - Install shared dev dependencies (TypeScript, ESLint, Prettier)
   - Add npm scripts for workspace-wide commands (lint, format, type-check)

2. **TypeScript base configuration**
   - Create root `tsconfig.json` with strict mode
   - Configure path aliases for cross-workspace imports (`@shared/*`)
   - Create workspace-specific tsconfig files that extend the base

3. **ESLint + Prettier setup**
   - Configure ESLint with TypeScript parser
   - Add Next.js plugin for frontend workspace
   - Add Node.js rules for backend workspace
   - Configure Prettier with consistent formatting rules
   - Integrate Prettier with ESLint (eslint-config-prettier)

4. **Workspace package.json files**
   - Create minimal package.json for frontend, backend, shared
   - Define workspace-specific dependencies
   - Add build/dev scripts placeholders

5. **Environment variable templates**
   - Create `.env.example` for backend (DATABASE_URL, GOOGLE_API_KEY, JWT_SECRET, etc.)
   - Create `.env.local.example` for frontend (NEXT_PUBLIC_API_URL, etc.)
   - Document all required environment variables with descriptions

6. **Shared utilities scaffold**
   - Create `src/shared/types/` directory
   - Create `src/shared/utils/` directory
   - Add initial TypeScript type definition file (demonstrates pattern)
   - Create `src/shared/index.ts` as main export entry point

7. **VS Code workspace settings**
   - Create `.vscode/settings.json` with TypeScript, ESLint, Prettier integration
   - Configure auto-format on save
   - Set TypeScript version to workspace version
   - Configure ESLint to auto-fix on save

### Single PR Summary

**PR Name:** Monorepo Initialization & Configuration  
**Branch:** `1.1-monorepo-init`  
**Description:** Set up TypeScript monorepo with npm workspaces, ESLint, Prettier, environment variable templates, and VS Code workspace settings

**Goal:** Establish foundational project structure with development tooling that enables type-safe development across frontend, backend, and shared workspaces

**Key Components/Files:**
- `package.json` (root workspace orchestrator)
- `tsconfig.json` (base + workspace-specific configs)
- `.eslintrc.js` + `.prettierrc.js`
- `.vscode/settings.json` (workspace configuration)
- `src/frontend/package.json` + `tsconfig.json` + `.env.local.example`
- `src/backend/package.json` + `tsconfig.json` + `.env.example`
- `src/shared/package.json` + `tsconfig.json` + `index.ts`
- `.gitignore` (node_modules, .env, dist/, .next/, build/)

**Dependencies:** None (first PR in Phase 1)

**Tech Details:**
- **Package Manager**: npm workspaces (native to Node 20+, no additional tools)
- **TypeScript**: Strict mode enabled with path aliases (`@shared/*` → `src/shared/*`)
- **ESLint**: Extends `next/core-web-vitals` (frontend), `eslint:recommended` + `@typescript-eslint/recommended` (backend)
- **Prettier**: Integrated via `eslint-config-prettier` to avoid conflicts
- **VS Code**: Auto-format on save, TypeScript IntelliSense, ESLint auto-fix

**Testing Approach:**
- Run `npm install` at root to verify workspace resolution
- Run `npm run lint` to verify ESLint configuration across all workspaces
- Run `npm run format:check` to verify Prettier
- Run `npm run type-check` to verify TypeScript compilation in all workspaces
- Create a simple shared type and import it in frontend to verify cross-workspace imports
- Open project in VS Code and verify settings apply correctly

---

## Implementation Sequence

1. **Create root package.json** with workspaces array and shared dev dependencies
2. **Install dev dependencies** at root: `typescript`, `eslint`, `prettier`, `@typescript-eslint/parser`, `@typescript-eslint/eslint-plugin`, `eslint-config-prettier`
3. **Create TypeScript base configuration** (`tsconfig.json`) with strict mode and path aliases
4. **Set up ESLint configuration** (`.eslintrc.js`) with TypeScript support
5. **Set up Prettier configuration** (`.prettierrc.js`) with formatting rules
6. **Create workspace package.json files** for `src/frontend`, `src/backend`, `src/shared`
7. **Create workspace-specific tsconfig.json** files extending the base
8. **Create environment variable templates** (`.env.example`, `.env.local.example`)
9. **Scaffold shared types/utils** (`src/shared/types/`, `src/shared/utils/`, `index.ts`)
10. **Create VS Code workspace settings** (`.vscode/settings.json`)
11. **Update .gitignore** to exclude node_modules, env files, build outputs
12. **Update root README.md** with setup instructions
13. **Test the setup** by running lint, format, and type checking commands

---

## Testing Strategy

**Manual Validation:**
- `npm install` completes without errors and hoists shared dependencies
- `npm run lint` passes in all workspaces
- `npm run format:check` passes across all files
- `npm run type-check` succeeds in all workspaces
- Create a test type in `src/shared/types/example.ts` and import it in a frontend file to verify cross-workspace imports work
- Open project in VS Code and verify TypeScript IntelliSense works across workspaces

**Automated Tests:**
- Not applicable for this PR (infrastructure only)
- CI/CD pipeline setup will come in later PRs

**Verification Checklist:**
- [ ] All workspace dependencies install correctly
- [ ] No TypeScript errors when running `npm run type-check`
- [ ] ESLint and Prettier rules don't conflict
- [ ] Cross-workspace imports resolve correctly (e.g., `import type { Example } from '@shared/types'`)
- [ ] Environment variable templates are complete and documented
- [ ] VS Code auto-formats files on save
- [ ] VS Code shows TypeScript errors inline

---

## Success Criteria

**Technical Completion:**
- ✅ Monorepo structure established with 3 workspaces (frontend, backend, shared)
- ✅ TypeScript configuration working across all workspaces
- ✅ ESLint and Prettier configured and passing
- ✅ Environment variable templates documented with all required variables
- ✅ Cross-workspace imports functional using `@shared` alias
- ✅ VS Code workspace settings applied and working

**Developer Experience:**
- ✅ Single `npm install` command sets up entire project
- ✅ IDE (VS Code) recognizes TypeScript types across workspaces
- ✅ Linting runs automatically in editor with auto-fix on save
- ✅ Prettier auto-formats code on save
- ✅ Clear documentation in README for local setup

**Documentation:**
- ✅ Update root README.md with:
  - Node.js version requirement (20+)
  - Setup instructions (`npm install`)
  - Available npm scripts (`lint`, `format`, `type-check`)
  - Workspace structure explanation
  - Environment variable setup guide
- ✅ Document workspace import patterns
- ✅ List all required environment variables with descriptions

---

## Known Constraints & Considerations

**Technical Constraints:**
- **npm workspaces require Node 20+**: Ensure developers have correct Node version (document in README and consider adding `.nvmrc`)
- **Next.js 14 App Router**: Must use compatible ESLint plugin (`eslint-config-next`)
- **No build outputs yet**: This PR doesn't include build scripts (added in PR 1.4 and 1.5)

**Workspace Dependencies:**
- Frontend and backend should NOT depend on each other (maintain clear separation)
- Only `shared` can be imported by both frontend and backend
- Avoid circular dependencies between workspaces

**Environment Variables:**
- Frontend uses `NEXT_PUBLIC_*` prefix for client-side accessible vars
- Backend uses standard naming (no prefix, server-side only)
- Never commit actual `.env` files (only `.env.example` templates)
- Document each variable with purpose and example value

**Tooling Decisions:**
- **Why npm workspaces over pnpm/yarn**: 
  - Native to Node.js 20+ (zero external dependencies)
  - Simpler learning curve for team
  - Vercel and Render deployment compatibility out-of-the-box
  - No lockfile format confusion (package-lock.json is standard)
- **Why not Turborepo**: 
  - Overkill for MVP (only 3 workspaces)
  - Can add later for build caching if needed
  - Adds complexity without clear MVP benefit
- **Path aliases**: Using `@shared` prefix for clarity; configured in tsconfig `paths` and referenced in ESLint/TypeScript resolvers

**Gotchas:**
- ESLint Next.js plugin may conflict with general TypeScript rules; use `overrides` in `.eslintrc.js` to apply Next.js rules only to `src/frontend`
- Prettier and ESLint can conflict on formatting; always use `eslint-config-prettier` as the LAST item in `extends` array to disable ESLint formatting rules
- Cross-workspace imports require BOTH TypeScript `paths` mapping AND npm workspace resolution to work correctly
- VS Code may need to be reloaded after initial setup for TypeScript to recognize workspace paths

**Required Environment Variables (Initial Set):**

Backend (`.env.example`):
```
DATABASE_URL=postgresql://user:password@localhost:5432/dovvydive
GOOGLE_API_KEY=your_google_api_key
GOOGLE_MAPS_KEY=your_google_maps_key
WEATHER_API_KEY=your_weather_api_key
JWT_SECRET=your_jwt_secret_for_session_signing
NODE_ENV=development
PORT=3001
```

Frontend (`.env.local.example`):
```
NEXT_PUBLIC_API_URL=http://localhost:3001/api/v1
NEXT_PUBLIC_GOOGLE_MAPS_KEY=your_google_maps_key
```

---

**Plan Status:** ✅ Finalized  
**Created:** 1 December 2025  
**Next Steps:** Begin implementation of PR 1.1 on branch `1.1-monorepo-init`
