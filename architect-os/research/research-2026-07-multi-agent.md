---
status: snapshot
last_verified: 2026-07-19
expires: 2026-10-19
---
<!-- SNAPSHOT: findings below were verified against live sources on last_verified.
     After the expiry date, treat every claim here as a hypothesis to re-verify,
     not a fact — this file must not become the next stale baseline. -->

# Research Report — Multi-Agent Orchestration for AI Coding (July 2026)

*Web research synthesis. 3 high-signal sources fetched 2026-07-19 plus synthesis from framework documentation. Designed to feed a new `multi-agent.md` doc and updates to `lifecycle.md`, `cost-control.md`, `failure-modes.md`, and `harness-matrix.md`.*

---

## Executive summary

Between mid-2025 and July 2026, multi-agent orchestration moved from a research curiosity to a production discipline. Anthropic shipped the reference architecture in June 2025 with its public **multi-agent research system** post — an orchestrator-worker pattern with a `LeadResearcher` agent that persists its plan to memory (because context truncates at ~200k tokens), spawns 3–10 subagents in parallel, and synthesizes results through a `CitationAgent`. The empirical findings are striking: a multi-agent Claude Opus 4 lead + Sonnet 4 subagents outperformed single-agent Opus 4 by **90.2%** on internal research evals, and **token usage alone explains 80% of performance variance** on BrowseComp (with tool call count and model choice making up the other 15%).

The cost story is the catch: agents use ~4× the tokens of chat, and multi-agent systems use ~15× the tokens of chat. Multi-agent is not a default — it is a tool for high-value tasks where parallelization buys back the token cost. Anthropic's own guidance: multi-agent excels at "valuable tasks that involve heavy parallelization, information that exceeds single context windows, and interfacing with numerous complex tools." Most coding work does not meet that bar. **architect-os's WIP=2 and narrow-context-per-ticket thesis is correct for the 80% case** — but the 20% case (large migrations, cross-cutting refactors, research-heavy spikes, parallel investigation of bugs) is currently undocumented in the methodology and that is the gap.

The framework market consolidated around four patterns: (1) **orchestrator-worker** (Anthropic Research, Claude Code subagents, OpenAI Swarm, LangGraph supervisor), (2) **DAG / event-driven flows** (CrewAI Flows + Crews, LangGraph state channels), (3) **role-based persona chains** (BMAD v6, CrewAI role-playing agents), and (4) **worktree-isolated parallel sessions** (Claude Code worktrees + Desktop app + Claude Code on the web, Claude DevFleet, Devin parallel sessions). The most operationally important finding: **synchronous orchestrator execution is the production bottleneck** (Anthropic explicitly calls this out), and the answer is async subagent execution with filesystem-based output passing to avoid the "game of telephone" — agents write to disk, return lightweight references, not full content through the coordinator.

For architect-os the recommended additions are: a `multi-agent.md` doc with a decision tree ("when to fan out vs stay single"), explicit support for the **Writer/Reviewer** and **Adversarial Review** subagent patterns that Claude Code now ships natively, an S5/S6 addition for parallel-ticket execution via git worktrees when tickets are dependency-independent, and a new failure-mode entry for "synchronous coordinator bottleneck."

---

## Part 1 — Anthropic's reference architecture (orchestrator-worker)

### Source: Anthropic, *How we built our multi-agent research system* (Jun 13, 2025)
URL: https://www.anthropic.com/engineering/built-multi-agent-research-system

### Architecture

A `LeadResearcher` agent receives the user query, **saves its plan to external Memory** (because context truncates above ~200k tokens — the plan must persist), then spawns 3–10 subagents in parallel. Each subagent independently uses search tools, evaluates results with **interleaved thinking** after each tool call, and returns findings. The `LeadResearcher` synthesizes, decides if more research is needed (can spawn more subagents), and finally hands off to a `CitationAgent` that locates citation anchors in the source documents.

Key engineering choices:
- **Plan persistence is mandatory.** If the lead agent's context exceeds ~200k tokens, it is truncated. The plan must live in external memory (file, KV, database) so the agent can re-read it after compaction.
- **Subagent filesystem output pattern** — subagents write findings to a filesystem or external store and return lightweight references to the lead agent. This bypasses the "game of telephone" where information degrades through coordinator translation. Token overhead drops dramatically because large outputs are not copied through conversation history.
- **Synchronous execution is the bottleneck.** Lead agent waits for each batch of subagents to complete before proceeding. Cannot steer subagents mid-flight; subagents cannot coordinate with each other. Anthropic flags async execution as a future improvement but acknowledges it adds coordination, state consistency, and error propagation challenges.

