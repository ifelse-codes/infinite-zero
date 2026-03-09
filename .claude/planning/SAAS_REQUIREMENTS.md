# SaaS Requirements: Gap Analysis

## What Proliferate Already Has

### Multi-tenancy ✅
- Org-based isolation built in from day one
- All data scoped to org
- Role-based membership (users belong to orgs)
- API keys per org

### Authentication ✅
- better-auth with OAuth (GitHub)
- Session management
- API key authentication for programmatic access

### Billing Foundation ✅ (partial)
- Autumn integration for credit-based metering
- Trial credits system
- Org pause on credit depletion
- Usage tracking exists

### Background Execution ✅
- BullMQ workers
- Trigger system (cron, webhooks, GitHub, Linear, Sentry)

### Audit Logging ✅ (partial)
- Structured logging with pino
- Session command logs

---

## Gaps for Commercial Launch

### 🔴 Critical (Blocking Revenue)

| Gap | Description | Effort |
|-----|-------------|--------|
| **Payment Processing** | Stripe/similar not integrated; Autumn handles metering but payment collection needs wiring | 3-5 days |
| **Self-serve Onboarding** | GitHub App setup is manual; needs guided wizard | 2-3 days |
| **Pricing Tiers** | No tier enforcement beyond credits; need Free/Pro/Team/Enterprise gates | 3 days |
| **Usage Dashboard** | No user-facing credit/usage visibility | 2 days |

### 🟡 Important (First 30 Days)

| Gap | Description | Effort |
|-----|-------------|--------|
| **Email Notifications** | No transactional email (invite, billing alerts, session complete) | 2 days |
| **Admin Panel** | No internal tools for managing orgs/users/billing | 3 days |
| **Invite Flow Polish** | Org invitations exist but UX needs hardening | 1 day |
| **Org Settings Page** | Members, billing, integrations in one place | 2 days |
| **Rate Limiting** | API + WebSocket rate limits per tier | 2 days |

### 🟢 Nice-to-Have (60-90 Days)

| Gap | Description | Effort |
|-----|-------------|--------|
| **SSO / SAML** | Enterprise requirement for team plans | 5 days |
| **SOC 2 Prep** | Audit logs, data retention policies | 5+ days |
| **SLA Dashboard** | Uptime/status page for enterprise | 2 days |
| **Webhooks** | Outbound webhooks for automation results | 2 days |
| **Custom Domains** | White-label for enterprise | 3 days |

---

## Technical SaaS Hardening Needed

1. **Environment variable management**: `NEXT_PUBLIC_` vars require full rebuild — document this for ops
2. **Multi-region**: Current infra is single-region EKS; needs CDN + multi-region Redis for scale
3. **Data residency**: EU/US region isolation for enterprise compliance
4. **Backup/DR**: PostgreSQL backup strategy not documented
5. **Observability**: Need application APM (Datadog/New Relic or OpenTelemetry)
6. **Secret rotation**: AWS Secrets Manager in place but rotation policy missing

---

## Recommended Build Order (Revenue-First)

```
Week 1: Payment processing + pricing tier enforcement
Week 2: Self-serve onboarding wizard + usage dashboard
Week 3: Email notifications + admin panel basics
Week 4: Rate limiting + org settings polish
Month 2: SSO + SOC 2 prep + observability
Month 3: Enterprise features + multi-region
```
