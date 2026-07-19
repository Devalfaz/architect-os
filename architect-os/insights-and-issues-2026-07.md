---
status: snapshot
last_verified: 2026-07-19
expires: 2026-10-19
---
<!-- SNAPSHOT: findings below were verified against live sources on last_verified.
     After the expiry date, treat every claim here as a hypothesis to re-verify,
     not a fact — this file must not become the next stale baseline. -->

# Insights & Issues — Architect OS Research Update (July 2026)

*Master synthesis of five parallel internet research passes conducted 2026-07-19. Companion to the five detailed reports in this directory: `research-2026-07-update.md` (cost & failure modes), `research-2026-07-memory.md` (memory & context), `research-2026-07-tools.md` (AI coding tools), `research-2026-07-specs.md` (spec-driven dev), `research-2026-07-review.md` (AI code review), `research-2026-07-multi-agent.md` (multi-agent orchestration).*

**Scope:** What changed since the mid-2025 architect-os baseline, what the existing research got right, what is now wrong or stale, and what is missing.

**Method:** Six research passes against authoritative primary sources — official docs (docs.anthropic.com, platform.openai.com, docs.github.com, docs.coderabbit.ai, docs.crewai.com, docs.mem0.ai, docs.letta.com, docs.anthropic.com/en/docs/claude-code), engineering blogs (Anthropic, OpenAI, GitHub, Cursor, Greptile, CodeRabbit), framework repos (GitHub Spec Kit, BMad, OpenAI Codex), and industry analysis (Greptile overnight-agents study, Anthropic March 2026 harness-design post, Anthropic April 2026 Claude Code quality post-mortem).

---

## TL;DR — what changed since mid-2025

Architect OS's core thesis — **spec-first, narrow-context, human-gated, fresh-session-per-ticket** — was independently validated by three of the most influential pieces of writing on AI dev in the last year: Anthropic's June 2025 multi-agent research post, Anthropic's September 2025 *Effective context engineering for AI agents* post, and Anthropic's March 2026 *Harness design for long-running application development* post. The lifecycle, the constitution, and the four-layer memory model all hold up.

But the supporting cast is now stale:

1. **The tool market reorganized around agent-native control centers**, not CLI-vs-IDE-vs-cloud. Claude Code (Desktop + Web + iOS + agent teams + routines + channels), GitHub Copilot app, Cursor Cloud Agents, Devin Cloud all converged on a "My Work" multi-session surface. The mid-2025 matrix's "Claude Code has no async fire-and-forget" limitation is obsolete. (`research-2026-07-tools.md`)

2. **The model landscape shifted faster than any other layer.** Anthropic added Fable 5, Opus 4.8, Sonnet 5; OpenAI added GPT-5.6 family (sol/terra/luna), cut o3 from $10/$40 to $2/$8; Sonnet 5 intro pricing ($2/$10) ends Aug 31 2026. The mid-2025 matrix's pricing is wrong; the July 2026 update is correct but does not mention LiteLLM/OpenRouter routing, Batch APIs, or the new `prompt_cache_key` per-key RPM ceiling. (`research-2026-07-update.md`)

3. **Spec-driven dev became orthodoxy, not experiment.** GitHub Spec Kit hit 122k stars (v0.13.0 Jul 17 2026); AWS Kiro shipped as a productized spec-first IDE; BMad v6 (50.8k stars, v6.10.0 Jul 3 2026) added scale-adaptive sizing; Anthropic's March 2026 harness-design post gave the first hard empirical evidence that a planner/generator/evaluator harness beats solo-agent by ~20× quality at ~20× cost. The architect-os S2→S5→S6→S7 loop is isomorphic to all of these but missing three things: a `/converge`-style conformance command, scale-adaptive spec sizing, and an explicit evaluator agent at S7. (`research-2026-07-specs.md`)

4. **AI code review split into three layers, and architect-os has only one.** Layer 1: read-only static-diff reviewers (CodeRabbit, Copilot Code Review, Cursor BugBot, Greptile, Qodo) — commoditized. Layer 2: independent-auditor framing (Greptile refuses to author code; argues same-vendor author+review is the Arthur-Andersen problem) — architect-os has the correlated-vendor problem because it uses Claude Code to author and claude-code-action to review. Layer 3: runtime/execution review (Greptile TREX, Jun 2026, runs the code in a sandbox) — architect-os has no equivalent. (`research-2026-07-review.md`)