### Empirical findings

| Metric | Number | What it means |
|---|---|---|
| Multi-agent vs single-agent on research eval | +90.2% | Opus 4 lead + Sonnet 4 subagents beats single Opus 4 |
| Token usage explaining variance | 80% | More tokens = more answers, almost linearly |
| Tool calls + model choice | 15% | The other 15% of variance on BrowseComp |
| Agent tokens vs chat tokens | ~4× | A single agent run burns 4× a chat of equivalent length |
| Multi-agent tokens vs chat tokens | ~15× | Multi-agent burns 15× chat for the same nominal task |
| Parallel tool calling speedup | up to 90% | Lead spins up 3–5 subagents in parallel + subagents use 3+ tools in parallel |

The 80/15/5 split is the most actionable number: **token usage dwarfs model choice and tool design**. If your multi-agent architecture is not spending meaningfully more tokens than the single-agent alternative, you are paying the coordination cost without the benefit.

### Prompting principles (verified in production)

1. **Teach the orchestrator how to delegate.** Vague subtask descriptions like "research the semiconductor shortage" cause duplicate work and gaps. Each subagent needs: objective, output format, tools and sources to use, clear task boundaries.
2. **Scale effort to query complexity.** Simple fact-finding = 1 agent, 3–10 tool calls. Direct comparisons = 2–4 subagents, 10–15 calls each. Complex research = >10 subagents with divided responsibilities. Embed these scaling rules in the orchestrator prompt.
3. **Tool design is critical.** Each tool needs a distinct purpose and clear description. Bad tool descriptions send agents down wrong paths. The "tool-testing agent" pattern (an agent that uses a tool dozens of times and rewrites its description) cut task completion time by 40%.
4. **Start wide, then narrow.** Agents default to overly specific queries that return few results. Prompt them to start short and broad, evaluate, then narrow.
5. **Parallel tool calling transforms speed and performance.** Both the lead agent (parallel subagents) and subagents (parallel tools) must fan out.

### Production reliability challenges

- **Agents are stateful and errors compound.** Cannot just restart from zero — restarts are expensive and frustrating. Must build resume-from-checkpoint systems and use the model's adaptability ("let the agent know when a tool is failing and let it adapt works surprisingly well") plus deterministic safeguards (retries, checkpoints).
- **Debugging needs new approaches.** Agents are non-deterministic between runs even with identical prompts. Need full production tracing, plus monitoring of agent decision patterns and interaction structures (without monitoring conversation contents for privacy).
- **Deployment needs careful coordination.** Use **rainbow deployments** — gradually shift traffic from old to new versions while both run simultaneously, so well-meaning code changes don't break running agents.

### Evaluation methodology

- **LLM-as-judge with rubric scoring** — single LLM call with a single prompt outputting 0.0–1.0 scores and pass/fail was most consistent. Evaluated: factual accuracy, citation accuracy, completeness, source quality, tool efficiency.
- **End-state evaluation for stateful agents** — judge whether the agent achieved the correct final state, not whether it followed a "correct" prescribed path. Agents can find alternative paths to the same goal.
- **Start with ~20 test cases** — early changes have dramatic effect sizes (30% → 80% from a prompt tweak). Don't wait for hundreds of test cases; small-scale testing spots these.

---

## Part 2 — Claude Code native multi-agent primitives

### Source: Anthropic, *Best practices for Claude Code* (fetched 2026-07-19)
URL: https://www.anthropic.com/engineering/claude-code-best-practices

Claude Code ships four parallel-execution primitives that map directly onto the orchestrator-worker pattern:

1. **Worktrees** (`claude` CLI in isolated git checkouts) — separate sessions per worktree so file edits cannot collide. This is the lowest-coordination form of multi-agent: each ticket runs in its own worktree, no shared state, merge at the end.
2. **Desktop app sessions** — manage multiple local Claude sessions visually, each in its own worktree. Same primitive as worktrees but with a UI.
3. **Claude Code on the web** — sessions run on Anthropic-managed cloud infrastructure in isolated VMs. This is the closest thing to "Codex Cloud" or "Devin" — fire-and-forget async sessions.
4. **Agent teams** — automated coordination of multiple sessions with shared tasks, messaging, and a team lead. This is the native Claude Code multi-agent system. It is the production-ready version of the orchestrator-worker pattern.

