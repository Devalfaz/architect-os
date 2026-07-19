# Rituals & Metrics — The Cadence and the Numbers

*What gets measured gets managed. These rituals keep the OS from becoming shelfware.*

---

## The four horizons

| Horizon | Ritual | Time | Purpose |
|---|---|---|---|
| Daily | Morning launch + evening dump | 40 min | Keep pipeline flowing |
| Weekly | Memory distill + sprint scan | 30 min | Prevent memory staleness |
| Monthly | Retro + metrics + learnings sweep | 1 hour | Course-correct the system |
| Quarterly | Subtraction ritual + research re-verification | 1 hour | Delete mechanisms that stopped earning their keep |

---

## Daily rituals

See [daily-loop.md](daily-loop.md) for full walkthrough. Non-negotiables:
1. Review waiting PRs before launching new agents
2. Write memory dumps on days you ship
3. Kill running agents at end of day

---

## Weekly rituals (30 min)

### Memory distill
Read all week's dumps. Run `/graph-update` → graph delta PR. Review and merge. Verify AGENTS.md `last_verified ≤7 days`. Update `project-gotchas.md` from dump surprises and hallucinations. Check `research/` for expired.

### Sprint scan
Delivery board scan. Stale In Progress? M tickets that became L? Weekly merge count vs last week.

### CI health
Red builds on main? Flaky tests → quarantine (C20).

### Skill changelog
Skill edits → record in `docs/agents/skill-changelog.md`: date, skill, what changed, why. Commit to `~/.claude/skills/` git repo.

---

## Monthly rituals (1 hour)

### Metrics tracking (spreadsheet: `docs/agents/metrics.md`)

| Metric | What it measures | Target |
|---|---|---|
| Tickets merged | Throughput | Track trend |
| Avg ticket size | Decomposition quality | M or smaller |
| Bounce rate | % PRs bounced to S5 | <10% |
| Fix rounds per PR | Review efficiency | Avg <1.5 |
| PR review time | Are you keeping up? | <24h |
| CodeRabbit FPR | Signal quality | <30% |
| CodeRabbit address rate | Is review signal landing? | ≥50%, trending up |
| Second AI unique catches | Value of second opinion | >0.5/review |
| Spec deltas filed | Spec quality | Trending down |
| Stale memory nodes | Graph maintenance | <10% |
| Total monthly cost | Budget | vs target |
| Cost per ticket | Efficiency | Trending down |
| Sessions killed | Agent failure rate | <10% |

### The metric that matters most: spec deltas per feature

When spec deltas rise: bounce rate rises, fix rounds rise, cost per ticket rises. When they fall: implementation becomes mechanical, review finds fewer surprises. The spec is the ceiling.

### Retro
Run `/retro`. Three questions: what worked, what didn't, what surprised — plus the trust-drift check: **"Did I approve anything this month where I deferred to the AI reviewer against my own judgment?"** (failure mode #15). Output → action: process friction → edit skill, tool friction → update harness-matrix, spec issue → review S2 gate, cost surprise → review routing.

