# OS Backlog — Pending Improvements

*Changes to the OS **itself**, not to any product repo. Same discipline the OS
demands of product work: every item has a size, a reason, and either an owner
decision or a next action. An item with neither is an idea, not a backlog entry —
park it at the bottom or delete it.*

*Reviewed at the monthly roadmap check ([rituals-and-metrics.md](rituals-and-metrics.md)).
Adopted items graduate to the quarterly subtraction inventory with a retest date.*

**last_verified:** 2026-08-27

## ID scheme

| Prefix | Meaning |
|---|---|
| **B*n*** | **Backlog item** — a change to make. Numbered in the order raised, never reused after completion. |
| **R*n*** | **Risk** — a standing condition to watch. Not work; it either persists or is retired. |

These are **local to this file** — unlike `C1–C37`, which are constitutional and
cited repo-wide ([constitution.md](constitution.md)), and unlike `S0–S9`, which
are lifecycle stages. If you see `B4` in a commit message or a ticket, it means
this file. Completed items move to **Done** at the bottom with their commit SHA
rather than being deleted — an ID that stops resolving is worse than a long file.

---

## Needs a decision from you (blocked)

### B1 — Two methodologies are active at once ⚠️
**Size:** XS to decide, S to act · **Raised:** 2026-08-27

`superpowers` is a **global** OpenCode plugin (`~/.config/opencode/opencode.json`),
so it runs in every session. It is not a skill pack — it is *"a complete software
development methodology for your coding agents."* Architect OS is also a complete
methodology. Where they disagree, the agent silently blends them:

| | superpowers | Architect OS |
|---|---|---|
| Work unit | 2–5 minute tasks | ticket ≤1 day, PR ≤400 lines |
| Concurrency | subagent per task, by default | WIP ≤2; multi-agent is the 20% case |
| Review | its own two-stage | converge → human → cross-family (C36) |

They agree in spirit — both spec-first, TDD, human-gated — which is precisely why
the conflict is easy to miss.

**Decision needed:** keep Architect OS primary and demote superpowers to a skill
library, or genuinely switch. Either is defensible; running both is not.
**Next action once decided:** record the choice in `templates/AGENTS.md` guidance
and in each repo's AGENTS.md.

---

## Ready (no decision needed, just effort)

### B2 — Self-review output contract (C22 form, not just content)
**Size:** S · **Source:** `ayghri/i-have-adhd` · **Est:** 20 min

C22 specifies what a self-review must *contain* (files changed, constitution
compliance, uncertainty, deviations) but nothing about **form**. A self-review
that buries "I touched a fourth file" in paragraph three defeats its purpose —
you are scanning for deviations under a 10-minute budget.

Borrow: **deviations first**, numbered, **cap lists at 5**, no filler, state
problems plainly. Applies equally to the `converge` report.
**Touches:** `constitution.md` (C22), `github/code-review-checklist.md`,
`skills/converge/SKILL.md`.
**Highest value-per-minute item on this list** — it compounds over every PR.

### B3 — Design dials in AGENTS.md
**Size:** S · **Source:** `Leonxlnx/taste-skill` · **Est:** 15 min

taste-skill expresses subjective preference as three named 1–10 knobs
(`DESIGN_VARIANCE`, `MOTION_INTENSITY`, `VISUAL_DENSITY`). Three integers beat
three paragraphs: less context, no ambiguity, and *tunable* rather than
rewritable. Add to the S3 section of `templates/AGENTS.md`; consider whether the
pattern generalises beyond design.

### B4 — Within-ticket atomization guidance for S6
**Size:** S · **Source:** `obra/superpowers` · **Est:** 20 min

S6 is deliberately a black box ("fresh session, narrow context, TDD"). Superpowers'
**2–5 minute tasks with exact file paths and verification steps** is a discipline
*inside* one ticket — no conflict with ticket sizing if framed that way.
**Blocked by B1** — don't borrow from a methodology you may be removing.

### B5 — Install ui-ux-pro-max for S3
**Size:** XS · **Source:** `nextlevelbuilder/ui-ux-pro-max-skill` · **Est:** 5 min

S3 Design is the thinnest stage in the OS — "fill design-brief.md, use Figma or
prototype," with no design intelligence at all. This ships 192 industry rules,
192 palettes, 74 font pairings, and per-industry anti-patterns. **Install, do not
rebuild.** Its production-resilient checklist (text reflow, focus states,
reduced-motion) is worth lifting into the S3 exit gate regardless.

Architectural note: a BM25-searchable CSV knowledge base the agent queries on
demand is the same just-in-time-retrieval pattern as graphify-for-code
([Layer 0](repo-memory.md)) — a lesson, not a thing to build.

### B6 — Model-pricing duplication
**Size:** XS · **Raised:** 2026-08-27

Model prices appear in both `models-cost-quality.md` (full landscape) and
`harness-matrix.md` (escape-hatch summary). They **agree today** — both were
updated on 2026-08-27 — but this is the same shape as the subscription table
(`caba5ba`) and the label list (`3b8758c`) that had already drifted. Decide a
canonical home and make the other a pointer, before the 2026-10-19 re-verify.

### B7 — Skill-context audit
**Size:** S · **Raised:** 2026-08-27

