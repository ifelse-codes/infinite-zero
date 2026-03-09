# Session 07 — Complete the Multi-Agent Parallel Test

## Context (from Sessions 01-06)

**Project**: Proliferate — open-source background agent platform, being commercialized as SaaS.
**Repo**: `c:/playground/try-proliferate/repo/`
**Knowledge base**: `c:/playground/try-proliferate/.claude/`

### Current State
- ✅ Full dev environment running (`pnpm dev:raw`)
- ✅ Web UI at http://localhost:3000
- ✅ First end-to-end agent session worked (Session 05)
- ✅ ISSUE-010 closed (OpenCode timeout fix confirmed)
- ✅ E2B template rebuilt (Session 06 — 29s, all 28 layers cached, pre-warm confirmed)
- ✅ Architecture fully understood (session creator, wake cycle, admission guard)
- ⏳ Multi-agent parallel test scoped but not launched yet

### Target Repo for Parallel Test
- **URL**: https://github.com/ifelse-codes/spring-open-telemetry
- **Feature 1**: Working logs demo
- **Feature 2**: Working metrics demo
- **Feature 3**: Working traces demo

---

## Objectives

### Step 1: Understand the Target Repo
Before launching sessions, read the target repo to understand its structure:
- What does `spring-open-telemetry` do?
- What files/modules are relevant to logs, metrics, and traces?
- Are these 3 features genuinely isolated (different files/subsystems)?
- Are there any existing stubs or TODOs for these?

Use WebFetch or WebSearch to read the repo structure at https://github.com/ifelse-codes/spring-open-telemetry

Write a brief assessment:
- What each agent will need to do
- Confirm the 3 features are non-conflicting (different files expected)
- Recommended prompt for each session

### Step 2: Install GitHub App on Target Repo
**User action required** — I (Claude) cannot do this in a browser.

Walk the user through:
1. Go to http://localhost:3000 → Settings (top right) → Repositories
2. Click "Install GitHub App" or "Add Repository"
3. Install `proliferate-iz` on `ifelse-codes/spring-open-telemetry`
4. Confirm the repo appears in Proliferate UI before proceeding

### Step 3: Craft the 3 Session Prompts
Based on the repo analysis from Step 1, write the exact prompts for each session.
Good prompts are:
- Self-contained (no cross-feature dependencies)
- Specific enough to produce runnable code in 5-15 min
- Clear about what "done" looks like

### Step 4: Launch 3 Sessions Simultaneously
**User action required** — walk the user through:
1. Open 3 browser tabs: http://localhost:3000
2. Navigate to the spring-open-telemetry repo in each tab
3. Start Session 1 (logs) in tab 1
4. Immediately start Session 2 (metrics) in tab 2
5. Immediately start Session 3 (traces) in tab 3
6. All 3 should be started within 30 seconds

Note the time each session was started and the sandbox IDs from the dev log.

### Step 5: Monitor All 3 in Parallel
Watch the dev log for all 3 sessions running concurrently:
```bash
tail -f /tmp/proliferate-dev.log | tr -cd '[:print:]\n' | grep -E "session|sandbox|error|ready|running"
```

Or just watch the Proliferate UI — all 3 sessions should show as active simultaneously.

Expected: 3 separate sandbox creation sequences running concurrently.

Document:
- Start times for all 3
- When each sandbox became ready
- Whether all 3 ran in parallel or sequentially

### Step 6: Review Each Agent's Output
For each session (logs / metrics / traces):
- What files did the agent create or modify?
- Does the code look correct?
- Did the agent open a PR?
- Does the PR contain only the expected changes (no accidental cross-feature changes)?

### Step 7: Assess Merge Strategy
- Can all 3 PRs be merged cleanly into main?
- If conflicts: identify exactly which files conflict and why
- Recommend a merge order if needed

---

## Success Criteria
- [ ] Target repo understood (structure analyzed before launching)
- [ ] GitHub App installed on `ifelse-codes/spring-open-telemetry`
- [ ] 3 sessions launched simultaneously (within 30s of each other)
- [ ] All 3 ran in parallel (confirmed from logs or UI)
- [ ] All 3 produced output (code changes, ideally PRs)
- [ ] Merge strategy assessed (clean or conflict analysis)

---

## Deliverables
1. **Repo analysis** — what each feature needs, are they non-conflicting
2. **Session prompts** — exact prompts used for each of the 3 sessions
3. **Parallel test results** — timing, what each agent produced, merge assessment
4. **Updated KNOWN_ISSUES.md** — any new issues found
5. **Updated SETUP_LOG.md** — append session 07 progress
6. **Session summary**: `prompts/07 - CLAUDE_CODE_PROMPT_summary.md`
7. **Next prompt**: `prompts/08 - CLAUDE_CODE_PROMPT.md` (options in chat)

---

## Notes for Claude
- Read the target repo FIRST before crafting prompts — don't guess the structure
- The 3 features (logs, metrics, traces) are likely in different Spring Boot modules/configs — confirm before launching
- If the GitHub App install isn't done yet, pause and walk the user through it — don't skip this
- Document exact timings from the dev log for the parallel execution proof
- If sessions are slow (cold start), that's still valuable data — note the timing