### Patterns Claude Code explicitly recommends

| Pattern | Description | Where it fits architect-os |
|---|---|---|
| **Writer/Reviewer** | Session A writes code, Session B reviews with fresh context (avoids self-justification bias). | Maps to architect-os S6+S7, but currently S7 uses CodeRabbit (model-coupled to the same author model). A fresh-session subagent reviewer is missing. |
| **Tests-first / code-second split** | One Claude writes tests, another writes code to pass them. | New pattern not covered in architect-os. Useful for high-stakes tickets. |
| **Fan out across files** | `for file in $(cat files.txt); do claude -p "Migrate $file from React to Vue" --allowedTools "Edit,Bash(git commit *)"; done` | Direct fit for large-batch refactors (currently architect-os treats these as separate tickets — this is faster). |
| **Adversarial review subagent** | After implementation, subagent reviews diff against PLAN.md in fresh context — only sees diff + criteria, not the reasoning. | Strong addition to S7. Claude Code ships a bundled `/code-review` skill that runs in a fresh subagent. |
| **Headless / non-interactive (`claude -p`)** | CI integration, pre-commit hooks, scripted workflows. JSON or stream-JSON output for programmatic parsing. | Currently architect-os only mentions this in passing. Should be in S6/S9. |

### Critical Claude Code best practice that maps to multi-agent

> "The longer Claude works unattended, the more an independent check matters before you count the work as done. A reviewer running in a fresh subagent context sees only the diff and the criteria you give it, not the reasoning that produced the change."

This is the operational definition of why fresh-context subagents matter for review: they are uncorrelated with the implementation context. A reviewer that shares the implementation context will justify the implementation.

> "A reviewer prompted to find gaps will usually report some, even when the work is sound, because that is what it was asked to do. Chasing every finding leads to over-engineering."

Important caveat — adversarial reviewers over-report. Must instruct them to flag only gaps that affect correctness or stated requirements, not style preferences.

---

## Part 3 — Framework landscape (mid-2026)

### CrewAI (Flows + Crews)

URL: https://docs.crewai.com/

CrewAI is the leading open-source multi-agent framework, with 100,000+ developers certified through community courses. The architecture separates two layers:

- **Flows** — the backbone. Event-driven, stateful, with control flow (conditionals, loops, branching). Manages state across steps and executions. The "manager" or process definition.
- **Crews** — the intelligence. Teams of role-playing agents with specific goals and tools that collaborate on a complex task delegated by a Flow.

The pattern: **start with a Flow, drop in a Crew when autonomy is needed.** This is structurally similar to architect-os's S5 (plan, deterministic) → S6 (autonomous agent) split, but at the agent-coordination layer instead of the workflow layer.

CrewAI explicitly ships **Claude Code, Codex, and Cursor skills** installable via `npx skills add crewaiinc/skills`. So the framework itself is now coding-agent-aware.

**Relevance to architect-os:** CrewAI is the canonical "DAG-of-crews" pattern. If architect-os wanted to add an explicit multi-agent layer, CrewAI Flows are the open-source primitive — but Claude Code's native "agent teams" feature is a closer fit because it doesn't add a framework dependency.

### Other major frameworks (synthesized from general knowledge, mid-2026 state)

| Framework | Pattern | Notable |
|---|---|---|
| **LangGraph** (LangChain) | Stateful graph with typed state channels, supervisor pattern, subgraph composition. | Most production-deployed. State channels are the differentiator. |
| **OpenAI Agents SDK** (formerly Swarm) | Lightweight handoff-based orchestration. Agents hand off to each other by returning another agent. | Minimal abstraction. OpenAI's answer to "we don't need a framework, just patterns." |
| **AutoGen / AG2** | Conversational multi-agent with round-robin and group chat. | Strong for debate/discussion patterns. AG2 is the community-maintained fork. |
| **Google ADK** (Agent Development Kit) | Agent primitives integrated with Gemini and Google Cloud. | Tight Google Cloud integration. |
| **Claude Agent SDK** | Anthropic's official SDK for building agents with Claude. | Subagent dispatch, tool use, MCP integration. Native to Claude Code. |
| **BMAD v6** (bmad.dev) | Role-based persona chain (analyst → PM → architect → dev) with story files as context carriers. | Already covered in architect-os harness-matrix as the heavy-ceremony option. |
| **Claude DevFleet** | Multi-agent orchestration via git worktrees. Each agent = one worktree = one ticket. | Open-source; directly aligned with architect-os's "one ticket = one branch" rule. |
| **Devin (Cognition)** | Parallel cloud sessions, persistent context, autonomous task execution. | Commercial; closest to "async fire-and-forget" for long tasks. |
| **Cline / Aider** | Single-agent but with multi-file awareness. Not truly multi-agent but often confused with it. | Both remain single-session; "parallel Cline" is multiple terminal windows, not a coordinated system. |

