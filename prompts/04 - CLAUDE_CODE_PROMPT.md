# Session 04 — Debug TLS Error + First Working Session

## Context (from Sessions 01-03)

**Project**: Proliferate — open-source background agent platform, being commercialized as SaaS.
**Repo**: `c:/playground/try-proliferate/repo/`
**Knowledge base**: `c:/playground/try-proliferate/.claude/`

### Current State
- ✅ Full dev environment running (`pnpm dev:raw`)
- ✅ Web UI at http://localhost:3000 (sign-in works)
- ✅ Gateway on port 8787
- ✅ Docker postgres (5433) + redis (6379)
- ✅ DB migrations applied
- ✅ ANTHROPIC_API_KEY set in all 3 env files
- ✅ GitHub App `proliferate-iz` configured (App ID: 3037923)
- ✅ E2B template `proliferate-base` built (ID: vj1zaph7rttwwh9oe7ug)
- ❌ **BLOCKER**: "Failed to initialize session" — TLS error in gateway

### Start Command
```bash
cd c:/playground/try-proliferate/repo
export PATH="/c/Program Files/nodejs:$PATH"
docker compose up -d postgres redis
pnpm dev:raw > /tmp/proliferate-dev.log 2>&1 &
```

### Dev Logs
```bash
tail -c 8000 /tmp/proliferate-dev.log | tr -cd '[:print:]\n' | tail -50
```

---

## Active Blocker: TLS Error (ISSUE-010)

**Error in gateway logs**:
```
Failed to initialize session — fetch failed: Client network socket disconnected before secure TLS connection was established
```

**What's known**:
- Error happens in ~169ms (too fast for E2B sandbox creation — suggests connection is immediately dropped)
- E2B `Sandbox.create("proliferate-base")` works from command-line node tests
- Same E2B API key works from command line but fails from the gateway process
- E2B SDK v2.10.2 reads `E2B_DOMAIN` from `process.env` → builds `https://api.${domain}`
- Tried `E2B_DOMAIN=api.e2b.dev`, `E2B_DOMAIN=` (empty), `E2B_DOMAIN=e2b.app` → all fail

**Current env**:
- `E2B_API_KEY=e2b_REDACTED`
- `E2B_DOMAIN=e2b.app`
- `E2B_TEMPLATE=proliferate-base`

---

## Objective

Fix the TLS error and get one complete agent session running end-to-end.

---

## Step-by-Step Debug Plan

### 1. Try `E2B_API_URL` Direct Override
The E2B SDK reads `E2B_API_URL` env var as a direct URL override (bypasses domain logic entirely):
```
E2B_API_URL=https://api.e2b.app
```
Add to all 3 env files. Restart gateway. Test session.

### 2. Check for Proxy / TLS Env Vars
Check if Windows system proxy or Node TLS vars are affecting the gateway subprocess:
```bash
# In the gateway startup, print relevant env vars
echo $HTTPS_PROXY $HTTP_PROXY $NODE_TLS_REJECT_UNAUTHORIZED $NODE_EXTRA_CA_CERTS
```
If any proxy is set, add `NO_PROXY=*.e2b.app` to the env files.

### 3. Add Fetch URL Interceptor to Gateway
Temporarily add a global fetch interceptor to `apps/gateway/src/index.ts` to log the exact URL that's failing:
```ts
const origFetch = globalThis.fetch;
globalThis.fetch = async (input, init) => {
  const url = typeof input === 'string' ? input : (input as Request).url;
  console.log('[FETCH]', url);
  return origFetch(input, init);
};
```
This will reveal exactly which URL the E2B SDK is calling.

### 4. Check Windows Antivirus / Defender
Windows Defender or antivirus can intercept HTTPS from subprocess processes differently than interactive shells. Check if disabling real-time protection temporarily fixes the issue (just to confirm the cause).

### 5. Test with `undici` vs Native Fetch
The E2B SDK might be using Node's native `fetch` which behaves differently from `undici`. Try setting:
```
NODE_OPTIONS=--no-experimental-fetch
```
in the gateway to force it to use `undici` instead.

### 6. If TLS Unresolvable — Try Modal Sandbox
If E2B TLS cannot be resolved on Windows, switch to Modal as the sandbox provider:
```
DEFAULT_SANDBOX_PROVIDER=modal
```
Modal uses a Python FastAPI deployed separately. Check `packages/modal-sandbox/README.md` for setup.

---

## Deliverables

1. **TLS error fixed** — working gateway → E2B connection
2. **First end-to-end session** — prompt → sandbox boot → agent response
3. **Updated KNOWN_ISSUES.md** — resolve ISSUE-010 or document root cause
4. **Updated SETUP_LOG.md** — append session 04 progress
5. **Updated NEXT_STEPS.md** — reflect state after first working session
6. **Session summary**: `prompts/04 - CLAUDE_CODE_PROMPT_summary.md`
7. **Next prompt**: `prompts/05 - CLAUDE_CODE_PROMPT.md` (options in chat)

---

## Communication Style
- Short responses in chat
- All detail in `.claude/` files
- Surface blockers immediately with proposed fix
- When done: present 3 options for Session 05 in chat
