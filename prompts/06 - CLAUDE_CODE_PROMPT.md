# Session 06 — Architecture Deep-Dive + Multi-Agent Parallel Workflow Test

## Context (from Sessions 01-05)

**Project**: Proliferate — open-source background agent platform, being commercialized as SaaS.
**Repo**: `c:/playground/try-proliferate/repo/`
**Knowledge base**: `c:/playground/try-proliferate/.claude/`

### Current State
- ✅ Full dev environment running (`pnpm dev:raw`)
- ✅ Web UI at http://localhost:3000
- ✅ First end-to-end agent session worked (JSON beautifier app)
- ✅ ISSUE-010 closed (OpenCode timeout fix confirmed)
- ⚠️ E2B template rebuild needs one retry (ECONNRESET at step 20/28 — transient network drop)

---

## Objectives

### Part 1: Architecture Deep-Dive (Read-Only — No Code Changes)
Understand what Proliferate actually does, how it works technically, and what value it provides:
1. What problem does it solve?
2. How does a session flow end-to-end (user prompt → agent response)?
3. What are the key components and how do they interact?
4. Where is the real value / competitive moat?
5. What is currently missing to make it a compelling SaaS product?

**Output**: A clear plain-English summary in chat + updated `.claude/knowledgebase/ARCHITECTURE.md` with any gaps filled.

### Part 2: Multi-Agent Parallel Workflow Test
Test the platform's core value prop: multiple coding agents working in parallel on the same GitHub repo.

**Goal**: Take one of the user's existing GitHub repos and have **3 agents working simultaneously** on 3 different features, then merge the results.

**User will provide**: A GitHub repo URL + 3 feature requests

**What to set up**:
1. Install `proliferate-iz` GitHub App on the user's chosen repo
2. Create 3 sessions simultaneously (one per feature)
3. Monitor all 3 in parallel — watch them work concurrently
4. Review each agent's output (PR / file changes)
5. Assess merge strategy (do the branches conflict? can they be merged?)

---

## Step-by-Step Plan

### Step 0: Retry E2B Template Rebuild (Quick — 5 min)
Before starting, retry the template build so future sessions are fast:
```bash
cd /c/playground/try-proliferate/repo/packages/e2b-sandbox
E2B_API_KEY=e2b_REDACTED \
E2B_TEMPLATE_ALIAS=proliferate-base \
npx tsx build.ts 2>&1 | tee /tmp/e2b-build.log
```
Watch for: `OpenCode migration pre-warmed:` in output (confirms pre-warm ran).

### Step 1: Architecture Deep-Dive
Read these files in order:
1. `repo/docs/specs/sessions-gateway.md` — session lifecycle
2. `repo/docs/specs/auth-orgs.md` — auth + multi-tenancy
3. `repo/apps/gateway/src/` — core gateway logic
4. `repo/apps/worker/src/` — background job processing
5. `repo/packages/e2b-sandbox/` — sandbox abstraction layer
6. `repo/apps/web/src/` — web UI + RPC layer

Then write a clear architecture summary covering:
- What Proliferate does (1-paragraph plain English)
- Session lifecycle diagram (text-based)
- Key value props + competitive differentiation
- Current gaps vs a production SaaS

### Step 2: Identify the Right User Repo
Ask the user:
- Which GitHub repo to use for the parallel test?
- What 3 features should the 3 agents build?
  - Good choices: self-contained, unlikely to conflict (e.g., different files/subsystems)
  - Examples: "Add a README", "Add input validation to the login form", "Add a dark mode toggle"

### Step 3: Install GitHub App on the Test Repo
- In UI: Settings → Repos → Install `proliferate-iz` on the chosen repo
- Confirm repo appears in Proliferate UI

### Step 4: Launch 3 Sessions in Parallel
- Open 3 browser tabs / sessions simultaneously
- Each session = one feature
- Start all 3 within 30 seconds of each other
- Note start times and sandbox IDs from logs

### Step 5: Monitor Progress
Watch all 3 in the dev log:
```bash
tail -f /tmp/proliferate-dev.log | tr -cd '[:print:]\n' | grep -E "session|sandbox|error|ready|running"
```
Expected: 3 separate sandbox creation sequences running concurrently.

### Step 6: Review Results
For each of the 3 sessions:
- What did the agent produce?
- Did it create a PR or just file changes?
- Are the changes correct?
- Would they conflict if merged?

### Step 7: Assess Merge Strategy
- Can all 3 PRs be merged cleanly?
- If conflicts: why, and how would a real team resolve them?
- Document findings in `.claude/knowledgebase/` for future reference

---

## Success Criteria
- [ ] E2B template rebuilt with pre-warm (fast cold starts going forward)
- [ ] Clear architecture summary written (what Proliferate does + how)
- [ ] 3 agents ran simultaneously on the same repo
- [ ] All 3 produced output (code changes / PRs)
- [ ] Merge strategy assessed

---

## Deliverables
1. **Architecture summary** in chat + updated `.claude/knowledgebase/ARCHITECTURE.md`
2. **Multi-agent test results** — what each agent produced, merge assessment
3. **Updated KNOWN_ISSUES.md** — any new issues found
4. **Updated SETUP_LOG.md** — append session 06 progress
5. **Session summary**: `prompts/06 - CLAUDE_CODE_PROMPT_summary.md`
6. **Next prompt**: `prompts/07 - CLAUDE_CODE_PROMPT.md` (options in chat)

---

## Notes for Claude
- For Part 1: READ extensively before writing. Don't summarize from filenames — actually read the specs and source.
- For Part 2: The user will provide the repo + feature list. Wait for that input before creating sessions.
- The 3 features should be scoped to be completable in one agent session (~5-15 min each).
- If sessions are slow (old template), that's fine — monitor and document the timing.