### Coordination patterns observed across frameworks

| Pattern | Description | When it applies |
|---|---|---|
| **Orchestrator-worker** (Anthropic Research, Claude Code agent teams, OpenAI Swarm, LangGraph supervisor) | Lead agent decomposes, workers execute in parallel, lead synthesizes. | Default for research and breadth-first tasks. |
| **DAG / event-driven** (CrewAI Flows, LangGraph state channels) | Workflow graph with typed state, conditional branching. | When the process shape is known and deterministic but has parallel branches. |
| **Role-based persona chain** (BMAD, CrewAI role-playing crews) | Sequential personas with artifact handoffs (analyst → PM → architect → dev). | Heavy-ceremony, regulated environments, compliance-heavy work. |
| **Worktree-isolated parallel sessions** (Claude Code worktrees, DevFleet, Devin parallel) | Each agent works in its own git worktree. Merge at the end. | When tickets are dependency-independent. Lowest coordination overhead. |
| **Council / debate** (AutoGen group chat, council skills) | Multiple agents argue different positions, a synthesizer picks. | Ambiguous decisions, tradeoffs, go/no-go calls. |
| **Blackboard** (legacy pattern, rare in 2026) | Shared scratchpad agents read/write to. | When state is the integration point, not control flow. |

---

## Part 4 — Failure modes of multi-agent systems

These are the empirically observed failure modes from Anthropic's production deployment and framework documentation:

### F1. Synchronous coordinator bottleneck
The lead agent waits for each batch of subagents before proceeding. The whole system blocks on the slowest subagent. **Mitigation:** async execution (with the coordination cost it implies) or smaller subagent batches.

### F2. Game of telephone
Information degrades as it passes through the coordinator. The lead agent receives a summary of a summary of findings. **Mitigation:** subagent filesystem output pattern — subagents write to disk, return references, not content.

### F3. Subagent duplication and gaps
Without explicit task boundaries, two subagents do the same search while a third area goes uncovered. **Mitigation:** orchestrator prompt must specify objective + output format + tools + task boundaries per subagent.

### F4. Subagent over-spawn
Lead agent spawns 50 subagents for a simple query. **Mitigation:** scaling rules in the orchestrator prompt (1 agent for fact-finding, 2–4 for comparisons, >10 for complex research).

### F5. Endless search
Subagents scour the web for sources that don't exist. **Mitigation:** effort-budget caps and explicit "stop when X" criteria in the subagent prompt.

### F6. Context truncation drops the plan
If the lead agent's context exceeds ~200k tokens, the plan is lost. The agent forgets what it was doing. **Mitigation:** plan persistence to external memory at the start of every run.

### F7. Non-deterministic behavior breaks evals
Same prompt produces different paths across runs. Cannot assert "agent followed correct steps." **Mitigation:** end-state evaluation, not turn-by-turn; rubric-based LLM-as-judge.

### F8. Deployment breaks running agents
Code change to agent logic mid-flight breaks in-progress agents. **Mitigation:** rainbow deployments (gradual traffic shift with both versions running simultaneously).

### F9. Race conditions in shared-state agents
Two agents edit the same file. Last write wins, earlier work lost. **Mitigation:** worktree isolation per agent; merge-based coordination, not shared-file coordination.

### F10. Cost explosion
Multi-agent burns 15× chat tokens. A task that should have been single-agent becomes a $50 run instead of a $3 run. **Mitigation:** explicit cost ceiling per orchestrator run; escalate to multi-agent only when the task is high-value.

### F11. Adversarial reviewer over-reporting
A reviewer subagent prompted to "find gaps" will report gaps even when work is sound, because that is what it was asked to do. Chasing every finding leads to over-engineering. **Mitigation:** instruct reviewer to flag only gaps that affect correctness or stated requirements, not style.

### F12. Coordinator memory pollution
The lead agent's context fills with intermediate subagent reports, degrading synthesis quality. **Mitigation:** subagents return references, not content; lead agent fetches content only if needed for final synthesis.

---

## Part 5 — Gaps in architect-os

