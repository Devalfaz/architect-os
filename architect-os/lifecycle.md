# The Lifecycle — Idea to Production

*The end-to-end workflow. Ten stages, each with an artifact and an exit gate. You are the architect at every gate; agents do the work between them.*

## The shape of the system

```mermaid
flowchart TD
    S0[S0 Capture<br/><i>idea.md</i>] --> S1[S1 Frame<br/><i>BRD</i>]
    S1 --> S2[S2 Specify<br/><i>PRD + FSD</i>]
    S2 --> S3[S3 Design<br/><i>design brief</i>]
    S3 --> S4[S4 Architect<br/><i>architecture.md + ADRs</i>]
    S4 --> S5[S5 Plan<br/><i>GitHub Issues, file-level plans</i>]
    S5 --> S6[S6 Implement<br/><i>narrow agent runs, TDD</i>]
    S6 --> S7[S7 Review<br/><i>human first, AI second</i>]
    S7 -->|fix rounds ≤2| S6
    S7 --> S8[S8 Merge & Release<br/><i>squash, release notes</i>]
    S8 --> S9[S9 Learn<br/><i>memory dump → graph</i>]
    S9 -.->|feeds next cycle| S2
    S6 -.->|discovery → spec delta| S2
```

Three loops matter:

1. **The fix loop** (S7 → S6): bounded at two rounds. A third round means the spec or the plan was wrong — go back to S5, not deeper into the diff. See [failure-recovery-playbook.md](failure-recovery-playbook.md).
2. **The discovery loop** (S6 → S2): implementation always discovers things the spec missed. Discoveries produce *written spec deltas* — never silent divergence. The agent flags it; you amend the FSD; the ticket updates.
3. **The learning loop** (S9 → everything): every merge feeds the memory system, which makes every future stage cheaper and more accurate.

**The gate principle:** each gate is cheap to pass if the previous stage was done honestly, and expensive to skip. Gates are where *you* work. Everything between gates is where agents work. This is what "architect, not babysitter" means operationally: your time concentrates at decision points, not in transcripts.

**Stage-skipping is allowed but explicit.** A one-line bugfix doesn't need a BRD. The rule is: you may skip a stage by *saying so in the ticket* ("no spec needed: trivial"), never by drifting past it. The lightweight profile in [adoption-plan.md](adoption-plan.md) defines standard skips.

---

## S0 — Capture

**Purpose:** get ideas out of your head at zero cost, without committing to anything.

| | |
|---|---|
| **Entry** | any idea, any time |
| **Activities** | 5 minutes, tops. Fill [templates/idea.md](templates/idea.md): the problem, who has it, evidence, kill criteria. |
| **Artifact** | `docs/product/ideas/<slug>.md` |
| **Harness** | none, or a chat with Claude to sharpen phrasing. Not worth an agent run. |
| **Exit gate** | none. Ideas sit in the inbox until weekly triage promotes or parks them. |

The kill-criteria field is the important one: writing down *what would make you drop this* at capture time prevents zombie projects.

## S1 — Frame (BRD)

**Purpose:** decide whether this is worth building, in business terms, before any product thinking.

| | |
|---|---|
| **Entry** | an idea promoted at triage |
| **Activities** | the agent interviews *you* using the grill questions embedded in [templates/brd.md](templates/brd.md) — it asks, you answer, it drafts, you edit. The agent's job is to refuse vague answers. |
| **Artifact** | `docs/product/<slug>/brd.md` |
| **Harness** | Claude Code, interactive session |
| **Skills** | `research` (market/competitor input if needed) |
| **Exit gate** | you can state problem, user, outcome, and non-goals in five sentences without reading the doc. Measurable success metric exists. Go/no-go recorded. |

For personal projects the BRD can be a half page. It still exists, because the non-goals section is what keeps S5 tickets from sprawling later.

## S2 — Specify (PRD → FSD)

**Purpose:** convert intent into a testable contract. This is the highest-leverage stage in the whole system — spec quality is the ceiling on everything downstream.

