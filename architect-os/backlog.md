# OS Backlog — Pending Improvements

*Changes to the OS **itself**, not to any product repo. Same discipline the OS
demands of product work: every item has a size, a reason, and either an owner
decision or a next action. An item with neither is an idea, not a backlog entry —
park it at the bottom or delete it.*

*Reviewed at the monthly roadmap check ([rituals-and-metrics.md](rituals-and-metrics.md)).
Adopted items graduate to the quarterly subtraction inventory with a retest date.*

**last_verified:** 2026-08-27

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