The architect-os repo has **near-zero multi-agent coverage** and explicitly avoids parallel agents. This is correct for the 80% case but leaves the 20% case undocumented.

### Specific gaps

1. **No `multi-agent.md` doc.** The harness-matrix.md and lifecycle.md are single-agent-only. There is no decision tree for "when to fan out vs stay single."

2. **WIP=2 is treated as a hard limit, not a default.** The lifecycle says "WIP limit: 2 concurrent runs" but does not distinguish:
   - 2 concurrent runs of the *same ticket* (almost never useful)
   - 2 concurrent runs of *different tickets* (often useful when tickets are dependency-independent)
   - N concurrent runs for batch refactors (useful for `claude -p` loops over file lists)
   The rule should be: "WIP=2 for *interactive* runs. Batch mode (`claude -p`) can scale higher when tickets are independent and the user is monitoring."

3. **No Writer/Reviewer subagent pattern.** S7 review uses CodeRabbit (correlated with author model) + Codex review (different model). The Claude Code native pattern — fresh subagent reviewer that sees only diff + criteria, not implementation context — is not in the review-workflow.md. This is the single highest-value missing addition.

4. **No adversarial subagent pattern at S6 end.** Claude Code ships a bundled `/code-review` skill that runs a fresh subagent review of the diff. architect-os should add this as a mandatory S6 exit step before requesting human review at S7.

5. **No filesystem output pattern for subagents.** architect-os subagents return content through conversation. The Anthropic-recommended pattern (subagents write to disk, return references) is not codified. This matters because it reduces token cost and avoids the game of telephone.

6. **No plan persistence requirement.** The lifecycle says "fresh session per ticket" but does not require the plan to be persisted to external memory before S6 begins. If the session truncates, the plan is lost. Should be an S5 exit criterion: "plan persisted to `docs/specs/<slug>/plan.md`."

7. **No batch refactor pattern.** The Claude Code `for file in $(cat files.txt); do claude -p "Migrate $file..." --allowedTools "Edit,Bash(git commit *)"; done` pattern is the standard approach for large migrations. architect-os treats each migration as N tickets, which is correct but slow. A "batch mode" addition to S6 would unlock 10× throughput on mechanical migrations.

8. **No async fire-and-forget for XS tickets.** The harness-matrix mentions Copilot coding agent and Codex Cloud as alternatives, but the lifecycle itself has no async lane. Claude Code on the web (isolated cloud VMs) is the native async option and is not mentioned.

9. **No cost ceiling per orchestrator run.** Multi-agent systems can burn $50 on a $3 task. cost-control.md has per-ticket budgets but not per-orchestrator budgets. The kill switch should trigger on token spend, not just fix-loop count.

10. **No end-state evaluation pattern.** failure-modes.md #11 (context window decay) and #8 (agent loops) are about the agent failing mid-run. The Anthropic-recommended mitigation — judge end state, not process — is not in the rubric. pr-review-rubric.md should add "end state achieved?" as a primary check.

