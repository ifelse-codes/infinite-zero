# Coding Standards

*Inherited from repo CLAUDE.md + project conventions*

## Language
- **TypeScript everywhere** — no plain JS files in apps/packages
- Strict mode enabled
- Run `pnpm typecheck` before committing

## Formatting
- **Tabs** for indentation (Biome enforced)
- **Semicolons** required
- Run `biome check` before committing (`pnpm lint:fix` to auto-fix)

## Naming
- `camelCase` — variables, functions
- `PascalCase` — types, interfaces, React components, classes
- Explicit names that carry meaning, no abbreviations
- Hook files: `kebab-case` (e.g., `use-repos.ts`)

## File Organization
- Web utilities used only in web: `apps/web/src/lib/`
- Shared pure helpers/types: `packages/shared/`
- Backend/business logic: `packages/services/`
- **All DB reads/writes in `packages/services/src/**/db.ts`**
- Do NOT import `@proliferate/db` directly outside services package

## Frontend Rules
- No `fetch("/api/...")` in components — use oRPC via TanStack Query
- WebSocket: use `@proliferate/gateway-clients` only
- UI: shadcn/ui + Tailwind semantic tokens only
- No raw Tailwind palette colors outside `components/ui/` (use `text-foreground`, `bg-card`, etc.)
- No native `alert/confirm/prompt`
- No raw HTML form elements in pages/features — use shadcn/ui primitives

## Backend Rules
- API routes are thin wrappers — business logic goes in `packages/services`
- Throw errors, don't return `{ ok: false }` objects
- Drizzle only — no raw SQL unless absolutely necessary

## Logging
- Use `@proliferate/logger` everywhere
- JSON structured logs in Node services
- No new `console.*` in gateway/worker/trigger-service/services
- Never log secrets, tokens, authorization headers, or full request bodies

## Testing
- Vitest for unit + integration tests
- Test pure functions + API handlers
- Skip unit tests for React UI (manual review)
- Run `pnpm test` before committing

## Dependencies
- MIT / Apache-2.0 / BSD only — **no GPL, AGPL, SSPL, BSL**
- Ask before adding any new dependency

## Security
- Never commit secrets, `.env` files, credentials, or internal URLs
- All config via environment variables
- Source of truth for env keys: `packages/environment/src/schema.ts`
