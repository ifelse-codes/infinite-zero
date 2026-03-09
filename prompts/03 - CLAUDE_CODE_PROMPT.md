# Session 03 — First Test Run + GitHub App Setup

## Context (from Sessions 01-02)

**Project**: Proliferate — open-source background agent platform, being commercialized as SaaS.
**Repo**: `c:/playground/try-proliferate/repo/`
**Knowledge base**: `c:/playground/try-proliferate/.claude/`

### Current State
- ✅ Full dev environment running
- ✅ Web UI at http://localhost:3000 (sign-in works)
- ✅ Gateway on port 8787
- ✅ Docker postgres (5433) + redis (6379)
- ✅ DB migrations applied
- ❌ GitHub App not configured (needed for repo connections)
- ❌ ANTHROPIC_API_KEY not set (needed for agent sessions)
- ❌ No account created yet

### Start Command
```bash
cd c:/playground/try-proliferate/repo
export PATH="/c/Program Files/nodejs:$PATH"
docker compose up -d postgres redis
pnpm dev:raw
```

### Windows-Specific Notes
- Postgres is on **port 5433** (not 5432 — conflict with native PG 13 service)
- Node 24 LTS installed at `C:\Program Files\nodejs\`
- 3 env files must be kept in sync: `.env`, `.env.local`, `apps/web/.env.local`
- Use `pnpm dev:raw` not `pnpm dev`

---

## Objective

Get a test agent session running end-to-end:
1. Create account and org in the UI
2. Set ANTHROPIC_API_KEY in env
3. Configure GitHub App and connect a repo
4. Run a test prompt in a session
5. Verify the E2B sandbox spins up and the agent responds

---

## Step-by-Step Instructions

### 1. Set ANTHROPIC_API_KEY
Add to all 3 env files (`.env`, `.env.local`, `apps/web/.env.local`):
```
ANTHROPIC_API_KEY=<key from console.anthropic.com>
```
Ask user for their Anthropic API key if not provided.

### 2. Create Account in UI
- Navigate to http://localhost:3000
- Sign up with email + password
- Create an organization (e.g., "test-org")
- Verify the dashboard loads

### 3. Configure GitHub App
Read the GitHub App setup instructions in `repo/README.md` Step 2.
The setup requires creating a GitHub App with specific permissions and setting:
```
NEXT_PUBLIC_GITHUB_APP_SLUG=<your-app-slug>
GITHUB_APP_ID=<app-id>
GITHUB_APP_PRIVATE_KEY=<pem-key>
GITHUB_APP_CLIENT_ID=<client-id>
GITHUB_APP_CLIENT_SECRET=<client-secret>
GITHUB_APP_WEBHOOK_SECRET=<already-set>
```
Set in all 3 env files, then restart dev server.

### 4. Connect a GitHub Repo
- In the UI, go to Repos section
- Install the GitHub App on a test repo (any small public or private repo)
- Verify the repo appears in the dashboard

### 5. Run a Test Session
- Create a new agent session on the connected repo
- Use a simple test prompt: "List the files in the root directory"
- Watch the session stream in the UI
- Verify the E2B sandbox spins up (check logs for E2B connection)

### 6. Verify E2B Integration
Check dev logs for:
```
E2B sandbox created
Sandbox ID: ...
```
If the sandbox fails, check:
- `E2B_API_KEY` is set correctly
- `E2B_TEMPLATE=proliferate-base` (the template may need to exist in E2B dashboard)

---

## Deliverables

1. **Working agent session** — end-to-end prompt → sandbox → response
2. **Updated KNOWN_ISSUES.md** — log any new issues found
3. **Updated SETUP_LOG.md** — append session 03 progress
4. **Updated NEXT_STEPS.md** — reflect state after first test run
5. **Session summary**: `prompts/03 - CLAUDE_CODE_PROMPT_summary.md`
6. **Next prompt**: `prompts/04 - CLAUDE_CODE_PROMPT.md` (options in chat)

---

## If Blockers Arise

- **No Anthropic key available**: Skip agent session, verify UI/auth flows only
- **GitHub App creation is blocked**: Test the UI and session creation UI without connecting a repo
- **E2B sandbox fails**: Check template name — may need to create `proliferate-base` template in E2B dashboard, or use a different template name
- **Session errors**: Check `docs/specs/sessions-gateway.md` for session lifecycle

---

## Communication Style
- Short responses in chat
- All detail in `.claude/` files
- Surface blockers immediately with proposed fix
- When done: present 3 options for Session 04 in chat
