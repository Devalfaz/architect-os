# 02 — Running the Loop

*What you actually do. Three rhythms: the day, the feature, the bug. Canonical sources: [daily-loop.md](../daily-loop.md), [lifecycle.md](../lifecycle.md), and the three worked walkthroughs in [adoption-plan.md](../adoption-plan.md).*

---

## The day (canonical: [daily-loop.md](../daily-loop.md))

**Morning (10 min):** open the Delivery project's "Needs me" view → triage overnight agent PRs and new issues → pick 1–3 tickets → glance at memory freshness flags.

**Work blocks (≤3/day):** pick an `ai:ready` ticket → 5-minute plan sanity check → launch the narrow agent run (fresh session: AGENTS.md + ticket + FSD section + planned files, *nothing else*) → **while it runs, review the previous run's PR** (the pipelining rule: you review run N−1 while N executes) → merge or bounce. WIP limit 2, always.

**Evening (10 min):** [memory dump](../templates/memory-dump.md) for anything merged → log costs → note one workflow friction in the retro file.

The feel to calibrate against: you are *never waiting* on an agent (something is always reviewable) and *never rushing* a review (nothing merges without the rubric). If either breaks, your WIP is wrong.

## The feature (canonical: [lifecycle.md](../lifecycle.md) + [walkthrough 1](../adoption-plan.md))

1. **Capture** the idea (`templates/idea.md`, 5 min) — kill criteria included.
2. **Size it** at S1 (XS/S/M/L table in lifecycle.md) — the size decides how much of steps 3–5 exist.
3. **Spec it** (S2): agent drafts PRD/FSD, *grills you* with questions, `grill-with-docs` verifies every external API claim. Your gate: every acceptance criterion mechanically testable; edge-case table filled.
4. **Plan it** (S5): `to-tickets` → GitHub Issues with file-level plans; **plan persisted** (`docs/specs/<slug>/plan.md` for M+). Your gate: this is your highest-leverage hour — a plan bug caught here costs 10× less than in a diff.
5. **Run it** (S6): one ticket = one branch = one fresh session. TDD. Agent posts self-review.
6. **Review it** (S7): converge gate (M+) → your [10-minute rubric](../pr-review-rubric.md) *before reading any AI comments* → CodeRabbit + cross-family second opinion (C36) → ≤2 fix rounds.
7. **Ship & learn** (S8–S9): squash merge, smoke-check prod yourself, evening dump, Friday distill.

## The bug (canonical: [walkthrough 2](../adoption-plan.md))

Bug issue ([form](../github/ISSUE_TEMPLATE/bug.yml)) → severity + `regression-test-required` → `diagnosing-bugs` skill: the agent must **reproduce in a failing test before touching code** (the failing test *is* the diagnosis) → if the fix has scope options, the agent proposes and *you* choose → surgical fix + the regression test → rubric (2 min at this size) → merge. The regression test is permanent armor; the dump records the gotcha.

## When it goes sideways (canonical: [failure-recovery-playbook.md](../failure-recovery-playbook.md))

The one rule that matters: **restarting with a sharper spec beats correcting a confused agent more than twice.** Two failed correction rounds → stop, kill the session, extract the lesson into the ticket/spec, relaunch fresh (C24). Everything else is a row in the playbook's symptom table.

## The week and the quarter (canonical: [rituals-and-metrics.md](../rituals-and-metrics.md))

Friday 30 min: distill dumps → graph + docs ([freshness protocol](../memory-freshness-protocol.md)), metrics glance (first-pass acceptance, fix rounds, address rate, cost/merged PR), prune the `ai:ready` queue. Monthly: retro + constitution learning loop. Quarterly: the **subtraction ritual** — disable one mechanism for a week; if quality doesn't drop, delete it — plus re-verification of the research snapshot (next due: **2026-10-19**).
