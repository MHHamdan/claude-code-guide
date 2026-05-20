# CLAUDE.md — Node.js / TypeScript Service

## Project overview
<!-- Describe what this service does, who calls it, what it depends on -->

## Environment
- Node.js 22 LTS
- Package manager: pnpm (use `pnpm` not `npm` or `yarn`)
- Run locally: `pnpm dev` (tsx watch, hot reload)
- Run tests: `pnpm test` (vitest)
- Lint + typecheck: `pnpm lint` (eslint + tsc --noEmit) — must pass before task complete
- Build: `pnpm build` (tsc → dist/)

## Critical rules
- TypeScript strict mode is on — no `any`, no `// @ts-ignore` without explicit justification
- All async functions must handle errors — never let unhandled rejections bubble
- Do not modify `drizzle/migrations/` — those files have been applied to production
- Environment variables accessed only via `src/config.ts` — never `process.env` directly
- New routes require an integration test in `test/api/`

## Code style
- ESLint + Prettier enforced — do not fight the formatter
- Explicit return types on exported functions
- Prefer `const` over `let`; never `var`
- Error handling: `Result<T, E>` pattern (see `src/types/result.ts`) — no thrown exceptions in business logic
- Avoid `Promise.all` when failures of one should not cancel others — use `Promise.allSettled`

## Architecture
```
src/
  routes/       # Fastify route handlers (thin — delegate to services)
  services/     # Business logic (no HTTP context, no DB calls — call repositories)
  repositories/ # All DB queries (Drizzle ORM) — no business logic
  middleware/   # Auth, rate limiting, request validation
  config.ts     # Zod-validated env config (single source of truth)
  types/        # Shared types and Result<T,E> pattern
test/
  api/          # Integration tests against real DB (testcontainers)
  unit/         # Pure unit tests (no I/O)
drizzle/
  migrations/   # READ ONLY — use `pnpm db:generate` to create new ones
  schema.ts     # Source of truth for DB schema
```

## Database
- ORM: Drizzle (never raw SQL unless explicitly asked)
- Adding a column: update `drizzle/schema.ts`, then run `pnpm db:generate` to generate migration
- Never run `pnpm db:push` in production — use the generated migration files

## What NOT to do
- Do not use `require()` — ESM only
- Do not use `console.log` in production code — use the structured logger (`src/logger.ts`)
- Do not add `express` or `koa` — we use Fastify
- Do not disable TypeScript checks to make something compile
- Do not commit `.env` or any credentials
- Do not use `setTimeout` for retry logic — use the `retry` utility in `src/utils/retry.ts`
