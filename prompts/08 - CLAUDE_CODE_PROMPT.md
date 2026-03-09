# Session 08 — [PLACEHOLDER: replace with chosen option]

<!-- User to replace this file after picking A/B/C from Session 07 chat -->
<!-- Options were:
  A — Launch + Monitor (run the 3 parallel sessions, capture results)
  B — Revenue: Stripe + Billing Integration
  C — Self-Serve Onboarding Wizard
-->

## Context (from Sessions 01-07)

**Project**: Proliferate — open-source background agent platform, being commercialized as SaaS.
**Repo**: `c:/playground/try-proliferate/repo/`
**Knowledge base**: `c:/playground/try-proliferate/.claude/`

### Current State
- ✅ Full dev environment running (`pnpm dev:raw`)
- ✅ Web UI at http://localhost:3000
- ✅ First end-to-end agent session worked (Session 05)
- ✅ Architecture fully understood (session creator, wake cycle, admission guard)
- ✅ E2B template rebuilt (Session 06 — pre-warm confirmed)
- ✅ Target repo analyzed (`ifelse-codes/spring-open-telemetry`) — Session 07
- ✅ 3 session prompts written and ready (logs, metrics, traces) — Session 07
- ⏳ GitHub App install on target repo — USER ACTION NEEDED
- ⏳ 3 parallel sessions not yet launched

---

## OPTION A: Launch + Monitor (Multi-Agent Parallel Test)

### Pre-condition
GitHub App `proliferate-iz` must be installed on `ifelse-codes/spring-open-telemetry` before starting.
Verify: http://localhost:3000 → Settings → Repositories → confirm repo shows up.

### Step 1: Install GitHub App (User Action — if not done yet)
1. Go to http://localhost:3000 → Settings → Repositories
2. Click "Install GitHub App" → select `ifelse-codes/spring-open-telemetry`
3. Confirm repo appears in UI before proceeding

### Step 2: Launch 3 Sessions Simultaneously (User Action)

Open 3 browser tabs: http://localhost:3000
Navigate to `ifelse-codes/spring-open-telemetry` in each.

**Tab 1 — Logs agent** (start first):
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

Note start time for each session. All 3 should start within 30 seconds of each other.

### Step 3: Monitor All 3

Watch the dev log:
```bash
tail -f /tmp/proliferate-dev.log | grep -E "session|sandbox|wake|ready|error|agent" --line-buffered
```

Document:
- Start time for each session
- Sandbox ID for each session
- When each sandbox became ready
- Whether all 3 ran in parallel or sequentially (key proof of concept)

### Step 4: Review Agent Output

For each session after it completes:
- What files did it create or modify?
- Does the code look correct (Java syntax, correct OTel API usage)?
- Did it open a PR?
- Does the PR contain only the expected new files (no accidental cross-feature changes)?

### Step 5: Assess Merge Strategy

- Can all 3 PRs merge cleanly into main (expected: yes, no overlapping files)?
- If conflicts: identify which files conflict and why
- Recommend merge order if needed

### Success Criteria
- [ ] GitHub App installed on target repo
- [ ] 3 sessions launched within 30s of each other
- [ ] All 3 ran in parallel (confirmed from logs)
- [ ] All 3 produced PRs with correct code
- [ ] Merge strategy: all 3 can merge cleanly

---

## OPTION B: Revenue — Stripe + Billing Integration

### Context
The Autumn billing library is already in the codebase but not wired to Stripe.
Read `repo/docs/specs/billing-metering.md` before making any changes.

### Objectives
1. Read `billing-metering.md` spec in full
2. Decide: Autumn library vs Stripe Direct (document ADR-005)
3. Implement Stripe Checkout session creation
4. Implement webhook handler for subscription lifecycle events (created, updated, cancelled)
5. Wire plan limits to the gateway's admission guard
6. Test with Stripe test mode (test API keys)

### Files Likely Involved
- `apps/web/` — Checkout redirect button
- `apps/gateway/src/api/` — Webhook receiver
- `packages/` — Billing service/client
- `repo/docs/specs/billing-metering.md` — Current billing spec

---

## OPTION C: Self-Serve Onboarding Wizard

### Context
New users land at http://localhost:3000 but have no guidance on how to install the GitHub App
or run their first session. This causes friction and drop-off.

### Objectives
1. Read existing auth flow + settings pages to understand current UI structure
2. Design a 3-step onboarding wizard:
   - Step 1: Install GitHub App (link + confirmation check)
   - Step 2: Pick a first repo
   - Step 3: Run first session (with a suggested prompt)
3. Implement wizard as a new page/modal in `apps/web/`
4. Wire completion state (skip wizard if already has sessions)
5. Test end-to-end from fresh account

---

_Delete this file contents and replace with the chosen option's objectives when starting Session 08._
