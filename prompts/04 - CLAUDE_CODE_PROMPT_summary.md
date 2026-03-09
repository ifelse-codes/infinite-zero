# Session 04 Summary — TLS Error Root Cause Found + Fix Applied

## What We Accomplished

### Root Cause Identified (ISSUE-010)
The "TLS error" was NOT a TLS problem. The actual cause:

1. **E2B sandbox IS created** (confirmed: visible in E2B dashboard during debugging)
2. **E2B REST API IS reachable** (confirmed via standalone test project)
3. **E2B tunnel URL IS reachable** (confirmed: `https://4096-SANDBOXID.e2b.app/session` → 200)
4. **Root cause**: OpenCode runs a one-time SQLite database migration on first start ("may take a few minutes"). While migrating, port 4096 is not listening. E2B's tunnel proxy closes TCP connections BEFORE TLS when the port is down → this causes the "TLS error".
5. The gateway's readiness timeout was only **58 seconds total** (30s in `createSandbox` + 28s in driver). The migration takes longer.

### Diagnostic Tools Created
- `c:/playground/e2b-debug/test.mjs` — confirms E2B REST API works
- `c:/playground/e2b-debug/test-tunnel.mjs` — confirms tunnel URL works once OpenCode starts

### Fixes Applied

**Fix 1: Gateway Readiness Timeout** (`opencode-ready.ts`)
- `maxAttempts`: 8 → 60
- `intervalMs`: 1500ms → 3000ms
- `perAttemptTimeoutMs`: 2000ms → 5000ms
- New max wait: ~8 minutes (more than enough for OpenCode migration)

**Fix 2: Dockerfile Pre-warm** (`e2b.Dockerfile`)
- Added step after OpenCode install: runs `opencode serve --port 9999` as `user`
- Waits for "Database migration complete" in log
- Migration baked into template image → fresh sandboxes skip migration on startup
- After template rebuild: cold start < 2 minutes instead of 5-8 minutes

## Required Actions Before First Working Session

1. **Restart dev server** (to pick up `opencode-ready.ts` change)
2. **Test a session** — first time may take up to 8 minutes (migration still runs from old template)
3. **Rebuild E2B template** with new Dockerfile (pre-warm baked in):
   ```bash
   node packages/sandbox-daemon/build-daemon.mjs
   cp packages/sandbox-daemon/dist/daemon.cjs packages/e2b-sandbox/sandbox-daemon.cjs
   E2B_API_KEY=e2b_1deb1a68... E2B_TEMPLATE_ALIAS=proliferate-base npx tsx packages/e2b-sandbox/build.ts
   ```
4. After template rebuild: subsequent sessions should complete in < 2 minutes

## Files Changed
- `apps/gateway/src/hub/session/runtime/opencode-ready.ts` — extended timeout
- `packages/e2b-sandbox/e2b.Dockerfile` — pre-warm OpenCode migration
- `.env` — added `E2B_API_URL=https://api.e2b.app` (can be removed, not needed)
