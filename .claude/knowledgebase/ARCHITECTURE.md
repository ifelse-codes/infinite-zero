# Architecture Overview

## System Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                │
│   Web (Next.js)  ·  CLI  ·  Slack App                               │
└────────────┬─────────────────────────────────┬───────────────────────┘
             │ HTTP (session lifecycle)          │ WebSocket (real-time)
             ▼                                   ▼
┌────────────────────────┐          ┌────────────────────────┐
│   Next.js API Routes   │          │       Gateway           │
│  (session CRUD only)   │◄────────►│  (WebSocket hub, state) │
└────────────┬───────────┘          └────────────┬────────────┘
             │                                    │ HTTP
             ▼                                    ▼
┌────────────────────────┐          ┌────────────────────────┐
│    PostgreSQL (meta)   │          │  Sandbox (Modal / E2B) │
│    Redis (queues)      │          │   + OpenCode agent      │
└────────────────────────┘          └────────────────────────┘
```

**Critical rule**: API routes are NOT in the real-time streaming path.
All streaming: `Client ↔ Gateway ↔ Sandbox`

---

## Monorepo Structure

```
apps/
  web/             — Next.js 16 frontend + API routes
  gateway/         — WebSocket server (real-time streaming hub)
  worker/          — BullMQ job processors (K8s pods)
  llm-proxy/       — LiteLLM proxy for model routing + spend tracking
  trigger-service/ — Webhook/cron/event trigger ingestion

packages/
  db/              — Drizzle schema + migrations (PostgreSQL)
  services/        — Business logic layer (DB reads/writes via db.ts files)
  shared/          — Pure helpers/types, sandbox config, provider defs
  gateway-clients/ — WebSocket client SDK used by web + CLI
  environment/     — Env schema validation (source of truth: schema.ts)
  logger/          — Structured logging (pino-based)
  providers/       — LLM provider adapters
  queue/           — BullMQ queue definitions
  triggers/        — Trigger handler implementations

infra/
  pulumi-k8s/      — Production EKS (AWS) via Pulumi + Helm
  pulumi-k8s-gcp/  — GKE (GCP) via Pulumi
  pulumi/          — Legacy ECS (manual only)
  terraform/       — Legacy Terraform (manual only)

charts/
  proliferate/     — Helm chart for K8s deployment
```

---

## Key Subsystems

### Sessions & Gateway (`docs/specs/sessions-gateway.md`)
- Session lifecycle: create → active → paused → deleted
- Gateway manages WebSocket connections and message state
- Each session runs in an isolated cloud sandbox

### Sandboxes (`docs/specs/sandbox-providers.md`)
- Providers: **Modal** (default) or E2B
- Each run gets fresh isolated environment mirroring Docker setup
- OpenCode is the embedded coding agent inside the sandbox
- Snapshots enable fast session resume

### Automations & Triggers (`docs/specs/automations-runs.md`, `triggers.md`)
- Triggers: GitHub webhooks, Linear, Sentry, PostHog, cron, webhooks
- Run pipeline: enrich → execute → finalize → outbox

### Auth & Orgs (`docs/specs/auth-orgs.md`)
- Library: **better-auth**
- Multi-tenant: Users belong to Orgs; all data is org-scoped
- API keys for programmatic access

### Billing (`docs/specs/billing-metering.md`)
- Provider: **Autumn**
- Credit-based metering, trial credits, org pause on depletion

### LLM Proxy (`docs/specs/llm-proxy.md`)
- LiteLLM with virtual keys
- Per-org spend tracking and model routing

---

## Data Flow: Session Execution

```
1. User sends prompt (web/CLI/Slack)
2. Gateway receives over WebSocket
3. Gateway routes to sandbox via HTTP
4. OpenCode agent executes in sandbox
5. Events stream back: Gateway → Client (WebSocket)
6. On completion: snapshot saved, PR/preview URL generated
7. Worker finalizes: artifacts, notifications, billing meter
```

---

## Infrastructure (Production)

- **Compute**: EKS (AWS) — Gateway + Worker as K8s deployments
- **Database**: PostgreSQL (RDS or self-hosted)
- **Cache/Queue**: Redis (ElastiCache)
- **Secrets**: AWS Secrets Manager → External Secrets → K8s secrets
- **CI/CD**: GitHub Actions (`deploy-eks.yml`)
- **Self-host**: Docker Compose with optional Caddy for HTTPS

---

## Session Creator — Key Detail (Session 06)
From reading `apps/gateway/src/api/proliferate/http/sessions/session-creator.ts`:
- Sessions are created as `pending` → sandbox boot is **always deferred** (eager-start called separately)
- Plan limits enforced at creation via `createSessionWithAdmissionGuard()`
- Each session: UUID, configurationId, orgId, snapshotId, provider type stored in DB
- Parallel sessions are fully independent — each gets its own UUID + sandbox boot

## Wake Cycle (Agent Execution Engine)
From `apps/gateway/src/harness/manager/wake-cycle/`:
- **4 phases**: triage → ingest → orchestrate → finalize
- Triage: determines session intent and context
- Ingest: hydrates prompt + context into OpenCode
- Orchestrate: streams agent events back to client
- Finalize: saves snapshot, records metrics, billing meter tick

## Session Admission Guard
- Concurrent session limits are plan-enforced via `billing.getOrgPlanLimits()`
- If limit reached: throws immediately (no queue, no wait)
- Without billing config: no limit enforced (self-hosted default)

---

## Strengths
- Clean separation: streaming never touches API routes or DB
- Spec-driven: every subsystem has an authoritative spec doc
- Multi-tenant from day one (org-based isolation)
- Sandbox isolation prevents cross-tenant contamination
- Sessions are fully parallel by design — each sandbox is independent
- Admission guard allows plan-based concurrency limits without code changes

## Weaknesses / Technical Debt (Updated Session 06)
- Legacy ECS infra still exists alongside K8s (dead weight)
- `apps/web` lib/ vs packages/ boundary violations probable
- No E2E test suite currently
- `NEXT_PUBLIC_` vars require full rebuild on change (documented limitation)
- Modal sandbox Python code lives outside monorepo TS toolchain
- No self-serve onboarding wizard (requires manual GitHub App install instruction)
- Billing enforcement only at session start — no mid-session quota checks
- Auth: `admin.sentryTestError` is publicly callable (un-gated oRPC endpoint — security debt)
