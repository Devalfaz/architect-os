# `AGENTS.md` — Agent Entrypoint Template

*Copy this to your repo root as `AGENTS.md`. Create a symlink: `ln -s AGENTS.md CLAUDE.md`. Keep it ≤150 lines — and treat that as a ceiling, not a target. Update `last_verified` weekly.*

**The pruning test, applied to every line:** *"Would removing this cause the agent to make a mistake?"* If not, cut it. Bloated entrypoint files cause agents to ignore the instructions that matter (Anthropic's guidance is blunt: bloated CLAUDE.md files make Claude ignore your actual instructions).

**Two reasons the ceiling is real, not stylistic** ([evidence](../research-2026-08-context-engineering.md)):
1. **Context rot.** Every token here is loaded into *every* session. Degradation as input grows is measured across frontier models — a long entrypoint doesn't just get skimmed, it degrades everything downstream of it.
2. **Prefix stability.** This file is the cached prompt prefix. Every edit invalidates that cache, and cache reads are 10–50× cheaper than misses. **Batch edits into the weekly distill; don't trickle them.** Churn here is expensive twice — in tokens, and in the recall that keeping history buys you.

| ✅ Include | ❌ Exclude |
|---|---|
| Commands the agent can't guess | Anything discoverable by reading code (`package.json`, directory tree, imports) |
| Style rules that differ from language defaults | Standard conventions the model already knows |
| Testing instructions and the preferred runner | Detailed API documentation (link instead) |
| Architectural decisions specific to this repo | Information that changes frequently (belongs in tickets/specs) |
| Domain language and gotchas | File-by-file descriptions of the codebase |
| Developer-environment quirks | Long explanations and tutorials |

---

```markdown
# [PROJECT NAME]

<!-- last_verified: YYYY-MM-DD -->

## What this is

[One sentence: what this repo is, who it's for, what problem it solves.]

## Commands

<!-- Only commands an agent can't guess from package.json scripts alone:
     non-obvious flags, required env setup, order dependencies. -->
- Dev: `pnpm dev`
- Test (single file): `pnpm vitest run <path>` — always run the single file first, full suite before PR
- Lint + typecheck: `pnpm lint && pnpm typecheck` — both must pass before any commit
- DB migrate (local): `pnpm db:migrate` — requires `pnpm db:start` first

## Domain language

<!-- The 5–10 terms agents must use correctly. Wrong entity names in code = bounce. -->
| Term | Definition |
|---|---|
| **[Entity 1]** | [What agents must understand to talk about this domain correctly] |
| **[Entity 2]** | |

## Architecture decisions that bind you

<!-- Not a description of the architecture (read docs/architecture/ for that) —
     the 3–5 DECISIONS an agent must not unmake. Each links its ADR. -->
- [e.g., All DB access goes through `src/server/<domain>/service.ts` — never query from routes (ADR-0003)]
- [e.g., Errors are Result values at service boundaries — no throws across layers (ADR-0007)]
- Full map: `docs/architecture/architecture.md` · Decisions: `docs/adr/`

## Constitution (see constitution.md — C1–C37)

The 5 most commonly violated rules here:
- **C4:** Tests before implementation (TDD)
- **C6:** Implement the ticket, not the repo — no scope creep
- **C8:** No new dependencies without an ADR line
- **C9:** PR ≤400 lines
- **C12:** Validate at boundaries (Zod)

## Gotchas

<!-- Things that have actually tripped up agents in THIS repo — with the workaround.
     Feed from memory dumps; prune fixed ones weekly. -->
- [Gotcha 1 — with workaround]

## Do not touch

- [e.g., `src/legacy/**` — migration in progress, ticket train #250–#255]
- Generated files (C30): [e.g., `src/db/migrations/**`]

## Constraints

- Squash merge only to `main` · branch naming `feat/NNN-slug`
- One ticket = one branch = one PR = one squash commit

## Context discipline

<!-- Keep these four lines verbatim — they are the highest-leverage lines in the file. -->
- **Load on demand, never wholesale.** Do not slurp `docs/`. Follow links:
  specs `docs/specs/` · graph `memory/repo-graph.json` · gotchas `memory/dumps/`
- **One ticket's worth of context.** Files in the ticket plan, the linked spec
  section, and this file. Nothing else. A wider window produces worse code, not
  just costlier code.
- **Cap what you pull in.** Prefer a targeted read over a full-file dump, and a
  file reference over pasted content.
- **If context is running out, stop and say so** — do not summarize the session
  and continue. Report what's done, what's left, and open questions; the work
  resumes in a fresh session against the frozen plan.
```

---

*What was deliberately left out of this template — and stays out:* a tech-stack list (readable from `package.json` in one tool call), file-tree conventions (readable from the tree itself), API documentation (linked, not pasted). If the agent keeps violating a rule that IS in the file, the file is probably too long and the rule is getting lost — cut something else before adding emphasis.
