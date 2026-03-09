# Git Workflow

## Branch Strategy
- `main` — production, always deployable
- `feat/<feature-name>` — new features
- `fix/<bug-name>` — bug fixes
- `chore/<task>` — maintenance, deps, infra

## Commit Messages (Conventional Commits)
```
feat: add Stripe billing integration
fix: resolve gateway WebSocket disconnect on session pause
chore: update pnpm lockfile
docs: update API documentation for sessions endpoint
```

## Before Committing Checklist
```bash
pnpm typecheck    # TypeScript errors
pnpm lint         # Biome + custom lint rules
pnpm test         # Vitest unit/integration
pnpm build        # Full build (optional, skip for speed in dev)
```

## Pull Requests
- Use `.github/PULL_REQUEST_TEMPLATE.md` — fill every section
- Link related spec file if subsystem changes
- Update `docs/specs/` in same PR if behavior changes
- Update `docs/specs/feature-registry.md` for new/changed/removed features

## Release Process
- Changesets for versioning (`pnpm changeset`)
- `changesets.yml` GitHub Action handles release pipeline
- Docker images published on `v*` tags via `docker-publish.yml`