### CodeRabbit learnings sweep (the constitution's learning loop)
On a recent PR, run `@coderabbitai emit path instructions` — it sweeps the last week of review feedback into suggested `.coderabbit.yaml` edits. Review the suggestions like code: merge the good ones into [.coderabbit.yaml](github/.coderabbit.yaml), and promote anything that should bind *all* agents (not just CodeRabbit) into [constitution.md](constitution.md) as an ADR amendment. This is how the rulebook learns from review outcomes — without it, the same false positives recur and address rate decays (failure mode #16; tripwire: address rate <35% for two consecutive weeks → prune rules, don't push harder).

### Roadmap check
Review `docs/product/ideas/` — kill criteria triggered? P3/P4 to promote? Prototypes >2 weeks old? Tech-debt tickets scheduled?

### Model routing review
Re-check [models-cost-quality.md](models-cost-quality.md) against current prices. Update per-ticket budgets from actual data. New model released?

---

## Quarterly rituals (1 hour)

### The subtraction ritual

Every mechanism in this OS encodes an assumption about where agents are weak. Models improve; the assumption rots; the mechanism stays — unless you delete it. (Anthropic deleted their own harness's sprint decomposition the week a stronger model shipped. The lesson generalizes: additions are hypotheses, not heirlooms.)

Once per quarter, walk the mechanism inventory. For each mechanism, ask: **what evidence says this earned its keep this quarter?** (a metric that moved, a catch that mattered, a cost that dropped). Then pick at least one mechanism and **disable it for one week**. If output quality doesn't measurably drop — bounce rate, fix rounds, address rate, defect escape — delete it permanently. If it does, reinstate and record why.

Seeded inventory (first retest: **2026-10-19**, aligned with the research snapshot expiry):

| Mechanism | Added | Evidence to check | Retest |
|---|---|---|---|
| `converge` gate (S7) | Jul 2026 | Conformance gaps caught that rubric missed? | 2026-10 |
| C36 cross-family second opinion | Jul 2026 | Unique catches by the cross-family reviewer (>0.5/review?) | 2026-10 |
| Severity batching (🟡/🔵) | Jul 2026 | Address rate trend | 2026-10 |
| Size classification (S1) | Jul 2026 | XS/S tickets shipping without over-spec? | 2026-10 |
| Plan persistence (S5) | Jul 2026 | Any bounce-back-to-S5 that used the frozen plan? | 2026-10 |
| Temporal memory fields + write-time invalidation | Jul 2026 | Stale-node count; any mid-week invalidation? | 2026-10 |
| Weekly memory distill | (original) | Still finding things the graph got wrong? | standing |
| Second-opinion AI review at all | (original) | Unique catches vs. the time it takes | standing |

New mechanisms added from now on enter this table with a retest date **at adoption time**. A mechanism without a retest date is process accretion.

### Research re-verification

Research snapshots carry `expires` dates. When one lapses: re-fetch the load-bearing facts (pricing, tool capabilities, model landscape) before trusting any recommendation built on them, or mark the file superseded. The harness matrix's confidence tiers (✅/📣/❔) tell you which facts need re-fetching first — ❔ before 📣 before ✅.

---

## What NOT to do

| Don't | Because |
|---|---|
| Skip daily dump — "small day" | Small days become big gaps |
| Skip weekly distill — "graph is fine" | Memory decay is invisible until wrong |
| Skip monthly retro — "nothing went wrong" | Things went wrong you didn't notice |
| Let metrics sheet go stale | Can't improve what you don't measure |
| Let skill edits go unversioned | Can't roll back bad edit without git |
| Let graph become "auto-generated only" | Auto noise vs hand signal. Weekly distill is the human filter |

---

## Metrics sheet template

```markdown
# Metrics — 2025-W03 (Jan 13–19)

| Metric | Value | Trend |
|---|---|---|
| Tickets merged | 8 | ↑ (7) |
| Avg ticket size | S | — |
| Bounce rate | 0% | — |
| Fix rounds avg | 1.2 | ↓ (1.5) |
| PR review time (avg) | 8h | ↓ (14h) |
| CodeRabbit FPR | 25% | ↓ (30%) |
| CodeRabbit address rate | 52% | ↑ (41%) |
| 2nd AI unique catches | 3 | — |
| Spec deltas filed | 1 | ↓ (2) |
| Stale memory | 3% | — |
| Total cost | $242 | ↑ ($230) |
| Sessions killed | 1 (5%) | ↓ (10%) |

## Retro summary
- Worked: grill-with-docs caught 3 edge cases agent would have missed
- Didn't: Sonnet struggled with 5-file refactor — should have used Opus. Updated routing.
- Surprised: CodeRabbit caught SQL injection in migration
```

---

*Rituals are the difference between adopting a workflow and living it. Daily loop = engine. Weekly distill = transmission. Monthly retro = steering wheel. Skip one and the car drifts.*
