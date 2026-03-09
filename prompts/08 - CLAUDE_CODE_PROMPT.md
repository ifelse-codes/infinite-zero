# Session 08 — Run the Multi-Agent Parallel Test

## Context (from Sessions 01-07)

**Project**: Proliferate — open-source background agent platform, being commercialized as SaaS.
**Repo**: `c:/playground/try-proliferate/repo/`
**Knowledge base**: `c:/playground/try-proliferate/.claude/`

### Current State
- ✅ Full dev environment running (`pnpm dev:raw`)
- ✅ Web UI at http://localhost:3000
- ✅ First end-to-end agent session worked (Session 05)
- ✅ Architecture fully understood (session creator, wake cycle, admission guard)
- ✅ E2B template rebuilt with pre-warm (Session 06)
- ✅ Target repo analyzed (`ifelse-codes/spring-open-telemetry`) — Session 07
- ✅ 3 session prompts written and ready (logs, metrics, traces) — Session 07
- ✅ ANTHROPIC_API_KEY rotated (new key in all 3 env files)
- ✅ Knowledge base pushed to `ifelse-codes/infinite-zero`
- ✅ PR #304 opened to proliferate upstream
- ⏳ GitHub App install on `spring-open-telemetry` — NOT YET DONE
- ⏳ 3 parallel sessions — NOT YET LAUNCHED

---

## Objectives

### Step 1: Start Dev Server
Ensure everything is running before launching sessions:
```bash
cd c:/playground/try-proliferate/repo
export PATH="/c/Program Files/nodejs:$PATH"
docker compose up -d postgres redis
pnpm dev:raw
```
Confirm http://localhost:3000 loads and http://localhost:8787 is listening.

### Step 2: Install GitHub App (User Action Required)
Walk the user through:
1. Go to **http://localhost:3000 → Settings → Repositories**
2. Click **"Install GitHub App"** → select `ifelse-codes/spring-open-telemetry`
3. Confirm the repo appears in Proliferate UI before proceeding

Do NOT proceed to Step 3 until confirmed.

### Step 3: Launch 3 Sessions Simultaneously (User Action Required)

Open **3 browser tabs** at http://localhost:3000, navigate to `spring-open-telemetry` in each.

Note the exact start time for each session.

**Tab 1 — Logs agent** (start first, note the time):
```
Create a working OpenTelemetry logs demo in this Spring Boot project.

Add two new files only (do not modify any existing files):
1. app/src/main/java/com/example/demo/service/LogsDemoService.java
2. app/src/main/java/com/example/demo/controller/LogsDemoController.java

The demo should:
- Expose a GET endpoint at /api/logs-demo
- Show all 5 log levels: TRACE, DEBUG, INFO, WARN, ERROR using SLF4J
- Use MDC (org.slf4j.MDC) to attach a correlation ID (UUID) to every log line in the request
- Log structured context: requestId, endpoint name, step number
- Log a simulated "success" path and a "warning" path (use a query param ?fail=true to trigger WARN/ERROR logs)
- Return a JSON response showing what was logged

The OTel Java agent is already attached (via JAVA_TOOL_OPTIONS env var in docker-compose.yml) and will auto-export logs via OTLP to the collector. No changes to build.gradle, application.properties, docker-compose.yml, or HelloController.java.

Open a PR titled "feat: add OpenTelemetry logs demo endpoint" when done.
```

**Tab 2 — Metrics agent** (start within 10s of Tab 1):
```
Create a working OpenTelemetry custom metrics demo in this Spring Boot project.

Add two new files only (do not modify any existing files):
1. app/src/main/java/com/example/demo/service/MetricsDemoService.java
2. app/src/main/java/com/example/demo/controller/MetricsDemoController.java

The demo should:
- Expose a GET endpoint at /api/metrics-demo
- Use the OpenTelemetry Metrics API (io.opentelemetry.api.GlobalOpenTelemetry) to create:
  - A Counter named "demo.requests.total" with attribute "endpoint" = "metrics-demo"
  - A Histogram named "demo.request.duration.ms" recording artificial processing time (50-200ms random)
  - An UpDownCounter named "demo.active.sessions" that increments on request and decrements after 5s (use a scheduled executor)
- Accept query param ?iterations=N (default 3) to record N metric data points per call
- Return a JSON response describing what metrics were recorded and their values

The build.gradle already has opentelemetry-api:1.31.0 and opentelemetry-sdk:1.31.0. The OTel Java agent exports metrics via OTLP automatically. No changes to build.gradle, application.properties, docker-compose.yml, or any existing files.

Open a PR titled "feat: add OpenTelemetry custom metrics demo endpoint" when done.
```

