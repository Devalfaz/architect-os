# Adoption Plan — Profiles, 30/60/90, Walkthroughs

*How to actually start using this without drowning in process on day one.*

## The three profiles

The OS is one system with three dial settings. The **stages never change; the ceremony per stage does.**

### Default (solo professional — recommended)

Everything in [lifecycle.md](lifecycle.md) as written. BRD may be half a page; FSD and ticket plans are never skipped for M-sized work. This is the profile the rest of the documentation assumes.

### Lightweight (hacking, prototypes, tiny fixes)

For throwaway explorations and XS/S changes to established repos.

| Kept (non-negotiable) | Dropped |
|---|---|
| One issue = one branch = one PR | BRD, PRD (intent goes in the issue body) |
| File-level plan *in the issue body* | Separate FSD doc (acceptance criteria live in the issue) |
| Human diff review with the rubric's top 3 axes | AI second-opinion review (CodeRabbit alone suffices) |
| Tests before merge | Design brief |
| AGENTS.md + constitution | Graph updates (dump a note; distill only if the repo graduates) |

The trap to avoid: prototypes that quietly become products. The rule from [templates/idea.md](templates/idea.md) applies — a prototype that survives two weeks gets promoted to the default profile *or deleted*.

### Heavyweight (team / enterprise-grade)

Everything in Default, plus:

