# Tech Stack

## Core Languages
| Language | Usage |
|----------|-------|
| TypeScript | Everything (API, frontend, gateway, workers) |
| Python | Modal sandbox image only (`packages/modal-sandbox/`) |

## Frontend (apps/web)
| Tech | Version | Rationale |
|------|---------|-----------|
| Next.js | 16.1.6 | Full-stack React, API routes, SSR |
| React | ^19.2.3 | UI framework |
| TanStack Query | latest | Server state, caching, data fetching |
| Zustand | ^4.5.0 | Client-only UI state |
| Tailwind CSS | latest | Utility-first styling |
| shadcn/ui | latest | Accessible component primitives |
| oRPC | ^1.13.4 | Type-safe RPC layer (replaces raw fetch) |

## Backend / API
| Tech | Usage |
|------|-------|
| Next.js API routes | Session lifecycle (create/pause/resume/delete) — NOT streaming |
| better-auth ^1.4.12 | Authentication, OAuth, API keys |
| Drizzle ORM | PostgreSQL queries (type-safe, no raw SQL) |
| Zod ^3.22.3 | Schema validation everywhere |

## Real-Time
| Tech | Usage |
|------|-------|
| WebSocket (ws ^8.16.0) | Gateway ↔ Client, Gateway ↔ Sandbox |
| Custom Gateway (apps/gateway) | WebSocket hub, session state |

## Infrastructure
| Tech | Usage |
|------|-------|
| PostgreSQL 15 | Metadata persistence (sessions, orgs, billing) |
| Redis 7 | BullMQ job queue + caching |
| BullMQ | Background job processing (workers) |
| Docker / Docker Compose | Local dev + self-hosting |
| Kubernetes (EKS) | Production compute |
| Helm | K8s deployment charts (`charts/proliferate/`) |
| Pulumi | Infrastructure as Code (EKS + GKE) |

## Sandbox Providers
| Provider | Status | Notes |
|----------|--------|-------|
| Modal | Default | Python-based cloud sandbox, fast cold start |
| E2B | Alternative | Fully managed sandbox alternative |
| OpenCode | Agent | Coding agent running inside sandbox |

## Build & Tooling
| Tool | Usage |
|------|-------|
| Turborepo ^1.12.0 | Monorepo task orchestration, caching |
| pnpm 8.15.1 | Package manager (workspaces) |
| Biome ^1.5.3 | Linter + formatter (replaces ESLint/Prettier for most) |
| ESLint | Web app only (additional rules) |
| Vitest | Unit + integration testing |
| TypeScript ^5.3.3 | Type checking |
| Husky + lint-staged | Pre-commit hooks |

## Integrations
| Service | Purpose |
|---------|---------|
| GitHub App | Repo access, PR creation, webhook triggers |
| Slack | Notifications, Slack-based agent sessions |
| Linear | Issue trigger source |
| Sentry | Error monitoring + trigger source |
| PostHog | Session replay trigger source |
| LiteLLM (llm-proxy) | Model routing + spend tracking |
| Autumn | Billing / credit metering |
| Nango | OAuth connection management |
| AWS Secrets Manager | Production secret management |

## License Constraints
All dependencies must be MIT / Apache-2.0 / BSD-compatible.
**No GPL, AGPL, SSPL, or BSL** — this repo is public/open source.