11. **Rainbow deployment concept is missing.** For teams operating long-running agents, code changes to agent logic can break in-flight agents. architect-os has no guidance on this because it assumes short-lived sessions. As async / background agents are added (gap #8), this becomes relevant.

12. **The "council" pattern is missing.** architect-os has no equivalent for ambiguous-decision moments where multiple perspectives matter. The `council` skill (4-voice simulated debate) exists in the broader ecosystem and could be added as an S4 option for architecture decisions.

### Outdated assumptions

- "Claude Code is synchronous only" (harness-matrix.md) — Claude Code on the web and Claude Code agent teams make this false as of 2026.
- "Fresh session means fresh context" (lifecycle.md) — true for the session, but agent teams now share state via team messaging and shared tasks. The dichotomy is no longer clean.
- "Context window decay" (failure-modes.md #11) — the mitigation ("fresh sessions") is correct, but the deeper mitigation is subagent architectures that isolate context per concern. The current write-up treats this as a session-level issue when it is fundamentally an agent-architecture issue.

---

## Part 6 — Recommended additions to architect-os

### A. New file: `architect-os/multi-agent.md`

A short doc (≤200 lines) covering:

1. **Decision tree:** when to fan out vs stay single
   - Single-agent (default): one ticket, one PR, ≤1 day, dependency on prior work
   - Parallel worktrees: 2+ dependency-independent tickets, same sprint
   - Batch `claude -p`: mechanical migration across N files, same transformation
   - Orchestrator-worker: research spike, cross-cutting investigation, >5 sub-questions
   - Async fire-and-forget: well-specified XS ticket, no unknowns, you're offline
2. **Native Claude Code patterns to use**
   - Worktrees for parallel ticket execution
   - Desktop app for visual management of parallel sessions
   - Claude Code on the web for async fire-and-forget
   - Agent teams for orchestrated multi-agent work
   - `/code-review` skill for adversarial fresh-context subagent review
3. **Filesystem output pattern** for subagents (write to disk, return references)
4. **Plan persistence requirement** before fanning out
5. **Cost ceilings per orchestrator run**
6. **Failure modes** (the 12 above)
7. **When multi-agent is wrong** (the 80% case where architect-os stays single-agent)

### B. Update `lifecycle.md`

- Add a new line under S6: "**Async mode:** XS tickets with no unknowns can run async via Claude Code on the web or Copilot coding agent. Monitor for completion; do not async anything that needs human judgment."
- Add a new line under S7: "**Adversarial subagent review:** before human review, run the bundled `/code-review` skill in a fresh subagent context. It sees only the diff and the plan, not the implementation reasoning."
- Add a new S5 exit criterion: "Plan persisted to `docs/specs/<slug>/plan.md` (or equivalent), so it survives session truncation."

### C. Update `harness-matrix.md`

- Update the Claude Code section: "no async fire-and-forget" limitation is obsolete. Claude Code on the web and agent teams are the async primitives.
- Add a new section: **"Multi-agent orchestration"** covering Claude Code agent teams, CrewAI Flows, LangGraph, OpenAI Agents SDK, AutoGen, BMAD v6, Claude DevFleet, Devin.
- Add a new row to the one-glance matrix for "Multi-agent (orchestrator-worker)" → Claude Code agent teams (default), CrewAI (alternative), LangGraph (alternative).

### D. Update `cost-control.md`

- Add **per-orchestrator cost ceiling** (separate from per-ticket ceiling). Multi-agent burns 15× chat; the kill switch should trigger on token spend, not just fix-loop count.
- Add **batch API** as a cost lever for S9 (memory dumps), S7 (bulk code review), and eval runs.

### E. Update `failure-modes.md`

Add 3 new failure modes (extending the existing 12):

- **#13 — Synchronous coordinator bottleneck.** Multi-agent system blocks on the slowest subagent. Mitigation: smaller batches or async execution.
- **#14 — Game of telephone.** Information degrades through coordinator translation. Mitigation: subagent filesystem output pattern.
- **#15 — Adversarial reviewer over-reporting.** Fresh-context reviewer flags style as gaps, leading to over-engineering. Mitigation: instruct reviewer to flag only correctness-affecting gaps, not style preferences.

### F. Update `pr-review-rubric.md`

Add a primary check: **"End state achieved?"** — does the diff achieve the spec's stated end state, regardless of the path the agent took to get there? (Anthropic-recommended end-state evaluation, not turn-by-turn.)

### G. New skill: `multi-agent-dispatch`

A skill that wraps the orchestrator-worker pattern for architect-os use cases:
- Input: a research question or multi-file investigation
- Spawns 3–5 subagents in parallel via Claude Code subagents
- Each subagent writes findings to `docs/research/<slug>/subagent-N.md`
- Returns references to the orchestrator
- Synthesizes into `docs/research/<slug>/synthesis.md`

---

## Summary

architect-os's single-agent, narrow-context, WIP=2 thesis is correct for the 80% case and should remain the default. The 20% case — large migrations, parallel investigations, research-heavy spikes, dependency-independent tickets in the same sprint — is currently undocumented. The recommended additions are conservative: one new doc (`multi-agent.md`), three new failure modes, native use of Claude Code's already-shipped patterns (worktrees, agent teams, `/code-review` skill, `claude -p` batch mode), and a plan-persistence requirement that hardens what the lifecycle already implies. The cost discipline is unchanged — multi-agent burns 15× tokens and should be opt-in for high-value tasks, not a default.

---

*Sources: Anthropic engineering blog (anthropic.com/engineering) — "How we built our multi-agent research system" (Jun 13, 2025) and "Best practices for Claude Code" (fetched 2026-07-19); CrewAI documentation (docs.crewai.com, fetched 2026-07-19). Framework landscape synthesized from general knowledge of LangGraph, OpenAI Agents SDK, AutoGen, Google ADK, BMAD v6, Claude DevFleet, and Devin state as of mid-2026.*
