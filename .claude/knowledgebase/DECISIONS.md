# Architecture Decision Records (ADRs)

## ADR-001: Use Proliferate MIT Repo as Base
**Date**: 2026-03-08
**Status**: Accepted
**Decision**: Fork/use `proliferate-ai/proliferate` (MIT) as foundation for commercial SaaS.
**Rationale**: Saves 6-12 months of infra work. Multi-tenancy, sandbox isolation, trigger system, and gateway are all production-grade. MIT license allows commercial use with proprietary enhancements.
**Risks**: Repo in active migration (unstable period). Upstream changes to track.

---

## ADR-002: Keep Turborepo Monorepo Structure
**Date**: 2026-03-08
**Status**: Accepted
**Decision**: Keep pnpm workspaces + Turborepo.
**Rationale**: Boundary enforcement via custom lint scripts (`check-db-boundary.mjs`, etc.) prevents architectural decay. Shared packages (`@proliferate/services`, `@proliferate/shared`) are already well-structured.
**Alternative rejected**: Split into microrepos — too much overhead for current team size.

---

## ADR-003: Streaming Architecture (No API Routes in Real-Time Path)
**Date**: 2026-03-08
**Status**: Inherited (keep)
**Decision**: All real-time streaming goes `Client ↔ Gateway ↔ Sandbox`. Next.js API routes handle session lifecycle only.
**Rationale**: Prevents Next.js serverless cold starts from interrupting streams. Gateway is always-on WebSocket server purpose-built for this.

---

## ADR-004: Sandbox Provider — E2B
**Date**: 2026-03-08
**Status**: Accepted
**Decision**: Use E2B as the sandbox provider for local dev and production.
**Rationale**: E2B is simpler to set up than Modal. Modal requires a separate Python deploy step (`modal deploy deploy.py`) which adds friction on Windows and CI. E2B is a pure HTTP API with a Node SDK — no Python toolchain needed.
**Alternative rejected**: Modal — too much setup friction (Python + modal CLI deploy) for initial dev velocity.
**Config**: `DEFAULT_SANDBOX_PROVIDER=e2b`, `E2B_TEMPLATE=proliferate-base`

---

## ADR-005: Billing — Evaluate Autumn vs Stripe Direct
**Date**: 2026-03-08
**Status**: Pending user decision
**Options**:
- **A) Autumn extension** — The codebase already has Autumn wired in (`apps/web/src/lib/autumn.ts`). Autumn wraps Stripe with metering/entitlements. Fastest path to billing.
- **B) Stripe direct** — Remove Autumn, wire Stripe directly. More control, more work.
**Recommendation**: Keep Autumn — it's already integrated. Faster to production revenue.

---

## ADR-006: Target Market — Pending Decision
**Date**: 2026-03-08
**Status**: Pending user decision
**Options**:
- **A) SMB SaaS teams** — Self-serve onboarding, lower ACV, faster to first paying customer. Invest in UX/onboarding.
- **B) Enterprise first** — Higher ACV but requires SSO (SAML), audit logs, longer sales cycle.
**Recommendation**: SMB first — generate revenue signal quickly; add enterprise features in Series A.

---

## ADR-007: Deployment Target — Pending Decision
**Date**: 2026-03-08
**Status**: Pending user decision
**Options**:
- **A) Keep EKS** — Production-grade but expensive ($300+/mo) and complex to operate for early stage.
- **B) Managed PaaS (Railway / Render)** — Lower ops overhead, faster deploys, easier to iterate. ~$50-150/mo for early traffic.
**Recommendation**: Railway/Render for MVP → migrate to EKS at scale.

---

*Add new ADRs here as decisions are made. Format: Date, Status, Decision, Rationale, Alternatives Rejected.*
