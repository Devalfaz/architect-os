# Skills Catalog — Repeatable Agent Workflows

*The library of skills that make up Architect OS. From Matt Pocock's ecosystem + custom OS-native skills. Skills versioned in git under `~/.claude/skills/`.*

---

## Skill architecture

| Type | Who invokes | Context cost |
|---|---|---|
| User-invoked | You type it | Zero (description never loaded) |
| Model-invoked | Agent reaches autonomously | Description sits in context |

**Source:** mattpocock/skills architecture.

---

## Core Skills (every project, every day)

### `/grill-with-docs` (user-invoked)
Relentless one-question-at-a-time interview. Sharpens thinking while building the domain model. Stateful — retains what it learns. **Use:** EVERY time you want to make a change. **OS mapping:** S1, S2, S4.

### `/to-spec` (user-invoked)
Turns conversation into a formal spec. Synthesizes — no new interview. Publishes to issue tracker. **Use:** when grilling is done and shared understanding reached. **OS mapping:** S2.

### `/to-tickets` (user-invoked)
Breaks spec into tracer-bullet tickets with blocking edges. Publishes in dependency order. **Use:** after spec exists. **OS mapping:** S5.

### `/implement` (user-invoked)
Builds work from spec/tickets. Drives `/tdd` internally, runs typecheck, closes with `/code-review`. **Use:** when ticket is `ready-for-agent`. **OS mapping:** S6.

### `/tdd` (model-invoked)
Red-green-refactor loop. Behavior-through-public-interfaces testing. **Agent fires:** when implementing per C4. Pulled by `/implement`. **OS mapping:** S6.

### `/code-review` (model-invoked)
Two-axis review run as parallel sub-agents: Standards (code smells) + Spec (does it match?). **Agent fires:** before committing per C22. Pulled by `/implement`. **OS mapping:** S7 self-review.

### `/handoff` (user-invoked)
Compacts conversation into handoff doc for fresh session. Bridges context windows. **Use:** multi-session work.

---

## High-Value Situational Skills

### `/diagnosing-bugs` (model-invoked)
6-phase diagnosis loop. Refuses to theorize without a red-capable command first. **Use:** something broken/throwing/failing/slow. **OS mapping:** bug recovery.

### `/triage` (user-invoked)
Moves issues through state machine: `needs-triage` → `needs-info` / `ready-for-agent` / `ready-for-human` / `wontfix`. **Use:** incoming issues from others. **OS mapping:** issue pipeline.

### `/wayfinder` (user-invoked)
Plans work too big for one session. Charts decision tickets on the tracker. **Use:** greenfield, huge features, foggy efforts. **OS mapping:** large refactors, greenfield architecture.

### `/improve-codebase-architecture` (user-invoked)
Scans for deepening opportunities. Visual HTML report of candidates. **Use:** every few days to counter AI-speed entropy. **OS mapping:** S4 existing repos, weekly rituals.

### `/prototype` (model-invoked)
Throwaway prototype. Logic (terminal) or UI (URL-param variations). **Use:** when a design question can't be settled on paper. **OS mapping:** S2 de-risk, S3 UI exploration.

### `/research` (model-invoked)
Background agent investigates primary sources, writes cited markdown. **Use:** need docs/API facts gathered. **OS mapping:** S1 market, S2 doc verification.

### `/domain-modeling` (model-invoked)
Builds and sharpens domain model. Updates CONTEXT.md, creates ADRs. **Use:** pulled by grill/triage/wayfinder. **OS mapping:** S4 ubiquitous language.

### `/codebase-design` (model-invoked)
Deep-module vocabulary (depth, seam, adapter, leverage). Design-it-twice pattern. **Use:** designing module interfaces. **OS mapping:** S4, S6.

---

## Meta & Support

### `/ask-matt` — Router skill. Use when you forget which skill to reach for.
### `/setup-matt-pocock-skills` — One-time-per-repo scaffold. Pre-S0 bootstrapping.
### `/writing-great-skills` — Reference for creating/improving skills.
### `/grill-me` — Same interview as grill-with-docs, but stateless (no codebase).
### `/resolving-merge-conflicts` — Works through merge conflicts by intent.

---

## OS-Native Skills (custom, in `~/.claude/skills/architect-os/`)

### `/memory-dump`
Generates structured daily memory dump. Cheapest viable model. **Use:** end of every coding day (S9).

### `/graph-update`
Reads week's dumps, produces graph delta PR. **Use:** weekly distill.

### `/retro`
Structured retrospective: what worked, what didn't, what to change. **Use:** monthly.

---

## The Main Flow

```
S0 Capture → manual
S1 Frame → /grill-with-docs + /research → BRD
S2 Specify → /research + /prototype + /to-spec + /grill-with-docs → PRD + FSD
S3 Design → /prototype → design brief
S4 Architect → /domain-modeling + /improve-codebase-architecture → ADRs
S5 Plan → /to-tickets + /wayfinder → GitHub Issues
S6 Implement → /implement → /tdd → /code-review (×N tickets, fresh context each)
S7 Review → human rubric → CodeRabbit → Codex review
S8 Release → CI/CD
S9 Learn → /memory-dump → /graph-update → /retro
```

### On-ramps

```
Bugs → /triage → agent-ready → /diagnosing-bugs → fix + regression
Foggy → /wayfinder → decisions → clear map → /to-spec → main flow
Entropy → /improve-codebase-architecture → candidate → main flow
```

---

## Skill versioning

All skills live in `~/.claude/skills/` under git. Versioning is mandatory.
- Pocock skills: installed via `npx skills@latest add mattpocock/skills`. Check monthly.
- OS-native skills: your own. Version alongside Pocock's.
- Retro-driven edits: when retro identifies friction, edit the skill, commit, update changelog.

**Source:** mattpocock/skills GitHub repository, skill files in skills/engineering/ and skills/productivity/.
