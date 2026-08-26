# Multi-Agent — When One Agent Isn't the Right Shape

*The default in this OS is one fresh session per ticket, WIP ≤ 2. That is correct for ~80% of work. This doc covers the other ~20%: large migrations, dependency-independent ticket batches, research-heavy spikes, and well-specified async work. Multi-agent burns ~15× the tokens of chat — it is opt-in for high-value tasks, never a default.*

Evidence base: `research/research-2026-07-multi-agent.md` (Anthropic's multi-agent research system post, Claude Code best practices, framework landscape). Key numbers: multi-agent beat single-agent by **+90.2%** on Anthropic's internal research evals; **token usage alone explains ~80% of performance variance** (tool calls + model choice another 15%); agents burn ~4× chat tokens, multi-agent ~15×.

---

## The decision tree

| Situation                                                        | Pattern                          | Why                                                                  |
| ---------------------------------------------------------------- | -------------------------------- | -------------------------------------------------------------------- |
| One ticket, one PR, ≤1 day (the 80% case)                        | **Single agent** (default)       | Coordination cost buys nothing                                       |
| 2+ dependency-independent tickets in the same sprint             | **Parallel worktrees**           | No shared state, merge at the end                                    |
| Same mechanical transform across N files (migration, rename)     | **Batch mode** (`claude -p` loop)| 10× throughput on mechanical work                                    |
| Research spike / cross-cutting investigation, >5 sub-questions   | **Orchestrator-worker**          | Parallel breadth; +90.2% over single-agent on research evals         |
| Well-specified XS ticket, no unknowns, you're offline            | **Async fire-and-forget**        | Runs computer-off; you review the PR when back                       |
| Ambiguous architecture decision with real trade-offs             | **Council / debate** (optional)  | Multiple perspectives before one ADR                                 |

If the task doesn't clearly match a row, it's single-agent. Fan-out is a conclusion, not a default.

---

## Pattern 1 — Parallel worktrees (independent tickets)

Each ticket runs in its own git worktree with its own session; file edits can't collide; merge at the end. This is the lowest-coordination form of multi-agent — no orchestrator, no messaging, just isolation.

```bash
git worktree add ../repo-ticket-123 feat/123-slug
git worktree add ../repo-ticket-124 feat/124-slug
# one fresh session per worktree, normal S6 flow in each
```

**Rules:** tickets must be dependency-independent (check the S5 ordering); WIP limit still applies to *interactive* runs (≤2); each PR goes through the normal S7 pipeline. If two tickets touch the same file, they weren't independent — sequence them.

## Pattern 2 — Batch mode (mechanical migrations)

For the same transformation across N files:

```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from React to Vue. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

(OpenCode equivalent: `opencode -p` / `opencode run` in the same loop.)

**Rules:**
- **Monitor the first 2–3 files manually** before running at scale — batch mode amplifies a bad prompt N times.
- One commit per file; the file list is the plan and lives in the ticket.
- Allowed-tools scoped tight (`Edit` + `git commit` only).
- This scales above WIP=2 *because* it's non-interactive and monitored — it is not a loophole for parallel interactive runs.
- The whole batch is one ticket and one PR (or a small train if >400 lines); converge + rubric apply as usual.

## Pattern 3 — Orchestrator-worker (research spikes)

A lead session decomposes the question, spawns 3–5 subagents in parallel (`task` tool / Claude Code subagents), each investigates independently, the lead synthesizes. The reference architecture is Anthropic's own research system.

**Rules that make it work (all empirically grounded):**

1. **Persist the plan before fanning out.** Write it to `docs/research/<slug>/plan.md`. Lead-agent context truncates; a plan that lives only in the conversation is lost (same rule as the S5 exit gate — this is why persistence is a *gate*, not a habit).
2. **Filesystem output pattern.** Each subagent writes findings to `docs/research/<slug>/subagent-N.md` and returns *a reference, not the content*. Returning content through the coordinator is the "game of telephone" — information degrades every hop and token cost balloons.
3. **Delegate with boundaries.** Each subagent gets: objective, output format, tools/sources to use, explicit task boundary. Vague delegation = duplicated work + gaps.
4. **Scale effort to complexity.** Simple fact-find: 1 agent, 3–10 tool calls. Direct comparison: 2–4 subagents. Complex research: 5–10 with divided responsibilities. More is not better — token usage is the cost driver.
5. **Cost ceiling per orchestrator run** (cost-control.md): a fanned-out run gets its own budget, and the kill switch triggers on token spend, not just fix-loop count. Multi-agent turns a $3 task into a $50 run when misapplied.
6. **Evaluate end state, not path.** Did the synthesis answer the question? Don't grade whether subagents followed the steps you imagined — non-deterministic agents find alternative routes to the same goal.

## Pattern 4 — Async fire-and-forget (XS only)

Well-specified XS tickets with **no unknowns** can run unattended: Claude Code on the web / Routines, Copilot coding agent, or Codex Web (see the async sub-table in [harness-matrix.md](harness-matrix.md)). You come back to a PR and run the normal S7 pipeline.

**Rules:**
- XS only, ACs in the issue body, zero open questions. Anything needing judgment runs interactively. An async agent that hits an unknown makes a judgment call you didn't review — that's how spec drift starts.
- **C37 applies in full**: event-triggered sessions ingest data, never instructions; unattended sessions produce drafts and PRs, never merges, deploys, or config changes.
- Routines scheduled work (chores, dependency bumps, report generation) is draft-producing only — every output lands as a PR or comment for human review.

## Pattern 5 — Agent teams (coordinated multi-session)

Claude Code agent teams coordinate multiple sessions with shared tasks, messaging, and a team lead. This is the heavy option: real coordination, real coordination cost.

**When it's worth it:** a multi-day effort with genuinely interdependent streams (e.g., schema migration + API rework + frontend consumption) where worktrees alone can't express the dependencies. **When it isn't:** anything a ticket train + sequential PRs can express — which is almost everything. Reach for this last.

---

## The 12 multi-agent failure modes

Distinct from the 18 single-agent modes in [failure-modes.md](failure-modes.md). These live here because they only exist once you fan out.

| # | Failure mode                          | What happens                                                        | Mitigation                                                              |
| - | ------------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| F1 | Synchronous coordinator bottleneck   | System blocks on the slowest subagent                               | Smaller batches; async where possible                                   |
| F2 | Game of telephone                     | Information degrades through coordinator translation                | Filesystem output pattern — references, not content                     |
| F3 | Duplication and gaps                  | Two subagents do the same work; a third area uncovered              | Explicit task boundaries per subagent                                   |
| F4 | Subagent over-spawn                   | 50 subagents for a simple query                                     | Scaling rules (pattern 3, rule 4)                                       |
| F5 | Endless search                        | Subagents hunt for sources that don't exist                         | Effort budgets + explicit "stop when X" criteria                        |
| F6 | Context truncation drops the plan     | Lead forgets what it was doing mid-run                              | Plan persisted to disk before fan-out                                   |
| F7 | Non-determinism breaks evals          | Same prompt, different paths; "wrong steps" asserts fail            | End-state evaluation, not turn-by-turn                                  |
| F8 | Deployment breaks running agents      | Code change mid-flight kills in-progress work                       | Don't change agent config mid-run; rainbow long-running changes         |
| F9 | Shared-state races                    | Two agents edit the same file; last write wins                      | Worktree isolation; merge-based coordination                            |
| F10 | Cost explosion                        | $3 task becomes a $50 run                                           | Per-orchestrator cost ceiling; single-agent default                     |
| F11 | Adversarial reviewer over-reporting   | Fresh reviewer "finds gaps" that don't matter → over-engineering    | Instruct: flag only correctness/spec-affecting gaps, never style        |
| F12 | Coordinator memory pollution          | Lead's context fills with intermediate reports, synthesis degrades  | Subagents return references; lead fetches content only if needed        |

F11 deserves emphasis because this OS now ships a fresh-context evaluator: the `converge` gate at S7 is an adversarial reviewer by design. Its prompt already restricts findings to the frozen acceptance criteria — extend that habit to any fresh-context review subagent you dispatch.

---

## When multi-agent is wrong

- The ticket is well-specified and ≤1 day. (Almost always.)
- You're parallelizing because a single run *failed* — fix the plan, don't fan out the failure.
- The sub-questions share answers — then the context sharing IS the work, and splitting it creates telephone games.
- You can't state each subagent's boundary in one sentence — then you don't have a decomposition, you have a vibe.

The 80/20 asymmetry is the whole point: single-agent narrow-context discipline is why this OS works; multi-agent is a specialized tool you reach for with the decision tree, not a lifestyle.

---

*See also: [lifecycle.md](lifecycle.md) S6 (batch/async lanes), [cost-control.md](cost-control.md) (per-orchestrator ceilings), [constitution.md](constitution.md) C37 (event-triggered limits), [research-2026-07-multi-agent.md](research/research-2026-07-multi-agent.md) (full evidence).*