5. **Context engineering became a named discipline.** Anthropic's September 2025 post gave the canonical framing: context is a finite resource with diminishing marginal returns; the goal is "the smallest possible set of high-signal tokens that maximize the likelihood of the desired outcome." Three concrete techniques dominate: **compaction, structured note-taking, sub-agent architectures**. The 150-line AGENTS.md cap in architect-os is correct in spirit but the *content mix* is slightly wrong — Anthropic now explicitly says "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!" and provides an Include/Exclude table that the architect-os Layer 1 doesn't fully honor. (`research-2026-07-memory.md`)

6. **Multi-agent orchestration is a real production discipline now.** Anthropic's June 2025 multi-agent research post shows +90.2% improvement over single-agent on research evals and quantifies the cost: 15× chat tokens. The orchestrator-worker pattern with filesystem-based subagent output (avoiding the "game of telephone") is the reference. Architect-os's WIP=2 single-agent thesis is correct for the 80% case but undocumented for the 20% case (large migrations, parallel investigation, dependency-independent ticket batches). Claude Code now ships native worktrees, agent teams, and `claude -p` batch mode that architect-os doesn't mention. (`research-2026-07-multi-agent.md`)

7. **15 new failure modes surfaced beyond the existing 12.** Most urgent: prompt injection via tool output, hooks executing before trust dialog, persistent memory poisoning, sandbox escape via creative problem-solving, self-evaluation leniency, caching optimization regressions (the April 2026 Claude Code quality post-mortem template), correlated-model review blind spots, collateral-damage edits (random unrelated line edits humans would never make), false confidence from "AI approved it," agent abandonment of subagent results, synchronous coordinator bottleneck, game of telephone. (`research-2026-07-update.md` + `research-2026-07-review.md` + `research-2026-07-multi-agent.md`)

---

## What the existing research got right (validated by mid-2026 evidence)

These seven theses from architect-os were independently validated by 2025–2026 industry writing and should not change:

1. **"The spec is the source of truth" (constitution C1).** Anthropic's March 2026 harness-design post is the hard empirical version of this: a generator/evaluator harness with explicit specs beats solo-agent by ~20×. GitHub Spec Kit (122k stars), AWS Kiro, and BMad v6 all converged on the same lifecycle architect-os already has.

