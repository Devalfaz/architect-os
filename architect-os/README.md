# Architect OS

*A personal operating system for AI-assisted software engineering. Spec-driven, memory-backed, human-gated. You architect and decide; agents execute and propose.*

## The one-paragraph version

Every piece of work flows **idea → BRD → PRD/FSD → design → architecture/ADRs → file-level tickets → narrow-context agent runs → human review first, AI review second → tested squash merge → memory update**. Specs are the source of truth, GitHub Issues are the execution system, and a layered repo memory (AGENTS.md → docs tree → knowledge graph → session dumps) makes every cycle smarter than the last. Your effort concentrates at the gates — spec approval, plan review, diff review — and almost nowhere else.

## Seven principles

1. **The spec is the source of truth.** Code that diverges from spec produces a spec delta, never silent drift. Prompt quality is a symptom; spec quality is the cause.
2. **Narrow context beats big context.** Each agent run gets one ticket's worth of context. More context makes agents worse *and* more expensive — it's a quality rule that happens to save money.
3. **Human review comes first.** You review every diff with the [rubric](pr-review-rubric.md) *before* reading AI reviews, to stay unanchored. AI reviewers are your second and third pass, not your first.
4. **Plans are reviewed harder than diffs.** Ten minutes on a file-level plan saves an hour on a wrong diff. If implementation goes sideways twice, the plan was wrong — restart, don't correct a third time.
5. **Memory has a freshness date.** Every doc, graph node, and note carries `last_verified`. Unverified memory is a hypothesis, not a fact ([protocol](memory-freshness-protocol.md)).
6. **Small everything.** Tickets ≤1 day. PRs ≤400 lines. Fix loops ≤2 rounds. WIP ≤2 agent runs. The limits are the system.
7. **Agents propose, you decide.** Agents never merge, never close issues, never mark their own work ready. Every irreversible action passes through you.

## Reading order

| # | Doc | What it gives you |
|---|---|---|
| 1 | [lifecycle.md](lifecycle.md) | The end-to-end workflow, stage by stage, with gates |
| 2 | [daily-loop.md](daily-loop.md) | The one-page loop you actually run each day |
| 3 | [constitution.md](constitution.md) | The rules every agent must follow (C1–C37, severity-tagged) |
| 4 | [harness-matrix.md](harness-matrix.md) | Which tool for which layer, and why — cited |
| 5 | [skills-catalog.md](skills-catalog.md) | The repeatable agent workflows and when to fire each |
| 6 | [github-setup.md](github-setup.md) | Labels, issue forms, rulesets, Projects — the execution system |
| 7 | [repo-memory.md](repo-memory.md) | The four-layer memory architecture |
| 8 | [tech-stack.md](tech-stack.md) | Default stack + when to deviate |
| 9 | [pr-review-rubric.md](pr-review-rubric.md) / [review-workflow.md](review-workflow.md) | The two-stage review system |
| 10 | [models-cost-quality.md](models-cost-quality.md) / [cost-control.md](cost-control.md) | Model routing and budgets |
| 11 | [rituals-and-metrics.md](rituals-and-metrics.md) | Daily/weekly/monthly/quarterly cadence, the numbers that matter |
| 12 | [multi-agent.md](multi-agent.md) | When one agent isn't the right shape — decision tree, patterns, failure modes |
| 13 | [failure-modes.md](failure-modes.md) / [failure-recovery-playbook.md](failure-recovery-playbook.md) | What goes wrong and what to do about it |
| 14 | [adoption-plan.md](adoption-plan.md) | Default/light/heavy profiles, 30/60/90, worked walkthroughs |

## July 2026 research update

Six parallel internet-research passes against primary sources, plus a master synthesis identifying what changed since the mid-2025 baseline, what the existing research got right, and what is now stale or missing. The synthesis is the entry point; the detailed reports are the evidence layer.

| File | Scope | Priority findings |
|---|---|---|
| [insights-and-issues-2026-07.md](insights-and-issues-2026-07.md) | **Master synthesis — start here** | 20 issues prioritized Critical/High/Medium/Low; 5 things to do this quarter |
| [research-2026-07-tools.md](research-2026-07-tools.md) | AI coding agents & tools | Claude Code async primitives shipped; new entrant wave (Devin, Replit Agent, Lovable, Bolt, v0, Windsurf, Trae, Zed AI, Cline, Continue, OpenCode); three-layer code review stack |
| [research-2026-07-specs.md](research-2026-07-specs.md) | Spec-driven development | GitHub Spec Kit 122k stars; AWS Kiro; BMad v6 scale-adaptive; Anthropic March 2026 harness-design post gives empirical evidence |
| [research-2026-07-memory.md](research-2026-07-memory.md) | Memory & context engineering | Context engineering as a discipline (Anthropic Sep 2025); Mem0/Letta/Zep; AGENTS.md content mix pruning; automatic invalidation |
| [research-2026-07-review.md](research-2026-07-review.md) | AI code review | Correlated-vendor problem (Greptile auditor post); runtime validation (TREX); 27.6% of merged PRs AI-authored; reviewer-independence rule |
| [research-2026-07-multi-agent.md](research-2026-07-multi-agent.md) | Multi-agent orchestration | Anthropic multi-agent research system (+90.2% over single-agent, 15× chat tokens); Claude Code agent teams/worktrees/`claude -p` batch; 12 multi-agent failure modes |
| [research-2026-07-update.md](research-2026-07-update.md) | Cost optimization & failure modes | Batch APIs, LiteLLM/OpenRouter, Anthropic automatic caching mode, OpenAI `prompt_cache_key` 15 RPM ceiling; 15 new failure modes beyond the existing 12 |

