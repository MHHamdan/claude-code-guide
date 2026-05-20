# CLAUDE.md — Monorepo

## Structure
```
apps/
  api/          # Node.js / Fastify API service
  web/          # Next.js frontend
  worker/       # Background job processor
packages/
  shared/       # Shared types, utilities (consumed by all apps)
  ui/           # Shared component library (consumed by web)
  db/           # Drizzle schema + migrations (consumed by api, worker)
infra/          # Terraform / pulumi definitions
```

## Critical rules
- Changes to `packages/shared/` or `packages/db/` affect ALL apps — test across apps before marking complete
- Do not add app-level dependencies that should be in `packages/`
- Migrations in `packages/db/migrations/` are append-only — never edit applied migrations
- Run `pnpm -w build` from the repo root to verify the full build before completing

## Package manager
- pnpm workspaces — run commands from repo root with `-w` or from the specific package dir
- Adding a dependency: `pnpm --filter @repo/api add express` (scoped to that package)
- Never run bare `pnpm install` inside a package directory

## Testing
- `pnpm -w test` — run all tests across all packages
- `pnpm --filter @repo/api test` — run tests for a specific package
- Integration tests in each app require the shared DB to be running: `pnpm -w db:test-up`

## Module boundaries (enforce these strictly)
- `apps/*` may import from `packages/*` ✓
- `packages/*` may NOT import from `apps/*` ✗
- `packages/ui` may NOT import from `packages/db` ✗ (UI should not know about DB)
- Cross-app communication via API only — never direct imports between apps ✗

## What NOT to do
- Do not add logic to `packages/shared/` that belongs in a specific app
- Do not run `pnpm install` inside individual packages — always from root
- Do not create circular dependencies between packages (the build will catch this, but don't create them)