| | |
|---|---|
| **Entry** | approved BRD |
| **Activities** | 1) optional `research` spike → `docs/research/` with expiry date. 2) optional `prototype` to de-risk the scary part — prototypes are *throwaway by contract*. 3) draft PRD ([templates/prd.md](templates/prd.md)) — numbered FR-x requirements. 4) draft FSD ([templates/fsd.md](templates/fsd.md)) — flows, states, edge-case table, API contracts, Given/When/Then acceptance criteria keyed to FR ids. 5) **grill the spec**: run `grill-with-docs` so every claimed API and library behavior is verified against real documentation, not the model's memory. |
| **Artifacts** | `docs/product/<slug>/prd.md`, `docs/specs/<slug>/fsd.md`, optionally `docs/research/*.md` |
| **Harness** | Claude Code (frontier model — this is not the stage to save money, see [models-cost-quality.md](models-cost-quality.md)) |
| **Skills** | `research`, `prototype`, `to-spec`, `grill-with-docs` |
| **Exit gate** | every acceptance criterion is mechanically testable. The edge-case table is filled (empty = not thought about ≠ none exist). External API claims carry doc links. You would bet a day of work on this spec being right. |

The FSD is the document agents will actually read during implementation. Write it for a competent contractor with no context: if a requirement needs the conversation to be understood, it isn't written yet.

## S3 — Design

**Purpose:** decide what it looks and feels like before code exists, so UI tickets are executable rather than exploratory.

| | |
|---|---|
| **Entry** | approved FSD (for features with UI; headless work skips to S4 — say so) |
| **Activities** | fill [templates/design-brief.md](templates/design-brief.md): screens/states inventory tied to FSD flows, tokens, component mapping to shadcn/ui, a11y requirements. Either Figma-first (design there, pull via Figma MCP) or prototype-first (agent builds throwaway HTML mocks, you react, brief captures the verdict). |
| **Artifact** | `docs/product/<slug>/design-brief.md` + Figma links or `prototypes/` |
| **Harness** | Claude Code + Figma MCP, or v0-style generation for direction-finding |
| **Skills** | `prototype` |
| **Exit gate** | every screen and state in the FSD has a design decision. Empty/loading/error states included — these are what agents invent badly when unspecified. |

## S4 — Architect

**Purpose:** make the decisions that are expensive to reverse, and write them down so agents can't unmake them.

| | |
|---|---|
| **Entry** | spec'd feature that touches architecture, or a new repo |
| **Activities** | 1) `domain-modeling`: nail the ubiquitous language — entity names agents must use, recorded in AGENTS.md. 2) update `docs/architecture/architecture.md` ([template](templates/architecture.md)). 3) one [ADR](templates/adr.md) per irreversible choice, each with an *agent instruction* line and a mechanical compliance check. 4) stack per [tech-stack.md](tech-stack.md) — deviations get an ADR. |
| **Artifacts** | `docs/architecture/architecture.md`, `docs/adr/NNNN-*.md` |
| **Harness** | Claude Code, frontier model, plan mode for exploring options |
| **Skills** | `domain-modeling`, `improve-codebase-architecture` (for existing repos) |
| **Exit gate** | ADRs accepted. Data model reviewed by you, line by line — schema mistakes are the most expensive agent mistakes. New repo: repo bootstrapped per [github-setup.md](github-setup.md), AGENTS.md written. |

## S5 — Plan (Ticket decomposition)

**Purpose:** convert the FSD into tickets so well-specified that implementation is almost mechanical. **This is where you spend the time you saved by not babysitting agents.**

| | |
|---|---|
| **Entry** | approved FSD + architecture |
| **Activities** | run `to-tickets`: decompose into GitHub Issues using [task.yml](github/ISSUE_TEMPLATE/task.yml). Every ticket gets: linked spec section, **file/function-level implementation plan**, out-of-scope list, Given/When/Then acceptance criteria, test plan, size XS/S/M. Order with sub-issues/dependencies. You review every plan — this review is 10x cheaper than reviewing the diff that a bad plan produces. |
| **Artifacts** | GitHub Issues labeled `ai:ready`, sequenced in the Delivery project |
| **Harness** | Claude Code against the repo (it must read actual code to name actual files); Traycer is the managed alternative for this exact step — see [harness-matrix.md](harness-matrix.md) |
| **Skills** | `to-tickets`, `wayfinder` (locate the right code first) |
| **Exit gate** | every ticket ≤1 day (size M max; `size:split-me` means split it). File lists name real files. No ticket depends on an unmade decision — those get `needs-decision` and stop here. You'd stake a code review on each plan being right. |

## S6 — Implement

**Purpose:** turn one ticket into one reviewable PR, with the smallest context that suffices.