## Repository layout (this document set)

```
architect-os/
├── README.md                      ← you are here
├── lifecycle.md                   ← the workflow spine
├── daily-loop.md                  ← 1-page operating loop
├── constitution.md                ← agent rules C1–C37, severity-tagged
├── harness-matrix.md              ← tool comparison & assignments
├── skills-catalog.md              ← agent workflow library
├── github-setup.md                ← execution-system narrative
├── repo-memory.md                 ← memory architecture
├── memory-freshness-protocol.md   ← anti-staleness rules
├── tech-stack.md                  ← default stack + variants
├── pr-review-rubric.md            ← human review, first pass
├── review-workflow.md             ← full two-stage review pipeline
├── models-cost-quality.md         ← model landscape & routing
├── cost-control.md                ← budgets, arbitrage, kill switches
├── rituals-and-metrics.md         ← cadences & measurement
├── multi-agent.md                 ← the 20% case: fan-out patterns & decision tree
├── failure-modes.md               ← the 18 ways this goes wrong
├── failure-recovery-playbook.md   ← symptom → action
├── adoption-plan.md               ← profiles, 30/60/90, walkthroughs
├── templates/                     ← BRD, PRD, FSD, ADR, AGENTS.md, dumps…
├── skills/                        ← OS-native skill specs (converge, …)
├── github/                        ← issue forms, labels, ruleset, CI, .coderabbit.yaml (copy into repo)
└── memory/                        ← repo-graph schema (v1.1, temporal) + example
```

## What a product repo looks like once installed

```
your-app/
├── AGENTS.md                      ← agent entrypoint, ≤150 lines (CLAUDE.md symlinks here)
├── .claude/skills/                ← repo-specific skills
├── .github/                       ← copied from architect-os/github/
├── docs/
│   ├── product/                   ← ideas, BRDs, PRDs, design briefs
│   ├── specs/                     ← FSDs (implementation contracts)
│   ├── adr/                       ← numbered decision records
│   ├── architecture/              ← architecture.md + diagrams
│   ├── research/                  ← spikes, with expiry dates
│   └── agents/                    ← agent-facing guides (verified APIs, gotchas)
├── memory/
│   ├── repo-graph.json            ← the knowledge graph
│   └── dumps/                     ← session memory dumps
└── src/ …
```

## Setup checklist (new machine / first time)

- [ ] **Pick your stack** ([the two stacks](adoption-plan.md)): **default** — install Claude Code, sign in on a Max plan ([cost rationale](cost-control.md)); **open-frontier** — install OpenCode plus DeepSeek/OpenRouter API keys ([setup](harness-matrix.md))
- [ ] Install the C36 cross-family reviewer — Codex CLI (default stack) or GLM 5.2 via OpenRouter (open-frontier stack) — plus `gh` CLI, authenticated
- [ ] Install skills: Matt Pocock's set + OS-native skills into `~/.claude/skills` ([catalog](skills-catalog.md))
- [ ] Put `~/.claude/skills` under git — versioning is mandatory ([rituals](rituals-and-metrics.md))
- [ ] Install the CodeRabbit GitHub App on your account
- [ ] Enable an async XS lane if using one (Claude Code on the web, Copilot coding agent, or Codex Web — C37 applies)

## Setup checklist (each new repo)

- [ ] `git init` + create GitHub repo, squash-merge only
- [ ] Copy `architect-os/github/` → `.github/`; run label sync
- [ ] Import [branch ruleset](github/rulesets/main-ruleset.json) on `main`
- [ ] Create the **Delivery** project with Stage/Size/Priority/Agent fields ([guide](github/projects-setup.md))
- [ ] Write `AGENTS.md` from the [template](templates/AGENTS.md); `ln -s AGENTS.md CLAUDE.md`
- [ ] Create `docs/` + `memory/` trees; commit an empty `repo-graph.json` with the [schema](memory/repo-graph.schema.json)
- [ ] Write ADR-0001 (stack) and ADR-0002 (architecture style) — even if they just say "defaults per tech-stack.md"
- [ ] CI green on a hello-world PR before any feature work

Full command sequence: [github-setup.md](github-setup.md).
