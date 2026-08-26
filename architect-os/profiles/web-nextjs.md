# Profile: Next.js Full-Stack Web

*The default profile — [tech-stack.md](../tech-stack.md) is its long-form rationale. This file exists so web repos are configured the same way as any other type: through a profile, not by being the unstated default.*

---

## Stack

The [tech-stack.md](../tech-stack.md) default, unchanged: TypeScript strict, Next.js App Router, Postgres (Neon/Supabase), Drizzle, Better Auth (or Supabase Auth — ADR-0001 picks), Tailwind + shadcn/ui, Zod at boundaries, Vitest + Testing Library + Playwright, pnpm, Vercel, Sentry. Deviations are ADR line-items.

## Harness & model routing

Either named stack as-is — this is the project type all the OS defaults were calibrated on ([harness-matrix.md](../harness-matrix.md) Part VI applies without adjustment). TypeScript is the best-served language in every model's training data; the standard routing (frontier for S2/S4/S5, implementation tier for S6) holds.

## MCP set

| MCP | Purpose | When |
|---|---|---|
| **Supabase MCP** (or Neon MCP) | Schema, migrations, logs, advisors | Always if that's the DB |
| **Figma MCP** | Design → component at S3/S6 | UI-heavy repos |
| **GitHub MCP** (or `gh` CLI) | Issues/PR flow | Always |
| **Playwright/browser MCP** | Drive the app, screenshot states at S7 | E2E-heavy repos |
| **Sentry MCP** | Production errors into triage | After first deploy |

## Constitution annex — W-rules

- **W1 — Server/client boundary is explicit.** 🟠 `"use client"` is a decision, not a fix; data access only in server components, route handlers, or server actions — never in client components.
- **W2 — Zod at every boundary.** 🟠 C12 concretized: route handlers, server actions, env vars (`env.ts`), external API responses. `as`-casting external data is a violation.
- **W3 — Migrations are generated, reviewed, and reversible.** 🔴 Drizzle-kit generates; humans review SQL line-by-line (lifecycle S4 rule); every migration states its rollback.
- **W4 — No client-side secrets.** 🔴 `NEXT_PUBLIC_` is a publish decision; C31 applied to the bundle boundary.
- **W5 — Loading/empty/error states ship with the screen.** 🟡 The FSD design-brief rule enforced at review: `loading.tsx`/`error.tsx`/empty-state or a stated reason why not.
- **W6 — a11y is analyzer-enforced.** 🟡 eslint-plugin-jsx-a11y on; violations are bugs, not nits.

## CI & checks

The base [github/workflows/ci.yml](../github/workflows/ci.yml) as-is (Typecheck, Lint, Test, Build) plus **E2E** (Playwright against a preview build) — required on M-size PRs, scheduled otherwise.

## Test strategy

| Layer | Tool | Converge evidence |
|---|---|---|
| Unit (services, utils, schemas) | Vitest | Per criterion |
| Component (interactive components) | Vitest + Testing Library | Behavior asserts (C16) |
| E2E (critical journeys) | Playwright | Happy path + the FSD's top edge cases |
| Visual (optional) | Playwright screenshots | Human eyeballs at S7 |

## Review path-instructions (append to .coderabbit.yaml)

```yaml
    - path: "src/**/*.{ts,tsx}"
      instructions: |
        Web annex rules W1–W6 (architect-os/profiles/web-nextjs.md) apply with
        C-rule severity semantics. Specifically flag: data fetching or DB access
        in client components [W1], external data cast with `as` instead of
        parsed with Zod [W2], NEXT_PUBLIC_ env vars carrying anything secret
        [W4], new screens without loading/empty/error handling [W5].
    - path: "**/drizzle/**"
      instructions: |
        Migrations: generated only [C30], reversible with stated rollback [W3].
        Flag destructive operations (DROP, column type changes) without a
        migration-safety note.
```

## AGENTS.md seed

Commands: `pnpm dev` / `pnpm vitest run <path>` / `pnpm lint && pnpm typecheck` / `pnpm db:generate` + `pnpm db:migrate`. Do-not-touch: `drizzle/meta/`, generated route types. Starter gotchas: server actions need `revalidatePath`/`revalidateTag` after mutations; Next caching defaults change across minors — pin and verify with grill-with-docs.
