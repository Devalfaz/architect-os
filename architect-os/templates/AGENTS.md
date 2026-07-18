# `AGENTS.md` — Agent Entrypoint Template

*Copy this to your repo root as `AGENTS.md`. Create a symlink: `ln -s AGENTS.md CLAUDE.md`. Keep it ≤150 lines. Update `last_verified` weekly.*

---

```markdown
# [PROJECT NAME]

<!-- last_verified: YYYY-MM-DD -->

## What this is

[One sentence: what this repo is, who it's for, what problem it solves.]

## Tech stack

- **Runtime:** Node.js 22 / TypeScript 5.x (strict mode)
- **Framework:** Next.js 15 (App Router)
- **Database:** PostgreSQL (Supabase), Drizzle ORM
- **Auth:** Supabase Auth
- **Validation:** Zod
- **UI:** Tailwind CSS + shadcn/ui
- **Testing:** Vitest + Playwright
- **Hosting:** Vercel

## Domain language

| Term | Definition |
|---|---|
| **[Entity 1]** | [Definition — what agents must understand to talk about this domain correctly] |
| **[Entity 2]** | |
| ... | |

## File conventions

- `src/app/` — Next.js App Router pages and API routes
- `src/server/` — Server-side logic: services, db, auth
  - `src/server/<domain>/service.ts` — Business logic for a domain
  - `src/server/db/` — Database schema, migrations, queries
- `src/components/` — Shared UI components
- `src/lib/` — Shared utilities (no business logic)
- `tests/` — Test files mirror `src/` structure
- `docs/` — Product specs, ADRs, architecture

## Architecture

[Architecture style, e.g., "Hexagonal architecture with ports and adapters"]
[Key design decisions, 3-5 bullet points]

## Constitution (see constitution.md)

The 5 most commonly violated rules:
- **C4:** Tests before implementation (TDD)
- **C6:** Implement the ticket, not the repo — no scope creep
- **C8:** No new dependencies without an ADR line
- **C9:** PR ≤400 lines
- **C12:** Validate at boundaries (Zod)

## Gotchas

Things that have tripped up agents before:
- [Gotcha 1 — with workaround]
- [Gotcha 2 — with workaround]
- ...

## Active constraints

- Squash merge only to `main`
- Branch naming: `feat/NNN-slug`, `fix/NNN-slug`
- WIP limit: 2 concurrent agent sessions
- One ticket = one branch = one PR = one squash commit
```

---

*Template last updated: 2025-01-15. Fill in the sections specific to your project. The convention: sections marked `<!-- last_verified: ... -->` are checked weekly in the memory distill.*
