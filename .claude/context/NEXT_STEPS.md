# Next Steps

## Environment: RUNNING ✅

All services are up. Dev environment is fully operational.

### Start Command (every session)
```bash
cd c:/playground/try-proliferate/repo
export PATH="/c/Program Files/nodejs:$PATH"   # Node 24 installed, needs PATH refresh in bash
docker compose up -d postgres redis            # ensure containers running
pnpm dev:raw                                   # start all services
```

### Key URLs
- Web UI: http://localhost:3000 (sign-in page for new users)
- Gateway: http://localhost:8787 (WebSocket — 404 on HTTP is normal)

---

## ✅ MILESTONE: First End-to-End Session Working (Session 05)
- Agent responded to a real prompt → generated a JSON beautifier app
- ISSUE-010 closed
- E2B template rebuild triggered (baking pre-warm for fast cold starts)

## ✅ Session 06: Architecture Deep-Dive + Multi-Agent Test Setup
- E2B template rebuilt ✅ (29s, all 28 layers cached, pre-warm confirmed)
- Architecture fully understood: session creator, wake cycle, admission guard
- Multi-agent parallel test scoped: https://github.com/ifelse-codes/spring-open-telemetry
- **Awaiting**: GitHub App install on target repo + 3 simultaneous session launches

## ✅ Session 07: Repo Analysis + Prompts + Git/PR Setup
- Target repo `ifelse-codes/spring-open-telemetry` fully analyzed
- 3 agent prompts written and ready to paste (logs, metrics, traces)
- ANTHROPIC_API_KEY rotated (old key was expired)
- Dev server port conflict fixed (stale next dev process killed)
- `build-daemon.mjs` fixed (removed hardcoded pnpm path)
- PR #304 opened → https://github.com/proliferate-ai/proliferate/pull/304
- Knowledge base live → https://github.com/ifelse-codes/infinite-zero (private)
- **Awaiting**: GitHub App install on `spring-open-telemetry` + 3 session launches

## Immediate Next Steps — Revenue Features

### 1. Stripe Payment Integration ← HIGHEST PRIORITY
Wire Stripe to the Autumn billing library already in the codebase.
- Read `repo/docs/specs/billing-metering.md` first
- Stripe Checkout + webhooks for subscription lifecycle
- ADR-005 pending: Autumn vs Stripe Direct

### 2. Pricing Tier Enforcement
- Enforce Free/Pro/Team session limits in gateway
- Rate limiting per org
- Block sessions when over quota

### 3. Self-Serve Onboarding Wizard
- Guide new users through GitHub App install
- Reduces support burden, increases activation rate

---

## Environment State ✅ FULLY CONFIGURED

| Component | Status |
|---|---|
| ANTHROPIC_API_KEY | ✅ Set |
| GitHub App `proliferate-iz` | ✅ Configured (ID: 3037923) |
| E2B template `proliferate-base` | ✅ Built (ID: vj1zaph7rttwwh9oe7ug) |
| Dev server | ✅ Running (http://localhost:3000) |
| Docker postgres/redis | ✅ Running |

---

## This Week (Revenue-First)

### Strategic Priorities
1. **Payment Processing** — Wire Stripe to Autumn billing (biggest unblock to revenue)
2. **Pricing Tiers** — Enforce Free/Pro/Team limits in gateway + API
3. **Self-serve Onboarding** — Guided GitHub App setup wizard

### Key Specs to Read Before Feature Work
- `repo/docs/specs/sessions-gateway.md` — core session lifecycle
- `repo/docs/specs/auth-orgs.md` — auth + multi-tenancy
- `repo/docs/specs/billing-metering.md` — current billing state

---

## Decisions Still Pending (DECISIONS.md)
- ⏳ **Billing**: Autumn vs Stripe Direct (ADR-005)
- ⏳ **Target market**: SMB vs Enterprise (ADR-006)
- ⏳ **Deployment**: EKS vs PaaS (ADR-007)

---

## Environment State (Permanent Notes)
- Docker postgres on **port 5433** (not 5432 — conflict with native Windows PostgreSQL 13 service)
- Docker redis on **port 6379**
- `docker-compose.override.yml` in repo root — keep this, recreate on fresh clone
- DB migrations: applied ✅
- **3 env files must be kept in sync**: `.env`, `.env.local`, `apps/web/.env.local`
- Node.js: 24 LTS (installed at `C:\Program Files\nodejs\`)
- pnpm: 10.7.0 (repo specifies 8.15.1 — non-blocking)
- Use `pnpm dev:raw` not `pnpm dev` (inline env var syntax breaks on Windows CMD)
