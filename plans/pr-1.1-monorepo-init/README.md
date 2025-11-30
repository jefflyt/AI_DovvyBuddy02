# PR 1.1: Monorepo Initialization & Configuration

**Branch:** `1.1-monorepo-init`
**Steps:** 4 steps (COMPLEX)

## Folder Structure

Each step has a corresponding folder with detailed implementation guides:
- `1-root-workspace-setup/` — Root package.json, TypeScript base config, .gitignore
- `2-linting-formatting/` — ESLint and Prettier configuration
- `3-workspace-packages/` — Individual workspace setup (frontend, backend, shared)
- `4-vscode-docs/` — VS Code settings and documentation

## Quick Reference

**Goal:** Set up TypeScript monorepo foundation with npm workspaces

**Key Deliverables:**
- npm workspaces configuration for 3 packages
- TypeScript strict mode with `@shared/*` path aliases
- ESLint + Prettier with auto-fix/format on save
- Environment variable templates for frontend and backend
- VS Code workspace settings for consistent developer experience

**Testing After Each Step:**
1. After Step 1: `npm install` works, TypeScript installed
2. After Step 2: `npm run lint` and `npm run format:check` pass
3. After Step 3: Cross-workspace imports work, `npm run type-check` passes
4. After Step 4: VS Code auto-formats on save, README is complete

## Implementation Order

Follow the steps sequentially (1 → 2 → 3 → 4). Each step builds on the previous one and should be validated before moving forward.

After completion, open PR against `main`.