`~/.config/opencode/skills/` holds **654 skills**. Model-invoked skills keep their
descriptions in context; user-invoked ones cost nothing until called. Measure the
split before assuming it's fine — this is a direct Layer 0 concern, and the
answer is a number, not an opinion.

---

## Design work (needs a spec before it can be built)

### B8 — `S-1 Excavate`: the brownfield on-ramp ⭐
**Size:** L — decompose before building · **Raised:** 2026-08-27

**The gap:** this OS is greenfield-biased. Every stage assumes you start from an
idea and produce docs as a *byproduct* of building. It says almost nothing about
the far more common case: **an existing codebase with no docs.** Today the only
guidance is one line in [adoption-plan.md](adoption-plan.md) ("have Claude draft
AGENTS.md from the code; you correct") and `improve-codebase-architecture` at S4.

**Proposed:** a stage that runs *before* S0 for existing repos and produces the
doc set greenfield gets for free.

#### What is and isn't derivable — the load-bearing distinction

| Derivable from code | Not derivable, ever |
|---|---|
| Structure, modules, dependencies, call graph | **Why** any of it is that way |
| Entry points, API surface, routes, schemas | Alternatives considered and rejected |
| Domain language (entity + type names) | Business context, users, success metrics |
| What the code does — flows, states | **Non-goals** — what was deliberately not built |
| Test coverage map | Which oddities are deliberate vs accidental |
| Conventions (from lint config, patterns) | Priorities, kill criteria |

This is exactly the [ADR / architecture.md](templates/architecture.md) split:
**`architecture.md` is derivable because it answers *what*; ADRs are not, because
they answer *why*.** Excavation can reconstruct roughly the descriptive 70% of
the doc set and none of the decisions. Any workflow that claims otherwise is
fabricating.

#### Sketch — five phases

1. **Map** *(mechanical, cheap model)* — graphify / repo-map the tree: structure,
   dependencies, entry points. → `architecture.md` skeleton + `repo-graph.json` seed.
2. **Describe** *(mid model, per module, fresh session each)* — what each module
   does, its public surface, how data flows. → layer table, domain language candidates.
3. **Interrogate** *(frontier model + **you**)* — the phase that makes this work.
   The agent **cannot know why**, but it *can* detect **where a decision was
   made**: an unusual library where a standard exists, a hand-rolled implementation,
   an oddly-enforced boundary, a "don't do X" comment, a workaround with no
   explanation. Each is an **ADR-shaped hole**. The agent surfaces them as
   questions; only you can answer. → ADR candidates.
4. **Verify** *(converge-style)* — every derived claim cites `file:line`. A claim
   that cannot cite is deleted, not softened.
5. **Stamp provenance** — everything derived carries `derived_from: <git-sha>` and
   `confidence: derived`. It is a **hypothesis until confirmed**, exactly as
   [memory-freshness-protocol.md](memory-freshness-protocol.md) already requires
   of stale memory. Confirmation is a human act.

#### The four failure modes to design against

1. **Fabricated rationale** 🔴 — the worst. An agent invents a plausible "why"
   for a decision nobody made, it lands in an ADR, and every future agent treats
   it as settled law. This is memory poisoning at the source. Mitigation: phase 3
   produces *questions*, never answers; an unanswered question stays `needs-decision`.
2. **Documenting bugs as intent** 🟠 — the code does X, so the doc says "the
   system shall X." **EARS makes this worse**, not better: `shall` language
   canonises observed behaviour as specification. Mitigation: derived requirements
   are written as *observations* ("the system currently does X"), and only become
   `shall` after you confirm X is wanted.
3. **Freezing tech debt as spec** 🟠 — documenting the current state as the
   desired state. The workaround becomes the contract.
4. **Context rot at scale** 🟠 — a large repo cannot be excavated in one session.
   Per-module fresh sessions (phase 2) and filesystem output, not a single
   mega-context. Standard [Layer 0](repo-memory.md) discipline.

#### Before building
Write an FSD. Pick one **real repo you know well** as the test — you can only
grade the output where you already know the truth. `~/projects/vyasan.design`,
`caveman`, or one of the `need-*` repos are candidates.

---

## Standing risks (not improvements, but tracked here so they aren't forgotten)

### R1 — This repo has no remote ⚠️
Every commit in this doc set exists on **one disk**. The `alfas-hancod` remote is
gone and `gh` now authenticates as `Devalfaz`. Same exposure applies to
`~/projects/momentum`, which holds a complete v0.1 FSD and zero backups.

### R2 — Research snapshot expires 2026-10-19
The July research and everything built on it. `harness-matrix.md` confidence tiers
tell you what to re-fetch first: ❔ before 📣 before ✅. The August file
(`research/research-2026-08-context-engineering.md`) re-verifies 2026-11-26.

---

## Parked

- **Momentum** — parked 2026-08-26 at the S5→S6 boundary. FSD-001 complete,
  Ticket 1 (SwiftData models + shared target) is the resume point.
- **Multi-tab "AI solution kit" app** — killed 2026-08-26. The category is
  commoditised (Monica, Team-GPT, Aizolo), and the user's answer to
  "for me or for others?" was *for me*, which removes the product case entirely.
  Recorded so it isn't re-proposed.

---

## Done

*Completed items stay here with their commit SHA so old references still
resolve. IDs are never reused.*

| ID | What | Landed |
|---|---|---|
| — | *(nothing completed yet)* | — |
