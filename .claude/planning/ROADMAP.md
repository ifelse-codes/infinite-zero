# 90-Day Roadmap

## Month 1: Revenue Foundation (March 2026)

### Week 1 (Mar 8–14): Environment + Payment
- Get local dev running (Docker + pnpm dev)
- Read key specs: sessions-gateway, auth-orgs, billing-metering
- Wire Stripe to Autumn billing
- Implement Free/Pro/Team tier enforcement

### Week 2 (Mar 15–21): Pricing + Gates
- Pricing tiers live in prod
- Session/minute limits enforced per tier
- Credit card capture at signup
- Billing portal (upgrade/downgrade/invoices)

### Week 3 (Mar 22–28): Onboarding
- Self-serve GitHub App setup wizard
- Usage dashboard (credits, sessions, minutes)
- Transactional email (Resend/Postmark)
- Org invite flow polish

### Week 4 (Mar 29–Apr 4): Polish + Ops
- Rate limiting (API + WebSocket per tier)
- Basic admin panel
- Error monitoring (Sentry or similar)
- Deploy to prod, invite first 10 beta users

---

## Month 2: Growth & Retention (April 2026)

- SSO/SAML (Enterprise blocker)
- Automation template gallery (10 pre-built automations)
- PostHog session replay → Linear ticket automation (showcase feature)
- Improved session history + team activity feed
- Webhook API (outbound) for external integrations
- SOC 2 readiness assessment

---

## Month 3: Enterprise & Scale (May 2026)

- Dedicated sandbox pools (no cold starts for Enterprise)
- Audit log API + CSV export
- Custom domains / white-label
- Multi-region deployment (EU + US)
- First enterprise deal ($5K+ ACV target)
- Public launch (Product Hunt, HN Show)

---

## Success KPIs

| Milestone | Target |
|-----------|--------|
| Beta users | 50 by end of Month 1 |
| Paying customers | 10 by end of Month 1 |
| MRR | $2K by end of Month 2 |
| MRR | $10K by end of Month 3 |
| Enterprise pipeline | 3 qualified deals by Month 3 |
