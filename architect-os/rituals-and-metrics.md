# Rituals & Metrics — The Cadence and the Numbers

*What gets measured gets managed. These rituals keep the OS from becoming shelfware.*

---

## The three horizons

| Horizon | Ritual | Time | Purpose |
|---|---|---|---|
| Daily | Morning launch + evening dump | 40 min | Keep pipeline flowing |
| Weekly | Memory distill + sprint scan | 30 min | Prevent memory staleness |
| Monthly | Retro + metrics + roadmap | 1 hour | Course-correct the system |

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
| Second AI unique catches | Value of second opinion | >0.5/review |
| Spec deltas filed | Spec quality | Trending down |
| Stale memory nodes | Graph maintenance | <10% |
| Total monthly cost | Budget | vs target |
| Cost per ticket | Efficiency | Trending down |
| Sessions killed | Agent failure rate | <10% |

### The metric that matters most: spec deltas per feature

When spec deltas rise: bounce rate rises, fix rounds rise, cost per ticket rises. When they fall: implementation becomes mechanical, review finds fewer surprises. The spec is the ceiling.

### Retro
Run `/retro`. Three questions: what worked, what didn't, what surprised. Output → action: process friction → edit skill, tool friction → update harness-matrix, spec issue → review S2 gate, cost surprise → review routing.

### Roadmap check
Review `docs/product/ideas/` — kill criteria triggered? P3/P4 to promote? Prototypes >2 weeks old? Tech-debt tickets scheduled?

### Model routing review
Re-check [models-cost-quality.md](models-cost-quality.md) against current prices. Update per-ticket budgets from actual data. New model released?

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
