# Session 05 — First Working Session End-to-End

## Context (from Sessions 01-04)

**Project**: Proliferate — open-source background agent platform, being commercialized as SaaS.
**Repo**: `c:/playground/try-proliferate/repo/`
**Knowledge base**: `c:/playground/try-proliferate/.claude/`

### Current State
- ✅ Full dev environment running (`pnpm dev:raw`)
- ✅ Web UI at http://localhost:3000
- ✅ Gateway on port 8787
- ✅ Docker postgres (5433) + redis (6379)
- ✅ DB migrations applied
- ✅ ANTHROPIC_API_KEY set in all 3 env files
- ✅ GitHub App `proliferate-iz` configured (App ID: 3037923)
- ✅ E2B template `proliferate-base` built (ID: vj1zaph7rttwwh9oe7ug)
- ✅ **ISSUE-010 FIXED**: Root cause was OpenCode one-time SQLite migration exceeding gateway's 58s timeout
  - `opencode-ready.ts` timeout increased to ~8 minutes (60 attempts × 8s)
  - `e2b.Dockerfile` updated with pre-warm step (migration baked into template image)
- ⚠️ **Dev server needs restart** to pick up the timeout fix
- ⚠️ **E2B template needs rebuild** to bake in the pre-warm Dockerfile change

### Start Command
```bash
cd c:/playground/try-proliferate/repo
export PATH="/c/Program Files/nodejs:$PATH"
docker compose up -d postgres redis
pnpm dev:raw > /tmp/proliferate-dev.log 2>&1 &
```

### Dev Logs
```bash
tail -c 8000 /tmp/proliferate-dev.log | tr -cd '[:print:]\n'
```

---

## Objective

Get the first complete end-to-end agent session working:
1. Restart dev server (picks up timeout fix)
2. Create account + org in browser (if not already done)
3. Install GitHub App on a test repo
4. Run a test session → get agent response
5. Rebuild E2B template with pre-warm baked in

---

## Step-by-Step Plan

### Step 1: Restart Dev Server
The `opencode-ready.ts` timeout increase requires a restart:
```bash
# Kill existing dev server (find node PIDs in Task Manager, or pkill)
cd c:/playground/try-proliferate/repo
docker compose up -d postgres redis
pnpm dev:raw > /tmp/proliferate-dev.log 2>&1 &
sleep 10
tail -c 2000 /tmp/proliferate-dev.log | tr -cd '[:print:]\n'
```

### Step 2: Verify Gateway Loaded New Code
Check that the gateway started with the updated timeout:
```bash
grep "opencode-ready\|maxAttempts\|Waiting for OpenCode" /tmp/proliferate-dev.log | head -5
```
(The gateway should auto-reload via tsx watch when files change)

### Step 3: Create Account + Org (Browser — user does this)
- Go to http://localhost:3000
- Sign up with email + password
- Create an organization

### Step 4: Install GitHub App + Connect Repo
- In the UI: Settings → Repos → Install GitHub App
- Install `proliferate-iz` on any small test repo (e.g., a personal repo)
- Confirm the repo appears in the Proliferate UI

### Step 5: Run Test Agent Session
- Click the repo → New Session
- Prompt: **"List all files in the root directory and tell me what type of project this is"**
- Watch the gateway logs for sandbox creation progress:
  ```bash
  tail -f /tmp/proliferate-dev.log | tr -cd '[:print:]\n'
  ```
- **First session may take 5-8 minutes** (OpenCode migration from old template)
- Expected progress in logs: "Sandbox created" → "OpenCode readiness probe" → "OpenCode ready" → "running"

### Step 6: Check If Session Works
Success criteria:
- Session status shows "running" in UI
- Agent responds with file listing
- No "Failed to initialize session" error

If it still fails, check the logs for the exact error and attempt count. If it's still timing out, the migration might be taking > 8 minutes — look at `last error` in the throw message.

### Step 7: Rebuild E2B Template (After Successful Session)
Bakes the pre-warm into the image so future cold starts are fast:
```bash
cd c:/playground/try-proliferate/repo
# Rebuild sandbox-daemon first (required before template build)
node packages/sandbox-daemon/build-daemon.mjs
cp packages/sandbox-daemon/dist/daemon.cjs packages/e2b-sandbox/sandbox-daemon.cjs

# Build the template
cd packages/e2b-sandbox
E2B_API_KEY=e2b_REDACTED \
E2B_TEMPLATE_ALIAS=proliferate-base \
npx tsx build.ts
```
Watch for: "OpenCode migration pre-warmed:" in the template build output — confirms the Dockerfile pre-warm step ran.

### Step 8: Test Second Session (Post-Rebuild)
Run another session to verify faster cold start (should be < 2 minutes).

---

## Troubleshooting

### If Still Timing Out
Check the error message carefully. The new error would say:
```
OpenCode not reachable after 60 attempts (fetch failed: ...)
```
If this appears, the migration is taking > 8 minutes. In that case:
- Check if the repo being cloned is very large (slow git clone → delays OpenCode start)
- Try with a tiny test repo (1-2 files)

### If Session Creates But Agent Doesn't Respond
- Check if ANTHROPIC_API_KEY is reaching OpenCode inside the sandbox
- Look for OpenCode logs: `sandbox.commands.run('tail -50 /tmp/opencode.log')`

### If GitHub App Install Fails
- Check `GITHUB_APP_PRIVATE_KEY` is correct in `.env.local`
- Verify `NEXT_PUBLIC_GITHUB_APP_SLUG=proliferate-iz`

---

## Deliverables

1. **First working session** — agent responds to a prompt ✅
2. **Rebuilt E2B template** — migration pre-warmed, faster cold starts
3. **Updated KNOWN_ISSUES.md** — close ISSUE-010 as resolved
4. **Updated SETUP_LOG.md** — append session 05 progress
5. **Updated NEXT_STEPS.md** — shift focus to revenue features
6. **Session summary**: `prompts/05 - CLAUDE_CODE_PROMPT_summary.md`
7. **Next prompt**: `prompts/06 - CLAUDE_CODE_PROMPT.md` (options in chat)

---

## Communication Style
- Short responses in chat
- All detail in `.claude/` files
- Surface blockers immediately with proposed fix
- When done: present 3 options for Session 06 in chat
