# Tech Stack — Default and When to Deviate

*Opinionated default for a modern full-stack app. Agent-compatible. Deviate with an ADR.*

---

## The Default Stack

```
Framework:    Next.js 15 (App Router)
Language:     TypeScript (strict mode)
UI:           Tailwind CSS + shadcn/ui
Database:     PostgreSQL (via Supabase or Neon)
ORM:          Drizzle ORM
Auth:         Supabase Auth or NextAuth.js v5
Validation:   Zod
API:          tRPC (internal) + Next.js Route Handlers (external)
State:        TanStack Query (server state) + Zustand (client state)
Testing:      Vitest + Playwright (E2E)
Hosting:      Vercel (frontend/serverless) or Cloudflare Workers
CI/CD:        GitHub Actions
Monorepo:     Turborepo (if >2 packages/apps)
```

### Why

**Next.js App Router:** Mature, strong TypeScript, huge agent training data.
**TypeScript strict:** Compiler catches what agents miss.
**Tailwind + shadcn/ui:** Utility CSS agents generate reliably. shadcn/ui composable components.
**PostgreSQL + Drizzle ORM:** TypeScript-native with excellent type inference.
**Supabase Auth:** Managed auth with RLS. Less to implement = less to get wrong.
**Zod:** Runtime boundary validation per C12.
**tRPC:** End-to-end type safety. Agents following types get API calls right.
**Vitest + Playwright:** Fast, modern, well-documented.
**Vercel:** Zero-config Next.js hosting with per-PR preview deployments.

---

## App-Type Variants

### SaaS
Add: Stripe (payments), Resend (email), Inngest (background jobs), PostHog (analytics), Sentry (errors)

### Internal Tools
Swap: Retool or Next.js + shadcn (admin), Clerk/WorkOS (SSO auth)

### Mobile
Swap: Expo (React Native) + Tamagui. Backend stays the same. Testing: Maestro (E2E mobile)

### CLI Tool
Swap: Bun runtime, Commander.js, pkg or Bun compile. TypeScript still.

### AI Application
Add: Vercel AI SDK (streaming), pgvector (PostgreSQL extension), dual-provider model SDKs (Anthropic + OpenAI). Cloudflare Agents SDK if on Workers.

### Data-Heavy
Swap: ClickHouse/TimescaleDB for analytics, Redis (Upstash) for caching, Fly.io/Railway for persistent compute, Inngest for data pipelines.

---

## Deviation triggers (ADR required)

- Different language (Rust, Go, Python) — adds training-data penalty
- Different database (MongoDB, Neo4j, DynamoDB) — agents weaker at NoSQL
- Different framework (Remix, SvelteKit, Nuxt) — thinner training data
- Keeping an existing stack — ADR justifies why
- Default unmaintained or has serious vulnerability — replace with ADR

---

## Agent compatibility

Claude and Copilot trained on code up to early 2025. Strong on: Next.js App Router, Drizzle, shadcn/ui, tRPC, Vitest, TanStack Query, Zustand, Playwright. Libraries after early 2025: always `grill-with-docs` to verify API claims.

---

## Monorepo

Default: single repo for solo projects. Turborepo triggers: ≥3 packages/apps sharing code, shared UI library, multiple deployables, build caching needed.