**Tab 3 — Traces agent** (start within 10s of Tab 2):
```
Create a working OpenTelemetry custom traces demo in this Spring Boot project.

Add two new files only (do not modify any existing files):
1. app/src/main/java/com/example/demo/service/TracesDemoService.java
2. app/src/main/java/com/example/demo/controller/TracesDemoController.java

The demo should:
- Expose a GET endpoint at /api/traces-demo
- Use the OpenTelemetry Tracing API (io.opentelemetry.api.GlobalOpenTelemetry) to create:
  - A parent span named "traces-demo-handler" with attributes: http.method=GET, demo.version=1
  - Two child spans: "step-1-validate" and "step-2-process" (sequential, each with their own attributes)
  - A span event on step-2 named "processing.complete" with attribute result.count
  - Error handling: if query param ?fail=true, record an exception on step-2 span and set span status to ERROR
- Add artificial sleep (100ms per step) so traces are visible in SigNoz
- Return a JSON response with the traceId and spanId so the user can look it up in SigNoz

The build.gradle already has opentelemetry-api:1.31.0. No changes to build.gradle, application.properties, docker-compose.yml, or any existing files.

Open a PR titled "feat: add OpenTelemetry custom traces demo endpoint" when done.
```

### Step 4: Monitor All 3 in Parallel

Watch the dev log while sessions are running:
```bash
cat /tmp/proliferate-dev.log | tr -cd '[:print:]\n' | grep -E "session|sandbox|wake|ready|error|agent"
```

Document for each session:
- Start time
- Sandbox ID (from log)
- When sandbox became ready
- Whether all 3 ran concurrently (key proof — overlapping timestamps)

### Step 5: Review Each Agent's Output

Once all 3 sessions complete, for each one:
- What files were created?
- Does the Java code look correct (proper OTel API usage)?
- Did the agent open a PR?
- Does the PR contain ONLY the expected new files (no cross-contamination)?

Check the PRs at: https://github.com/ifelse-codes/spring-open-telemetry/pulls

### Step 6: Assess Merge Strategy

- Can all 3 PRs merge cleanly into `main`? (Expected: yes — no overlapping files)
- If conflicts exist: identify exactly which files conflict and why
- Recommend merge order if needed

---

## Success Criteria
- [ ] GitHub App installed on `spring-open-telemetry`
- [ ] 3 sessions launched within 30s of each other
- [ ] All 3 ran in parallel (confirmed from logs — overlapping sandbox activity)
- [ ] All 3 produced PRs with correct new files
- [ ] Merge strategy: all 3 merge cleanly (no conflicts)

---

## Deliverables
1. **Timing log** — start times, sandbox IDs, ready times for all 3
2. **PR links** — all 3 PRs with file review notes
3. **Merge assessment** — clean or conflict analysis
4. **Updated KNOWN_ISSUES.md** — any new issues
5. **Updated SETUP_LOG.md** — append session 08 results
6. **Session summary**: `prompts/08 - CLAUDE_CODE_PROMPT_summary.md`
7. **Next prompt**: `prompts/09 - CLAUDE_CODE_PROMPT.md` (options in chat)

---

## Notes for Claude
- Confirm dev server is running FIRST before asking user to install GitHub App
- If sessions take >10 min each, that's still valid data — note cold-start timing
- The 3 PRs should touch completely different files — flag immediately if an agent modifies existing files
- After reviewing PRs, push updated knowledge base to `infinite-zero` with session 08 results
