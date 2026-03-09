# MVP Features for Commercial Launch

## Core Value Proposition
"Give your AI agents secure access to real dev environments, run them in the background, get back a live preview + PR."

## Tier 1: Already Built (Ship As-Is)
These work and differentiate us:
- [ ] Agent sessions in isolated sandboxes (Modal/E2B)
- [ ] Real-time multiplayer (watch/steer any session)
- [ ] GitHub integration (clone repo, create PR)
- [ ] Background execution with result artifacts
- [ ] Trigger system (cron, GitHub webhooks, Linear, Sentry)
- [ ] Multi-tenant orgs with member management
- [ ] CLI client
- [ ] Slack integration (run sessions from Slack)

## Tier 2: Must-Have Before Revenue (4 weeks)

### Week 1–2: Payment + Pricing
- [ ] Stripe integration wired to Autumn metering
- [ ] Free tier: X sessions/month, limited sandbox minutes
- [ ] Pro tier ($49/mo): Unlimited sessions, priority sandboxes
- [ ] Team tier ($199/mo): 5 seats, shared session history, admin controls
- [ ] Credit card capture at signup (no credit card = free tier only)
- [ ] Billing portal (view invoices, upgrade/downgrade)

### Week 3: Onboarding
- [ ] Guided setup wizard (GitHub App → first repo → first session)
- [ ] Usage dashboard (credits used, sessions run, sandbox minutes)
- [ ] Org invite flow polish (email invite → accept → team access)

### Week 4: Retention & Ops
- [ ] Transactional email (welcome, billing alert, session complete)
- [ ] Basic admin panel (list orgs, impersonate, manage credits)
- [ ] Rate limiting per tier (API + sessions/day)

## Tier 3: Post-Launch (Month 2–3)
- [ ] SSO / SAML (Enterprise gate)
- [ ] Custom MCP server marketplace
- [ ] PostHog → Linear → PR automation template gallery
- [ ] Audit log export (CSV/API) for compliance
- [ ] Dedicated sandboxes for Enterprise (no cold starts)
- [ ] White-label / custom domain option

## Non-Goals for MVP
- Multi-region deployment (single US region fine for beta)
- SOC 2 certification (start process, not required at launch)
- Mobile app
- On-premise deployment (self-host Docker Compose covers this)
