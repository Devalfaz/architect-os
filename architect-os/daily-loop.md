# The Daily Operating Loop

*One page. Print it. This is what you actually do each day.*

---

## Morning (30 min)

### Memory pull (5 min)
Read yesterday's dumps. Scan AGENTS.md — `last_verified > 7 days ago`? Flag for weekly distill.

### Delivery board scan (5 min)
Open GitHub Projects "Agent Queue" view. What's `ai:ready` and unblocked? Pick 1–2 tickets. Check: any PRs in Review needing your review? Do them first.

### Launch agents (10 min)
For each chosen ticket: `claude` in repo directory. Context: "Implement issue #NNN. Context: AGENTS.md, the FSD section linked, and files listed in the implementation plan." WIP limit: 2 concurrent agent runs.

### Review waiting PRs (10 min)
PRs that came in overnight. [10-minute rubric](pr-review-rubric.md). Post review. If fixes needed: leave review, agent addresses, you resolve later.

---

## Midday

### Between tasks
Check agent progress. Agent done → PR opened → quick scan (directionally right?). Don't review yet — let CI run and CodeRabbit post first. If agent struggling (3+ test-fix cycles): kill session, re-plan ticket.

**Rule:** Review once per PR, not continuously.

---

## Afternoon (10 min)

### Close completed work
Any PRs approved today? Squash merge. Deployed? Quick smoke check on live. Verify issues auto-closed.

### Write memory dump (5 min)
For every merged PR today, one dump entry using [template](templates/memory-dump.md). What changed, decisions, surprises, hallucinations caught, graph delta. Save to `memory/dumps/YYYY-MM-DD.md`. One file per day. No coding today = no dump.

### Plan tomorrow (2 min)
Highest-priority `ai:ready` ticket? Blockers? Unblock or create `needs-decision` issue.

### Shutdown (1 min)
Kill any running agent sessions (don't leave overnight — stale context = bad code). Close terminals.

---

## Weekly (30 min, Sunday evening or Monday morning)

### Memory distill
Read all week's dumps. Run `/graph-update` → graph delta PR. Review and merge. Verify AGENTS.md freshness. Update `docs/agents/project-gotchas.md`. Check `docs/research/` for expired.

### Sprint scan
Delivery board: what's Done? What's stalled? Anything In Progress >1 week without PR → investigate. Count: tickets merged this week vs last.

### CI health
Any red builds on main? Flaky tests? Quarantine with `test.skip` + ticket (C20).

---

## Monthly (1 hour, last day of month)

### Metrics review
Throughput, bounce rate, fix-round distribution, CodeRabbit FPR, cost totals, spec deltas, stale memory % — from [rituals-and-metrics.md](rituals-and-metrics.md).

### Retrospective
Run `/retro`. Output: `docs/agents/retro-YYYY-WW.md`. What worked, what didn't, what surprised.

### Roadmap check
Ideas with triggered kill criteria → kill. P3/P4 to promote? Prototypes >2 weeks → promote or delete. Tech-debt tickets scheduled.

### Model routing review
Re-check [models-cost-quality.md](models-cost-quality.md) against current prices. Adjust budgets from actual cost data. New model dropped?

---

**One-sentence version: Morning: review PRs, launch 1–2 agents. Afternoon: dump memory, plan tomorrow. Weekly: distill memory. Monthly: retro + metrics. Your time goes to gates — not transcripts.**
