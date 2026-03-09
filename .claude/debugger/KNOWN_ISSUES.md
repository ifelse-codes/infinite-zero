# Known Issues

## Active

### ISSUE-001: Repo in Active Migration
**Severity**: Medium
**Status**: Monitor
**Description**: README states "active migration, may be unstable for the next couple of days" (as of 2026-03-08).
**Impact**: Dev environment may not run cleanly until migration completes.
**Action**: Attempt setup; document any failures here. Check repo for migration-related commits.

---

### ISSUE-002: NEXT_PUBLIC_ Vars Require Full Rebuild
**Severity**: Low (ops risk)
**Status**: Known limitation (documented)
**Description**: `NEXT_PUBLIC_` environment variables are baked into the Next.js client bundle at build time. Changing them in AWS Secrets Manager or K8s env at runtime has no effect.
**Impact**: Any client-facing config change requires a full image rebuild + deploy.
**Action**: Document all `NEXT_PUBLIC_` vars and their change procedure. Source of truth: `packages/environment/src/schema.ts`.

---

### ISSUE-003: Legacy ECS Infrastructure
**Severity**: Low (maintenance debt)
**Status**: Deferred cleanup
**Description**: `infra/pulumi/` and `infra/terraform/` contain legacy ECS infra alongside current EKS (`infra/pulumi-k8s/`).
**Impact**: Confusion about which infra is active. Extra maintenance surface.
**Action**: Confirm ECS is fully decommissioned before deleting. Add to cleanup backlog.

---

### ISSUE-004: Native Windows PostgreSQL Port Conflict
**Severity**: Medium (blocks DB migrations from host)
**Status**: Workaround applied
**Description**: Windows has `postgresql-x64-13` service running on port 5432 (PID 7672). Docker container also binds to 5432 — connection attempts from Node hit the native PG, which has different credentials.
**Impact**: `make db-migrate` and all direct DB connections from host fail with "password authentication failed".
**Resolution**: Created `docker-compose.override.yml` to remap Docker postgres to port 5433. Updated `.env` DATABASE_URL to `postgresql://postgres:postgres@localhost:5433/proliferate`.
**Note**: `docker-compose.override.yml` is gitignored (good — never commit). Must be recreated on fresh clone on this machine.

---

### ISSUE-005: Node.js 18 Incompatible — Upgrade Required
**Severity**: High (blocks app startup)
**Status**: Resolved
**Description**: Installed Node.js is v18.20.1. Next.js 16 requires >=20.9.0. Also, Node 18 lacks `--env-file` flag used by tsx watch scripts in gateway/worker/trigger-service.
**Impact**: `pnpm dev` fails immediately — `@proliferate/web#dev` exits with "For Next.js, Node.js version >=20.9.0 is required".
**Resolution needed**:
```
winget install OpenJS.NodeJS.LTS
# Then restart terminal and verify: node --version (should be 20.x or 22.x)
```
**Affects**: `apps/web`, `apps/gateway`, `apps/worker`, `apps/trigger-service`

---

### ISSUE-006: pnpm Version Mismatch (Non-blocking)
**Severity**: Low
**Status**: Non-blocking (install succeeded)
**Description**: Repo specifies `packageManager: pnpm@8.15.1` but installed version is 10.7.0. Corepack tries to fetch pnpm metadata and gets ETIMEDOUT but falls through.
**Impact**: Minor — `pnpm install` still succeeds. No functional difference observed.
**Resolution**: Either accept pnpm 10.7.0 or run `corepack enable && corepack prepare pnpm@8.15.1 --activate`.

---

### ISSUE-007: .env.local vs .env — tsx watch scripts
**Severity**: Medium (blocks gateway/worker/trigger-service)
**Status**: Resolved
**Description**: tsx watch scripts use `--env-file=../../.env.local` but only `.env` exists. Also, `--env-file` requires Node 20+.
**Resolution**: Created `.env.local` (copy of `.env`) at repo root for gateway/worker/trigger-service. Also created `apps/web/.env.local` since Next.js loads env files from the app directory, not monorepo root.
**Note**: Both `.env.local` files must be kept in sync when env changes. On fresh clone: `cp .env .env.local && cp .env apps/web/.env.local`

---

### ISSUE-008: Windows esbuild Banner Quoting
**Severity**: Low (build-time only)
**Status**: Workaround applied
**Description**: On Windows, `pnpm bundle` in `packages/sandbox-daemon` fails because `--banner:js='#!/usr/bin/env node'` is split by the shell at the space in `node --external:...`.
**Resolution**: Use esbuild JS API instead of CLI. Script: `packages/sandbox-daemon/build-daemon.mjs`. Run with `node build-daemon.mjs`.

---

### ISSUE-010: Gateway TLS Error — OpenCode Readiness Timeout
**Severity**: High (blocks all agent sessions)
**Status**: ✅ RESOLVED (Session 05)
**Root cause**: NOT a real TLS issue. OpenCode's one-time SQLite migration exceeded the gateway's 58s timeout.
**Resolution**:
1. `opencode-ready.ts`: maxAttempts 8→60 (~8 min max wait) — sufficient for first-run migration
2. `e2b.Dockerfile`: Pre-warm step bakes migration into template image (future fast cold starts)
3. First end-to-end agent session confirmed working — user generated a JSON beautifier app ✅

---

### ISSUE-009: sandbox-daemon.cjs Must Be Rebuilt on Fresh Clone
**Severity**: Medium (blocks E2B template build)
**Status**: Known — documented
**Description**: `packages/e2b-sandbox/sandbox-daemon.cjs` is gitignored. On any fresh clone, `pnpm build:template` will fail with "No files found in sandbox-daemon.cjs".
**Resolution**:
```bash
node packages/sandbox-daemon/build-daemon.mjs
cp packages/sandbox-daemon/dist/daemon.cjs packages/e2b-sandbox/sandbox-daemon.cjs
cd packages/e2b-sandbox && E2B_API_KEY=... E2B_TEMPLATE_ALIAS=proliferate-base npx tsx build.ts
```

---

## Resolved

### ISSUE-R003: Node.js 18 (resolved)
**Status**: Resolved
**Resolution**: Installed Node.js 24 LTS via `winget install OpenJS.NodeJS.LTS`.

### ISSUE-R004: Next.js .env location (resolved)
**Status**: Resolved
**Resolution**: Created `apps/web/.env.local` — Next.js in monorepo loads env from the app directory, not monorepo root.

### ISSUE-R005: pnpm dev:raw vs pnpm dev on Windows (resolved)
**Status**: Resolved
**Description**: `pnpm dev` uses `WATCHPACK_POLLING=true turbo run dev` which fails on Windows CMD (inline env var syntax).
**Resolution**: Use `pnpm dev:raw` instead, which runs `turbo run dev --concurrency 15` directly.

### ISSUE-R001: setup-env.sh on Windows
**Status**: Resolved
**Description**: Concerned setup-env.sh would fail on Windows bash.
**Resolution**: Ran cleanly via Git Bash. `openssl` available, `sed -i` works.

### ISSUE-R002: Docker context mismatch
**Status**: Resolved
**Description**: Docker context was set to `desktop-linux` (not running). `docker compose up` failed.
**Resolution**: `docker context use default` switched to Windows engine context. Containers start normally.