- **Role separation, BMAD-style**: distinct planning personas (analyst → PM → architect) with human sign-off between each; story files carry full context into dev ([harness-matrix.md](harness-matrix.md) covers when BMAD's machinery earns its weight).
- **Two human reviewers** on `main` (ruleset: 2 approvals + CODEOWNERS enforcement), merge queue on, deployment environments with required reviewers.
- **Compliance layer**: AI-assistance disclosure required on every PR (already in the [PR template](github/PULL_REQUEST_TEMPLATE.md)), audit log of agent sessions, SAST + dependency scanning as required checks, DORA metrics tracked, not just the personal metrics in [rituals-and-metrics.md](rituals-and-metrics.md).
- **Shared memory governance**: graph and docs edits go through PRs like code; a rotating "librarian" owns the weekly distill.

## The two stacks

The profiles above set *ceremony*; the stack sets *tools*. The two choices are orthogonal — Lightweight ceremony runs fine on either stack.

### Default stack — Claude Code + Anthropic

| Layer | Tool |
|---|---|
| Harness | Claude Code (terminal, IDE, Desktop, web) |
| Planning models (S2/S4/S5) | Opus 4.8 |
| Implementation (S6) | Sonnet 5 (⏰ intro pricing ends 2026-08-31) |
| Learning (S9) | Haiku 4.5 |
| Always-on review | CodeRabbit |
| Cross-family second opinion (C36) | Codex review (Copilot code review as alternative) |
| Async XS lane | Claude Code on the web / Routines, Copilot coding agent, Codex Web |
| Cost | ~$232–249/mo subscriptions ([cost-control.md](cost-control.md)) |

Choose when: you want the most mature agent loop (subagents, skills, hooks, Routines), already pay for Max, or rely on Claude-specific features. The workflow docs (lifecycle, cost-control, README checklist) assume this stack.

### Open-frontier stack — OpenCode + DeepSeek/OpenRouter

| Layer | Tool |
|---|---|
| Harness | OpenCode (Aider, Cline as alternatives) |
| Planning models (S2/S4/S5) | DeepSeek V4 Pro |
| Implementation (S6) | DeepSeek V4 Flash |
| Learning (S9) | Gemini 2.5 Flash |
| Always-on review | CodeRabbit |
| Cross-family second opinion (C36) | GLM 5.2 via OpenRouter (Llama 4 Maverick as alternative) |
| Async XS lane | OpenCode backgrounded, Codex Web |
| Cost | ~$32–65/mo ([harness-matrix.md](harness-matrix.md)) |

Choose when: cost discipline matters, you want zero vendor lock-in, or you accept a younger harness in exchange for 60–85% lower spend.

**Rules that hold on both stacks:** the lifecycle, the constitution, the review pipeline (C36 works on both — Claude authors → Codex reviews; DeepSeek authors → GLM/Llama reviews; Codex authors → claude-code-action or CodeRabbit), the GitHub execution system, the memory architecture, and the rituals. Pick **one stack per repo** and record the choice in AGENTS.md (or ADR-0001); don't mix stacks mid-ticket. Migration between stacks is cheap by design — every artifact in this OS is plain markdown.

## Tooling matrix

Full comparison with citations: [harness-matrix.md](harness-matrix.md). The one-glance version (**default stack** — the open-frontier mapping is the matrix's Part II):

| Layer | Default | Alternative | Skip when |
|---|---|---|---|
| Spec & planning | Claude Code + skills | Spec Kit (ceremony, agent-agnostic) | — |
| Implementation | Claude Code | Codex CLI (2nd opinion), Cursor (IDE feel) | — |
| Async small tickets | Copilot coding agent | Codex cloud | no GitHub org / rarely have XS tickets |
| AI PR review | CodeRabbit | Codex review, claude-code-action | lightweight profile: CodeRabbit only |
| Execution system | GitHub Issues + Projects | Linear (nicer UI, weaker agent hooks) | — |
| Memory | AGENTS.md + docs + graph | — (this layer has no product substitute) | never |

## 30 / 60 / 90

**Days 1–30 — install the skeleton, run the loop on one repo.**
- Week 1: machine + repo setup checklists in [README.md](README.md). Pick ONE active repo. Write its AGENTS.md and architecture.md (have Claude draft from the code; you correct — the corrections are the value).
- Weeks 2–4: run S5→S9 only (tickets → implement → review → merge → dump) on real work. Don't attempt BRD/PRD discipline yet. Goals: 15+ PRs through the full review pipeline; daily loop feels automatic; first weekly distills done.
- Exit criteria: first-pass acceptance ≥40%, zero merges without the rubric, memory dumps exist for every merge day.

**Days 31–60 — add the front half, tune the system.**
- Take one *new* feature through the full S0→S9 lifecycle, honestly, including the FSD grill.
- Bootstrap `memory/repo-graph.json` (agent-generated from the repo + your corrections), start using graph ego-loads in tickets.
- Start the metrics sheet ([rituals](rituals-and-metrics.md)); first monthly retro at day ~45; adjust skills based on retro — every friction becomes a skill edit, versioned.
- Add the second AI reviewer (Codex review or claude-code-action) and compare catch rates against CodeRabbit for a month.

**Days 61–90 — scale out and stress-test.**
- Roll the OS onto your remaining active repos (setup is now <1 hour each).
- Try one deliberately hard thing: the large-refactor walkthrough below, on a real refactor.
- Run the model-routing review against current prices ([models-cost-quality.md](models-cost-quality.md)); set per-ticket budgets from your own 60-day cost data, not guesses.
- Decision point at day 90: which ceremony earned its keep? Prune honestly — an unused template is workflow debt.

---

## Walkthrough 1 — New feature, idea to merged PR

Scenario: "users should be able to export their data as CSV."

1. **S0** — `docs/product/ideas/csv-export.md`: problem (users asking for data portability), evidence (3 support emails), kill criteria (if <5 requests by March, park it). 5 minutes.
2. **S1** — promote at triage. Claude Code, `brd` interview: who needs it (compliance-conscious admins), outcome (retention: exports correlate with trust), non-goals (no scheduled exports, no XLSX, v1 is sync-download only). Half a page. You approve.
3. **S2** — `to-spec`: PRD (FR-1 export button on settings, FR-2 all user-owned records, FR-3 ≤30s for 50k rows…), then FSD: flow diagram, edge cases (empty account, concurrent export, unicode in fields, 500k-row account → needs async job? **spec decision: cap at 100k rows in v1, error above**), API contract (`POST /api/export` → 202 + job id? No — v1 sync `GET /api/export.csv`, ADR-worthy simplification), Given/When/Then per FR. `grill-with-docs` verifies the CSV-streaming API you plan to use actually exists in the version you run. You gate: edge-case table honest? Criteria testable? Approve.
4. **S3** — one settings-page section; design brief maps it to existing shadcn components; empty/loading/error states specified. 20 minutes.
5. **S4** — no new architecture; one ADR line-item: "sync export capped at 100k rows; revisit async at demand" appended as ADR-0014. Data-model delta: none.
6. **S5** — `to-tickets` produces: #231 `feat: export service + streaming CSV endpoint` (M — files: `src/server/export/service.ts` new, `src/app/api/export/route.ts` new, `src/server/export/service.test.ts`), #232 `feat: settings UI export section` (S — depends on #231), #233 `chore: e2e export happy path + cap error` (S). You review the plans, fix one thing (service should reuse existing `serializeRow` util — the graph's ego-view of `src/server/` surfaced it). Label `ai:ready`.
7. **S6** — morning: launch Claude Code on #231 (fresh session, context = AGENTS.md + #231 + FSD §export + the 3 files). TDD: streaming test with 100k-row fixture first. While it runs, you review yesterday's PR. #232 goes to Copilot coding agent async — it's fully specified UI glue. WIP = 2. ✓
8. **S7** — #231 PR: self-review comment posted; you run the rubric (10 min: conformance ✓, architecture — flag: it added a new `csv` dependency; constitution C-rule requires an ADR line, bounce). Fix round 1: agent swaps to the stdlib-adjacent approach from the FSD. CodeRabbit finds a missing `Content-Disposition` escape; Codex review finds nothing new. You resolve threads, approve.
9. **S8** — squash merge (`feat: streaming CSV export (#231)`), Vercel deploys, you download an export from prod. Works. #232/#233 follow the same path.
10. **S9** — evening dump: "surprise: `serializeRow` didn't handle BOM; fixed in-place and noted. Graph delta: +node `export-service`, +edge `export-service depends_on serialize-row`." Friday distill updates the graph. Next export-related ticket starts smarter.

Elapsed architect time: ~3.5 h across three days. Agent-executed. Zero babysitting.

## Walkthrough 2 — Bug, report to regression test

Scenario: "login fails for emails with a `+` tag."

1. **Capture** — [bug.yml](github/ISSUE_TEMPLATE/bug.yml) issue #240: repro steps, expected/actual, severity=high, regression-test-required ☑ (it's in the template — this box is the whole point).
2. **Triage** — P1, `area:api`. No spec needed; the bug report *is* the spec, but the acceptance criterion is explicit: "user with `a+b@x.com` can log in; regression test proves it."
3. **Diagnose** — Claude Code, `diagnosing-bugs` skill: hypothesis-first, not fix-first. Agent reproduces in a test *before* touching code (constitution: failing test first — for bugs this is non-negotiable, the failing test is the reproduction). Finds: email normalization strips `+tag` before lookup but not before storage. Hypothesis confirmed by the failing test.
4. **Scope check** — the fix could be "normalize everywhere" (migration! risk!) or "stop stripping at lookup" (surgical). Agent proposes both with trade-offs; **you** decide (surgical now, migration ticket #241 filed for weekly triage). Decisions like this never belong to the agent.
5. **Implement** — 15-line diff + the regression test that currently fails → passes. PR links #240, notes the deliberate non-fix of stored data with a pointer to #241.
6. **Review** — rubric (2 min at this size), CodeRabbit passes it, merge. The regression test is now permanent armor: this bug class can never silently return.
7. **Learn** — dump: "gotcha: normalization is split across `auth/` and `users/` — graph edge added; candidate for #241's plan. `diagnosing-bugs` worked well; no skill changes."

Total: under an hour, most of it agent time. The system's contribution: the forced repro-test, the scope decision escalation, and the memory note that makes #241 cheaper.

## Walkthrough 3 — Large refactor without destroying the repo

Scenario: extract a tangled `lib/billing.ts` (2,400 lines, 40 call sites) into a proper module with interfaces.

1. **Never as one ticket.** A "refactor billing" ticket is how repos die. This runs as a **campaign**: S4 first, then a ticket *train*.
2. **S4 — architecture first.** `improve-codebase-architecture` skill produces an analysis; you turn it into ADR-0020: target module shape (`billing/{invoices,subscriptions,webhooks}` with a public `billing/index.ts` facade), the invariant ("no file outside `billing/` imports from `billing/internal/*`" — mechanically checkable, goes into CI as a lint rule *on day one*), and the migration strategy: **strangler, not big-bang**.
3. **Characterization first.** Ticket #250: write characterization tests around current behavior *without changing any code* — golden-master tests on invoice math, webhook handling fixtures. These are the safety net; the agent writes them, you spot-check that they'd actually fail on behavior change (mutation spot-check per [rituals](rituals-and-metrics.md)).
4. **The train** — `to-tickets` against ADR-0020, each step shippable and boring: #251 create facade re-exporting current internals (no behavior change), #252 move invoices behind the facade + fix its call sites, #253 subscriptions, #254 webhooks, #255 delete the legacy file + flip the lint rule to error. Every ticket: ≤1 day, characterization suite green = the acceptance criterion, file list explicit.
5. **Narrow runs, sequential.** One ticket at a time — refactor tickets are the exception to WIP=2 because they share files; parallel agents on shared files is the [playbook's](failure-recovery-playbook.md) "merge hell" tripwire. Fresh session each; the graph's `billing` ego-network is the context, not the whole repo.
6. **Review discipline changes**: the rubric's conformance axis becomes "zero behavior change" — you review diffs *specifically for* accidental semantics changes; CodeRabbit is good at spotting them mechanically. Any behavior change found = not a fix round; it's a bounce (the characterization suite was supposed to catch it — strengthen it first, then redo).
7. **Abort rule**: if two consecutive train tickets bounce, the decomposition is wrong. Stop the train, revisit ADR-0020, re-decompose the remainder. Sunk tickets stay merged — every step was independently shippable, so stopping mid-train leaves the repo *better*, never broken.
8. **S9** — the graph updates as modules move (that's `graph-update` doing real work), ADR-0020 flips to `implemented`, and the dump records what the decomposition got wrong for next time.

The property that makes this safe: at every commit on the train, `main` is releasable, the characterization suite passes, and the lint invariant narrows. The refactor cannot "destroy the repo" because no state of the repo is ever more than one small revert from healthy.