2. **"Narrow context beats big context" (principle #2).** Anthropic's September 2025 context engineering post is the canonical formulation: "the smallest possible set of high-signal tokens that maximize the likelihood of the desired outcome." The Claude Code best practices doc explicitly warns "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"

3. **"Human review comes first" (principle #3).** Greptile's overnight-agents data and the auditor post validate this — but with a twist. The bias risk is not anchoring; it's "trust drift" where humans sign off because the AI reviewer signed off. The two-pass model is right; the second pass should be *uncorrelated* AI, not the same vendor.

4. **"Plans are reviewed harder than diffs" (principle #4).** Anthropic's harness-design post explicitly calls out the planner agent as the high-leverage role. The architect-os S5 plan-review gate is the right shape.

5. **"Memory has a freshness date" (principle #5).** Zep's April 2026 S&P Market Intelligence recognition as the enterprise memory layer validates this — its core differentiator is *automatic invalidation of stale facts when new evidence contradicts old ones*. architect-os's "stale = hypothesis, not fact" framing is correct in spirit but operationally too manual. The fix is automatic invalidation, not weekly review.

6. **"Small everything" (principle #6).** The 400-line PR cap, ≤1 day tickets, ≤2 fix rounds, ≤2 WIP are all still correct. None of the 2026 writing contradicts these.

7. **"Agents propose, you decide" (principle #7).** Validated by every major incident post-mortem this year — the April 2026 Claude Code quality degradation (3 overlapping bugs, 6 weeks of degradation) and the September 2025 Anthropic infrastructure post-mortem both show that even the vendor struggles to detect these failure classes for weeks. Human gates are the last line of defense.

---

## Issues with the existing research (prioritized)

### Critical (ship blockers — fix within 30 days)

#### I1. The harness matrix's "Claude Code has no async fire-and-forget" limitation is obsolete

`harness-matrix.md:68` says: "No async fire-and-forget — you launch a session and it runs synchronously (or backgrounded). Can't be triggered by a GitHub webhook."

This was true in mid-2025. As of July 2026, Claude Code ships:
- **Routines** — Anthropic-managed, scheduled or event-triggered, runs even when computer is off
- **Channels** — push Telegram/Discord/iMessage/webhook events into a session
- **Claude Code on the web** — isolated cloud VMs for async sessions
- **Agent teams** — automated coordination of multiple sessions
- **Slack integration** — `@Claude` in Slack with a bug report → PR back
- **GitHub Code Review** — automatic review on every PR (native)

The architect-os thesis that CLI agents are the right primary harness is intact; the supporting claim that they can't do async is wrong.

**Fix:** Update `harness-matrix.md` Claude Code section. Add a new "Async primitives" subsection. Update the one-glance matrix S6 row for "S6 (XS async)" to include Claude Code Routines.

#### I2. The review workflow has a correlated-vendor problem

`review-workflow.md` and `lifecycle.md` S7 specify: "CodeRabbit auto-review + a second AI opinion (Codex review or claude-code-action)."

Greptile's August 2025 auditor post (https://www.greptile.com/blog/auditor) argues that author and reviewer sharing the same vendor is the Arthur-Andersen problem applied to PRs. Greptile's overnight-agents data (https://www.greptile.com/blog/rise-of-the-overnight-agents) gives the empirical version: Claude-authored PRs reviewed by Claude-family models miss IDOR/tenancy bugs at 1.75× the human rate; Cursor BG + BugBot miss n+1 bugs at 3.45× the human rate.

The architect-os S7 currently allows the *worst* configuration (Claude authored → claude-code-action reviews) as one of two acceptable options.

**Fix:** Update `review-workflow.md` to make the second-opinion choice explicit:
- If PR authored by Claude Code → second opinion must be OpenAI-family (Codex review or Copilot Code Review) or runtime (Greptile TREX)
- If PR authored by Codex → second opinion must be Anthropic-family (claude-code-action with Opus, or CodeRabbit)
- Always: claude-code-action supports Bedrock/Vertex with non-Claude models — run it against a non-Claude model for uncorrelated review from the same action

Add a new rule to the constitution: **C36 — Reviewer independence.** The AI reviewer must not share model family with the author when both are AI.

#### I3. The constitution has no learning loop

CodeRabbit's `Learnings` feature and the `@coderabbitai emit path instructions` command turn resolved review comments into permanent rule updates. architect-os's C1–C35 are static — there is no mechanism for the constitution to learn from review outcomes. Greptile's noise-control data shows that without this loop, address rate stays at 19%; with it, address rate goes to 55% in two weeks.

**Fix:** Add a new ritual in `rituals-and-metrics.md`: **Monthly constitution review.** Pull resolved CodeRabbit comments, run `@coderabbitai emit path instructions`, merge suggestions into `.coderabbit.yaml` and proposed amendments into `constitution.md` as ADRs.

#### I4. No runtime validation layer in S7

Greptile TREX (Jun 2026, https://www.greptile.com/blog/trex) runs changed code in a sandbox and produces artifacts (stdout, screenshots, test output, generated images). This catches a class of bugs that static-diff review structurally cannot — "function signature matches the docs but the installed package's export name differs," "regex compiles but doesn't match test fixtures." This is the largest capability delta in the 2026 review landscape.

architect-os has no equivalent. The `pr-review-rubric.md` 10-minute rubric is static-diff only.

**Fix:** Add an optional "runtime validation" step to `review-workflow.md` between AI review and human merge. Tools: Greptile TREX (commercial), or a self-rolled sandbox that runs the diff's affected tests + a smoke against the changed entrypoints.

#### I5. Three new failure modes need to be added to failure-modes.md

The existing 12 failure modes don't cover what 2026 surfaced. Add:

- **#13 — Correlated-model review blind spots.** When author and reviewer share model family, both miss the same bug class. Mitigation: C36 (review independence), runtime validation.

- **#14 — Collateral-damage edits.** AI agents make random unrelated line edits that humans would never make (whitespace "fixes," import reordering, comment rephrasing). These pollute diffs and hide real changes. Mitigation: `git diff --word-diff` review mode; constitution C7 update to explicitly ban "improvements" outside the planned file list even when adjacent.

- **#15 — False confidence from "AI approved it."** Human reviewers sign off because the AI reviewer signed off. Mitigation: the human rubric must run *before* reading AI comments (already in `pr-review-rubric.md` — keep it); add a "trust drift" check to monthly retro: "Did I approve anything this month where I deferred to the AI reviewer against my own judgment?"

### High priority (fix within 90 days)

#### I6. Models-cost-quality.md is missing the new cost-control levers

The July 2026 update added Fable 5, Opus 4.8, Sonnet 5, GPT-5.6 family. But it does not mention:

- **Batch APIs** (50% off, stacks with caching for 30–98% additional savings) — natural fit for S9 dumps, code-review bulk scans, eval runs
- **LiteLLM routing** — 7 routing strategies, routing groups, weighted failover
- **OpenRouter** — unified API across hundreds of models, MCP server at `https://mcp.openrouter.ai/mcp`
- **Anthropic automatic caching mode** — top-level `cache_control`, 20-block lookback, cache diagnostics beta, workspace-level isolation (Feb 5 2026)
- **OpenAI `prompt_cache_key`** — the 15 RPM per-key ceiling that silently destroys cache hit rates if not partitioned
- **Sonnet 5 intro pricing ends Aug 31 2026** — needs explicit note in routing table; standard price returns $3/$15

**Fix:** Update `cost-control.md` with a "Cost levers" section covering Batch APIs, LiteLLM, OpenRouter. Update `models-cost-quality.md` routing table with the Aug 31 2026 Sonnet 5 price change note.

#### I7. No `/converge`-style conformance pass at S7

GitHub Spec Kit ships `/speckit.converge` — "assess codebase against spec/plan/tasks and append remaining work as new tasks." This is the mechanical version of architect-os's "discovery loop" (S6 → S2 spec delta). architect-os has the loop as a diagram edge but no command/role that *closes* the gap between what was built and what was specified.

Anthropic's March 2026 harness-design post validates this: the harness's *evaluator agent* was the difference between "core feature literally doesn't work" and "plays end to end."

**Fix:** Add a new skill `converge` that runs at S7: reads the FSD, reads the diff, produces a spec-conformance report listing what's done, what's partially done, what's missing, what's extra. Add to `lifecycle.md` S7 as an optional pre-merge step.

#### I8. AGENTS.md content mix needs pruning toward Anthropic's new guidance

Anthropic's *Best practices for Claude Code* (fetched 2026-07-19) provides an explicit Include/Exclude table for CLAUDE.md:

| ✅ Include | ❌ Exclude |
|---|---|
| Bash commands Claude can't guess | Anything Claude can figure out by reading code |
| Code style rules that differ from defaults | Standard language conventions Claude already knows |
| Testing instructions and preferred test runners | Detailed API documentation (link to docs instead) |
| Repository etiquette | Information that changes frequently |
| Architectural decisions specific to your project | Long explanations or tutorials |
| Developer environment quirks | File-by-file descriptions of the codebase |
| Common gotchas | Self-evident practices |

architect-os's Layer 1 spec ("Domain language (5–10 terms), File conventions (10 most important paths), Constitution summary, Architecture style and key decisions, Gotchas, Active constraints") slightly over-encroaches on the Exclude column. "File conventions (10 most important paths)" is borderline — Anthropic now says "Anything Claude can figure out by reading code" should be cut.

**Fix:** Reframe `repo-memory.md` Layer 1 contents: drop "File conventions (10 most important paths)," keep only "files Claude can't guess." Keep the 150-line cap. Add the Anthropic Include/Exclude table to `repo-memory.md`.

#### I9. No multi-agent decision tree

architect-os has WIP=2 and "fresh session per ticket." This is correct for 80% of cases but undocumented for:
- 2+ dependency-independent tickets in the same sprint (worktree parallelism)
- Mechanical migrations across N files (`claude -p` loop)
- Research spikes with >5 sub-questions (orchestrator-worker)
- Async fire-and-forget for XS tickets (Claude Code Routines)

**Fix:** Add a new `multi-agent.md` doc (≤200 lines) with a decision tree, native Claude Code patterns to use (worktrees, agent teams, `/code-review` skill, `claude -p` batch mode, Claude Code on the web), the filesystem output pattern for subagents, plan persistence requirement, cost ceilings per orchestrator run, and the 12 multi-agent failure modes.

#### I10. No plan persistence requirement at S5

Anthropic's June 2025 multi-agent post explicitly notes: "if the context window exceeds 200,000 tokens it will be truncated and it is important to retain the plan." architect-os says "fresh session per ticket" but does not require the plan to be persisted before S6 begins. If the session truncates, the plan is lost.

**Fix:** Add to `lifecycle.md` S5 exit criteria: "Plan persisted to `docs/specs/<slug>/plan.md` before S6 begins." Add to `templates/fsd.md` an explicit plan section.

### Medium priority (fix within 180 days)

#### I11. Memory freshness protocol is too manual

Zep's April 2026 S&P Market Intelligence recognition as the enterprise memory layer validates automatic invalidation: when new evidence contradicts an old fact, the old fact is invalidated at write time. architect-os's protocol invalidates at *review time* (weekly distill).

**Fix:** Add to `memory-freshness-protocol.md` an "automatic invalidation" mechanism: when an agent reads a graph node and discovers it conflicts with current code, it flags the node as `stale: true` immediately, not at next distill. The weekly distill becomes the *confirmation* step, not the *discovery* step.

#### I12. Layer 3 knowledge graph has no automated construction

Aider's repo map (https://docs.aider.ai/) uses tree-sitter + PageRank-style ranking to auto-build a code knowledge graph sized to a token budget. architect-os's `repo-graph.json` is hand-curated via weekly distill. This is correct as a *review* mechanism but labor-intensive as a *construction* mechanism.

**Fix:** Add to `repo-memory.md` Layer 3 a note: "Construction: optional auto-build via tree-sitter + dependency ranking (see Aider). Hand-curation remains the weekly review step." Don't mandate auto-build — keep it optional. The hybrid (LLM proposes graph deltas via dumps, human reviews weekly) is what the existing `memory/dumps/` → `repo-graph.json` pipeline already gestures at; just make the LLM extraction contract explicit.

#### I13. No scale-adaptive spec sizing

BMad v6's "Scale-Domain-Adaptive" mechanism automatically adjusts planning depth based on project complexity. architect-os handles this with "stage-skipping is allowed but explicit" (`lifecycle.md:31`) but doesn't have a mechanism that *automatically* sizes the spec to the work. The result: over-specifying a one-line bugfix because the lifecycle says so.

**Fix:** Add to `lifecycle.md` S0/S1 a "size classification" step: XS (≤1 file, no unknowns) → skip BRD, write acceptance criteria in issue body. S (1–3 files) → 1-paragraph BRD. M (3–5 files) → full BRD. L (>5 files or architectural impact) → full BRD + FSD. Already partially in `adoption-plan.md` profiles but should be in the lifecycle itself.

#### I14. The council pattern is missing for ambiguous decisions

architect-os has no equivalent for ambiguous-decision moments where multiple perspectives matter. The `council` skill (4-voice simulated debate) and BMad v6's "Party Mode" both address this. architect-os's S4 (Architect) is single-perspective.

**Fix:** Add an optional S4 step: "For ambiguous architecture decisions, run a `council` or `party-mode` debate with 3–4 perspectives (e.g., performance, security, maintainability, simplicity). Record the debate in the ADR's 'Alternatives considered' section."

#### I15. No batch refactor pattern in S6

Claude Code's `for file in $(cat files.txt); do claude -p "Migrate $file from React to Vue. Return OK or FAIL." --allowedTools "Edit,Bash(git commit *)"; done` is the standard approach for large mechanical migrations. architect-os treats each migration as N tickets, which is correct but slow. A "batch mode" addition would unlock 10× throughput.

**Fix:** Add to `lifecycle.md` S6 a "Batch mode" subsection: "For mechanical migrations across N files with the same transformation, use `claude -p` in a loop with `--allowedTools` scope. Each file becomes its own commit. Monitor the first 2–3 files manually before running at scale."

#### I16. End-state evaluation is not in the rubric

Anthropic's June 2025 post explicitly recommends "end-state evaluation" for stateful agents — judge whether the agent achieved the correct final state, not whether it followed a "correct" prescribed path. architect-os's `pr-review-rubric.md` is turn-by-turn (check each file, check each criterion). The Anthropic recommendation is to add a primary check: "End state achieved?" — does the diff achieve the spec's stated end state, regardless of path?

**Fix:** Add to `pr-review-rubric.md` minute 0–1: "End-state check: does the diff achieve the spec's stated end state? List any gaps. This is the primary check; everything else is secondary."

### Low priority (consider for next revision)

- **I17.** Adoption-plan.md should reference Kiro and BMad v6 explicitly as alternatives for teams wanting productized surfaces.
- **I18.** Skills-catalog.md should include the BMad v6 module ecosystem (Test Architect, Creative Intelligence Suite) and Spec Kit extension/preset/bundle system as references.
- **I19.** tech-stack.md should be reviewed against current defaults — particularly whether the Node/TypeScript/PostgreSQL/Vercel default still holds in 2026 given the rise of Bun, Hono, Cloudflare Workers, Turso.
- **I20.** The visual-appendix.md should add diagrams for: the multi-agent orchestrator-worker pattern, the three-layer AI review stack, the spec-conformance converge loop.

---

## The five most important things to do this quarter

If only five updates ship in the next 90 days, in priority order:

1. **Update `harness-matrix.md`** — Claude Code async primitives (Routines, Channels, agent teams, Claude Code on the web), the new entrant wave (Devin, Replit Agent, Lovable, Bolt, v0, Windsurf, Trae, Zed AI, Cline, Continue, OpenCode), the three-layer code review stack (CodeRabbit, Greptile TREX, Copilot Code Review now at Business tier). The matrix is the most-read doc and the most stale.

2. **Add Reviewer Independence (C36) and the runtime validation layer to S7.** The correlated-vendor problem is the single biggest correctness risk in the current workflow. C36 is one constitution rule; runtime validation is one optional S7 step.

3. **Add the `/converge` skill and the spec-conformance check at S7.** Anthropic's March 2026 harness-design post is the strongest empirical evidence yet that the evaluator agent is what makes a generator/evaluator harness work. architect-os has the human rubric but no LLM-driven spec-conformance pass.

4. **Add a `multi-agent.md` doc with the decision tree and native Claude Code patterns.** The 20% case is undocumented. Worktrees, agent teams, `claude -p` batch, and the `/code-review` skill are already shipped by Anthropic — architect-os just needs to reference them.

5. **Update `cost-control.md` and `models-cost-quality.md` with Batch APIs, LiteLLM/OpenRouter, automatic caching mode, the `prompt_cache_key` 15 RPM ceiling, and the Aug 31 2026 Sonnet 5 price change.** Cost discipline is what makes the whole system sustainable; the levers got materially better in 2025–2026.

---

## What to *not* change

Resist the temptation to:

- **Increase WIP above 2.** The 2026 evidence validates low WIP, not high. Multi-agent is opt-in for specific cases, not a default.
- **Drop the 150-line AGENTS.md cap.** Anthropic's explicit warning about bloated CLAUDE.md files is stronger now than in 2025. The cap stays; the *content mix* gets pruned.
- **Drop the 400-line PR cap.** No 2026 evidence contradicts this.
- **Replace CodeRabbit with a different reviewer.** CodeRabbit's `path_instructions`, `Learnings`, and `@coderabbitai emit path instructions` are exactly the constitutional hooks architect-os needs. The fix is layering runtime validation *on top of* CodeRabbit, not replacing it.
- **Move to BMad or Spec Kit wholesale.** architect-os's lifecycle is isomorphic to both. The wins are absorbing their good ideas (`/converge`, scale-adaptive sizing, BMad Test Architect), not migrating.
- **Auto-build the knowledge graph without human review.** The hand-curated weekly distill is the quality mechanism. Auto-build is fine as a *proposal* layer; the *review* layer stays human.

---

## Companion reports in this directory

| File | Scope | Lines | Sources fetched |
|---|---|---|---|
| `research-2026-07-update.md` | Cost optimization & failure modes | 475 | 11 |
| `research-2026-07-memory.md` | Memory & context engineering | 438 | 7 |
| `research-2026-07-tools.md` | AI coding agents & tools | 357 | 12 |
| `research-2026-07-specs.md` | Spec-driven development | 208 | 9 |
| `research-2026-07-review.md` | AI code review | 248 | 10 |
| `research-2026-07-multi-agent.md` | Multi-agent orchestration | 293 | 3 + synthesis |

Total: ~2,000 lines of new research across 52 webfetches on primary sources.

---

## Source authority matrix

| Source | Type | Date | Why it matters |
|---|---|---|---|
| Anthropic, *How we built our multi-agent research system* | Engineering blog | Jun 13, 2025 | Reference orchestrator-worker architecture; quantified cost/quality tradeoff |
| Anthropic, *Effective context engineering for AI agents* | Engineering blog | Sep 29, 2025 | Canonical framing of context engineering as a discipline |
| Anthropic, *Best practices for Claude Code* | Official docs | fetched 2026-07-19 | Include/Exclude table for CLAUDE.md; subagent patterns; `/code-review` skill |
| Anthropic, *Harness design for long-running application development* | Engineering blog | Mar 2026 | Empirical evidence: 3-agent harness beats solo by ~20× quality at ~20× cost |
| Anthropic, *Claude Code quality post-mortem (April 23, 2026)* | Incident post-mortem | Apr 23, 2026 | 3 overlapping bugs, 6 weeks of degradation — even the vendor struggles to detect these |
| Anthropic, *Prompt caching docs* | Official docs | fetched 2026-07-19 | Automatic caching mode, 20-block lookback, workspace isolation, cache diagnostics |
| OpenAI, *Prompt caching docs* | Official docs | fetched 2026-07-19 | `prompt_cache_key` 15 RPM ceiling, explicit breakpoints, 30-min TTL, 1.25× write billing |
| OpenAI, *Batch API docs* | Official docs | fetched 2026-07-19 | 50% off, 24h window, separate rate-limit pool |
| Anthropic, *Batch API docs* | Official docs | fetched 2026-07-19 | 50% off, stacks with caching, 100k requests/256MB per batch |
| LiteLLM, *Routing docs* | Official docs | fetched 2026-07-19 | 7 routing strategies, routing groups, weighted failover |
| OpenRouter, *Quickstart* | Official docs | fetched 2026-07-19 | Latest aliases, MCP server at mcp.openrouter.ai |
| GitHub, *Spec Kit repo* | GitHub repo | Jul 17, 2026 | 122k stars, v0.13.0; `/speckit.converge` is the spec-conformance command |
| AWS, *Kiro about page* | Product docs | fetched 2026-07-19 | Productized spec-driven IDE; "agent hooks" for post-implementation async |
| BMad, *BMad Method v6 repo* | GitHub repo | Jul 3, 2026 | Scale-adaptive sizing; Party Mode multi-persona; Test Architect module |
| Cursor, *Self-driving codebases* | Engineering blog | Feb 2026 | `scratchpad.md` should be rewritten frequently, not appended to |
| Mem0 | Official docs | fetched 2026-07-19 | Vector-store memory pattern; Claude Code/Cursor/Codex plugins |
| Letta / MemGPT | Official docs | fetched 2026-07-19 | Block/actor memory; agents write their own blocks |
| Zep | Official docs | fetched 2026-07-19 | Temporal knowledge graphs; automatic invalidation at write time |
| Aider, *Repo map* | Official docs | fetched 2026-07-19 | Tree-sitter + PageRank auto-built repo graph |
| CodeRabbit, *Path Instructions* | Official docs | fetched 2026-07-19 | `path_instructions` + `Learnings` + `@coderabbitai emit path instructions` |
| Greptile, *Auditor post* | Engineering blog | Aug 2025 | Author+reviewer same-vendor is the Arthur-Andersen problem |
| Greptile, *Overnight agents study* | Engineering blog | Apr 2026 | 27.6% of merged PRs AI-authored; agent failure modes differ from human ones |
| Greptile, *TREX announcement* | Engineering blog | Jun 2026 | Runtime validation in sandboxed container; largest capability delta in 2026 review |
| Greptile, *Make LLMs shut up* | Engineering blog | Dec 2024 | Embedding clustering for noise control; 19% → 55% address rate in 2 weeks |
| CrewAI, *Introduction* | Official docs | fetched 2026-07-19 | Flows (DAG) + Crews (role-playing agents); 100k+ developers certified |
| OpenAI Codex repo | GitHub repo | Jul 18, 2026 | 99.5k stars, Rust-based CLI, 929 releases; Codex Web at chatgpt.com/codex |
| Vercel, *v0 spec mode* | Product page | fetched 2026-07-19 | Spec→plan→tasks collapsed into v0 product; creative exploration via parallel UI implementations |

---

*End of synthesis. The five detailed reports in this directory contain the full source-attributed findings behind every claim above. This document is the executive layer; those are the evidence layer.*