| | |
|---|---|
| **Entry** | ticket labeled `ai:ready`, its dependencies merged |
| **Activities** | one issue = one branch (`feat/123-slug`) = one PR. Launch a **fresh** Claude Code session per ticket: context = AGENTS.md + the ticket + linked FSD section + files in the plan. Nothing else. TDD per the `tdd` skill: failing test first, then implementation, per [constitution](constitution.md) testing rules. Agent self-reviews its own diff and posts the self-review comment before requesting review. Fully-specified XS tickets can go async to Copilot coding agent instead. WIP limit: 2 concurrent runs. |
| **Artifacts** | a PR ≤400 lines, linked `closes #123`, CI green, self-review comment posted |
| **Harness** | Claude Code (primary); Copilot coding agent (async XS); Codex CLI (second implementation when you want to compare approaches) |
| **Skills** | `implement`, `tdd`, `diagnosing-bugs` (when tests fail unexpectedly) |
| **Exit gate** | CI green, PR template complete, diff touches only planned files (deviations flagged per constitution). |

Narrow context is not a cost optimization — it's a *quality* mechanism. An agent that can see the whole repo will helpfully "improve" things you didn't ask about; an agent that sees one ticket's worth of context physically can't.

## S7 — Review

**Purpose:** two-stage review, human strictly first. Full pipeline in [review-workflow.md](review-workflow.md).

| | |
|---|---|
| **Entry** | PR open, CI green, self-review posted |
| **Activities** | 1) **You** run the [10-minute rubric](pr-review-rubric.md) *before reading any AI comments* — reading AI review first anchors you on mechanical issues and blinds you to design drift. 2) then CodeRabbit auto-review + a second AI opinion (Codex review or claude-code-action). 3) fix loop: the agent answers every thread with a commit SHA or reasoned pushback; **you** resolve threads; two rounds max. |
| **Artifacts** | review threads, fix commits |
| **Harness** | you + CodeRabbit + Codex/Claude action |
| **Skills** | `code-review` (for the agent's self-review pass) |
| **Exit gate** | your approval + AI findings addressed + CI green. Any constitution violation cited by rule id. |

## S8 — Merge & Release

**Purpose:** land it, ship it, write down what shipped.

| | |
|---|---|
| **Activities** | squash merge (PR title = conventional commit). Deploy via CI (Vercel preview → production). Smoke-check the deployed change yourself. Batch release notes per [template](templates/release-notes.md) at milestone close. |
| **Artifacts** | merged main, deployment, release notes |
| **Exit gate** | deployed and smoke-checked. Broken main = revert first, diagnose second ([playbook](failure-recovery-playbook.md)). |

## S9 — Learn

**Purpose:** convert the session's experience into durable memory. The stage everyone skips, and the reason everyone's agents stay dumb.

| | |
|---|---|
| **Activities** | fill [templates/memory-dump.md](templates/memory-dump.md) (5 min, evening): what changed, decisions, surprises, hallucinations caught, graph delta. Weekly: distill dumps → `memory/repo-graph.json` + docs updates ([freshness protocol](memory-freshness-protocol.md)). Friction with the *workflow itself* → retro file → skill updates ([rituals](rituals-and-metrics.md)). |
| **Artifacts** | `memory/dumps/YYYY-MM-DD-*.md`, graph delta, updated docs |
| **Harness** | Claude Code, cheap model — this is mechanical |
| **Skills** | `memory-dump`, `graph-update`, `retro`, `handoff` (end of multi-day efforts) |
| **Exit gate** | none daily; weekly distill is the enforcement point. |

---

## Where your time goes

| Stage | Your time | Agent time |
|---|---|---|
| S0 Capture | 5 min | — |
| S1 Frame | 30 min (answering grills) | drafting |
| S2 Specify | 1–2 h (editing, gate) | drafting, grilling docs |
| S3 Design | 30–60 min (taste calls) | prototyping |
| S4 Architect | 1 h (decisions) | options analysis |
| S5 Plan | **1–2 h (plan review — the big one)** | decomposition |
| S6 Implement | ~0 (launch + glance) | the bulk |
| S7 Review | 10–15 min per PR | AI review, fixes |
| S8 Release | 10 min | notes, deploy |
| S9 Learn | 5 min/day + 30 min/week | distillation |

If your S6/S7 time balloons, the problem is upstream in S2/S5 — fix the spec process, don't review harder. That asymmetry is the entire thesis of this OS.
