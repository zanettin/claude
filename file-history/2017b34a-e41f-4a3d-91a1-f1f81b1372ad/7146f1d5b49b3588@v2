# Plan: Audit & Update MEMORY.md

## Context
Current MEMORY.md has only 15 lines covering OneTrust CMP details and basic architecture facts. The codebase exploration revealed dozens of important patterns, conventions, and architectural details that I frequently need when making changes. Filling these gaps will reduce exploration time at the start of every conversation.

## Strategy
- Keep MEMORY.md under 200 lines (truncation limit) — concise bullet points only
- Move detailed reference material (orbit.config schema, lib catalog) to separate topic files
- Focus on **things I get wrong or have to re-discover** — not obvious things I can grep for

## Changes

### 1. Update `MEMORY.md` (~120 lines total)
Add these sections:

- **Core Stack** — Astro 5, Node 24, Tailwind 4, Alpine.js (not React for interactivity — React only for specific islands like boersenspiel), Fastify production server, Vite 6
- **Config System** — `orbit.config.ts` per app, `virtual:siteconf` module, env overrides via `envs.stage`/`envs.production` merged at runtime by `initEnvsMiddleware`, `APP_ENV` drives selection
- **Data Fetching** — Connector abstraction (`@dtc/connector`), dual CMS gateways (Drupal legacy + Ring Publishing migration), GraphQL with persisted queries to S3, Zod validation on all responses
- **Middleware Chain** — `sequence()` in each app's `src/middleware/index.ts`, order matters (basicAuth → initEnvs → audit → debug → i18n → preview → app-specific → edgeHeaders LAST)
- **Routing** — Catch-all `[...slug].astro`, dynamic rewrites for search/author paths, `/_/` → `/internal/` rewrite, API routes at `src/pages/api/`
- **Styling** — Tailwind 4 CSS-first config (no JS config), theme via `virtual:themes` Vite plugin, per-app `global.css` imports base config
- **i18n** — de/fr only, `libs/i18n/`, accessed via `Astro.locals.t()`, virtual module `virtual:i18n`
- **Testing** — Vitest unit (co-located `*.test.ts`), Playwright E2E (`apps/*/e2e/`), custom render util `@dtc/utils/src/test-renderer`, NX targets: `test:unit`, `test:e2e`
- **Deployment** — Docker (app + nginx containers), ECS Fargate, Akamai CDN, S3 for persisted queries + NX cache, GitHub Actions CI/CD
- **Key Libs** — One-liner for each lib in `/libs/` (link to detail file)
- **Gotchas** — Alpine.js uses `data-x-` prefix (not `x-`), `handleEdgeHeaders` must be last middleware, `virtual:siteconf` is the way to access config in components (not direct import of orbit.config.ts)

### 2. Create `libs-reference.md` (detail file)
Full table of all libs with package name, path, and description — too verbose for MEMORY.md but useful when I need to find the right import.

### 3. Create `orbit-config-schema.md` (detail file)
The full orbit.config.ts shape with all feature flags — critical reference when adding/modifying features.

## Files to modify
- `/Users/zar/.claude/projects/-Users-zar-development-rms-orbit/memory/MEMORY.md` — rewrite
- `/Users/zar/.claude/projects/-Users-zar-development-rms-orbit/memory/libs-reference.md` — create
- `/Users/zar/.claude/projects/-Users-zar-development-rms-orbit/memory/orbit-config-schema.md` — create

## Verification
- Confirm MEMORY.md stays under 200 lines
- Confirm all facts are accurate against actual source files
- Confirm no stale/wrong information carried over
