---
status: snapshot
last_verified: 2026-07-19
expires: 2026-10-19
---
<!-- SNAPSHOT: findings below were verified against live sources on last_verified.
     After the expiry date, treat every claim here as a hypothesis to re-verify,
     not a fact — this file must not become the next stale baseline. -->

# Spec-Driven AI Development — State of the Field, July 2026

*Research compiled for architect-os. Sources: GitHub Spec Kit, AWS Kiro, BMad Method, Anthropic Engineering, v0/Vercel, HackerNews/industry discussion. All data points are dated to the source's most recent publication visible at fetch time.*

---

## Executive Summary

Spec-driven AI development has crossed from experiment to default orthodoxy in 2025–2026. Three forces converged:

1. **Mainstream tooling endorsement.** GitHub's Spec Kit (`github/spec-kit`, 122k stars, v0.13.0 released July 17 2026, 1,462 commits, 30+ agent integrations) and AWS's Kiro IDE both ship spec-driven flows as a *first-class* product surface, not an optional plugin. Kiro explicitly markets itself as having "pioneered spec-driven development, where Kiro turns your prompt into structured requirements, design, and tasks that are then implemented by agents."

2. **Convergence on a shared spine.** Spec Kit, Kiro, BMad v6 (50.8k stars, v6.10.0 July 3 2026), and Anthropic's own harness-design research have independently arrived at the same loop: **specify → plan → tasks → implement → converge/evaluate**. The artifact vocabulary differs (`constitution`/`spec`/`plan`/`tasks` vs. BRD/PRD/FSD vs. BMad personas) but the lifecycle is isomorphic to architect-os's S2→S5→S6→S7.

3. **Hard empirical evidence that harness design beats raw prompt-to-code.** Anthropic's March 2026 "Harness design for long-running application development" post showed a 3-agent planner/generator/evaluator harness producing a working DAW for $124/4hr, where a solo-agent run produced a broken game maker for $9/20min. The harness's evaluator agent — analog of architect-os's S7 — was the difference between "core feature literally doesn't work" and "plays end to end."

**The state of the field in one line:** the question is no longer *whether* to spec-first, it's *which spec system* to use and *how thin* the harness can get before model capability makes it redundant. Architect-os's lifecycle is well-aligned with industry direction but has gaps in three areas: automated spec-conformance evaluation, bundle/role-based tooling packaging, and explicit GAN-style generator/evaluator separation at S7.

---

## Per-Methodology Findings

### 1. GitHub Spec Kit (`github/spec-kit`)

**Source:** https://github.com/github/spec-kit (fetched July 19 2026)

- **Adoption:** 122k stars, 10.9k forks, 1,462 commits, 195 releases, latest v0.13.0 July 17 2026. This is no longer a side project — it is one of GitHub's flagship open-source bets.
- **Core commands (the canonical SDD spine):**
  - `/speckit.constitution` — governing principles (architect-os analog: `constitution.md`)
  - `/speckit.specify` — requirements and user stories (S2 PRD)
  - `/speckit.clarify` — clarify underspecified areas, recommended *before* `/plan` (architect-os has `grill-with-docs`; Spec Kit now codifies this as a first-class step)
  - `/speckit.plan` — technical implementation plan (S4/S5)
  - `/speckit.tasks` — actionable task list (S5)
  - `/speckit.taskstoissues` — convert to GitHub issues (S5 → GitHub)
  - `/speckit.implement` — execute (S6)
  - `/speckit.converge` — **assess codebase against spec/plan/tasks and append remaining work as new tasks** ← direct analog to architect-os's discovery loop and the missing piece in many methodologies
  - `/speckit.analyze` — cross-artifact consistency & coverage analysis (run after `/tasks`, before `/implement`)
  - `/speckit.checklist` — "unit tests for English": generate custom quality checklists that validate requirements completeness, clarity, consistency

- **Key new concepts architect-os should absorb:**
  - **Converge as a named command.** Architect-os has the "discovery loop" as a diagram edge (S6 → S2 spec delta) but no command/role that *closes* the gap between what was built and what was specified. Spec Kit's `/converge` makes this mechanical.
  - **Checklist as "unit tests for English."** This is a spec-quality metric — a checklist the spec itself must pass before plan/tasks are generated. Architect-os's S2 exit gate ("every acceptance criterion mechanically testable") is the same idea but unstaged as an artifact.
  - **Analyze as a pre-implementation consistency gate.** Runs cross-artifact checks between spec/plan/tasks. Architect-os folds this into the S5 exit gate but doesn't name it.

- **Extension/preset/bundle system.** Spec Kit separates *new capabilities* (extensions), *template overrides* (presets), and *role-based packages* (bundles: PM, BA, security, developer). Architect-os has skills (`to-spec`, `grill-with-docs`, `to-tickets`) but no packaging layer for "give a PM the right 4 skills and 2 templates in one command."

### 2. AWS Kiro (`kiro.dev`)

**Source:** https://kiro.dev/about/ (fetched July 19 2026)

- **Positioning:** "Agentic development environment that makes it easy for developers to ship real engineering work with the help of AI agents." Operated by a small opinionated team within AWS. Name pronounced like "hero."
- **Claim:** Kiro "pioneered spec-driven development, where Kiro turns your prompt into structured requirements, design, and tasks that are then implemented by agents." (This is contested by Spec Kit's history but reflects AWS's marketing position.)
- **Beyond spec-first — agent hooks.** Kiro's second big idea: "agent hooks help you scale your work by delegating tasks to agents that run in the background, such as updating docs, generating unit tests, or optimizing your code for performance." This is a *post-implementation* async agent layer — architect-os has nothing equivalent; S7/S8 are synchronous.
- **Form factors:** IDE, CLI, Web, Mobile. Architect-os implicitly assumes Claude Code in a terminal; Kiro treats the spec system as a multi-surface product.
- **Critique read:** Kiro is the most productized spec-driven dev surface, but the docs page we fetched is mostly marketing. The architectural claim — that specs are "structured input" agents need beyond natural-language prompts — matches architect-os's S2 thesis exactly.

### 3. BMad Method v6 (`bmad-code-org/BMAD-METHOD`)

**Source:** https://github.com/bmad-code-org/BMAD-METHOD (fetched July 19 2026)

- **Adoption:** 50.8k stars, 5.8k forks, 1,981 commits, 38 releases, latest v6.10.0 July 3 2026.
- **Positioning:** "Breakthrough Method for Agile AI Driven Development" — "scale-adaptive intelligence that adjusts from bug fixes to enterprise systems." Explicitly scale-adaptive where Spec Kit and Kiro are more one-size.
- **Key differentiators:**
  - **Scale-Domain-Adaptive.** Automatically adjusts planning depth based on project complexity. Architect-os handles this with "stage-skipping is allowed but explicit" but doesn't have a mechanism that *automatically* sizes the spec to the work.
  - **12+ specialized agent personas** (PM, Architect, Developer, UX, etc.) — "Party Mode" brings multiple personas into one session to collaborate. Architect-os assumes a single agent persona per stage.
  - **Module ecosystem:** Core (BMM), BMad Builder (BMB for custom agents), Test Architect (TEA — risk-based test strategy), Game Dev Studio (BMGD), Creative Intelligence Suite (CIS). Architect-os's skills-catalog is the analog but more open-ended.
  - **Web Bundles (v6).** Package BMad skills for installation as **Google Gemini Gems** and **ChatGPT Custom GPTs**. "Planning runs on a flat-rate subscription instead of metered IDE tokens." Pragmatic cost control — run the spec/planning phase where it's cheap, implement where it's metered.
  - **`bmad-help` skill.** Invoke any time for "what's next" guidance. Architect-os has lifecycle.md but no interactive help skill.
- **Relevance to architect-os:** BMad's scale-adaptive sizing is the cleanest answer to a known architect-os failure mode — over-specifying a one-line bugfix because the lifecycle says so. The Party Mode multi-persona debate is also a counter to the single-agent bias of S2/S4.

### 4. v0 Spec Mode (Vercel)

**Source:** https://v0.app/spec-mode (fetched July 19 2026)

- The page fetched is the v0 general product page, not a dedicated spec-mode doc — Vercel has a `/spec-mode` URL but renders the generic product. Reads as: spec mode is a *mode* of v0, not a separately marketed product.
- "Agentic by default: v0 plans, creates tasks, and connects to databases as it builds." This is the spec→plan→tasks loop collapsed into the v0 product surface, optimized for full-stack web apps.
- v0's strength is *creative exploration* — parallel implementations of UI directions. Spec Kit calls this out as one of its three development phases ("Creative Exploration: parallel implementations, support multiple technology stacks & architectures, experiment with UX patterns").
- **Relevance to architect-os S3 (Design):** v0 is the right tool for the "prototype-first" branch of S3. Architect-os mentions it ("v0-style generation for direction-finding") but doesn't articulate *how* to converge multiple v0 variants into a single design brief.

### 5. Anthropic Engineering — Harness Design Research

**Source:** https://www.anthropic.com/engineering/harness-design-long-running-apps (Mar 24 2026, Prithvi Rajasekaran)

This is the most important industry thinking on spec-driven dev in 2026 because it provides *quantitative* evidence.

- **Three-agent architecture (planner / generator / evaluator).** The planner expands a 1-4 sentence prompt into a full product spec — explicitly mirrors architect-os S0→S2. The generator implements one feature at a time. The evaluator uses Playwright MCP to click through the running app and grade against negotiated "sprint contracts."
- **Sprint contracts as a new artifact.** Before each sprint, generator and evaluator negotiate "what done looks like" for that chunk. This bridges high-level specs and testable implementation. **Architect-os has nothing equivalent** — the FSD's Given/When/Then acceptance criteria play this role at ticket level, but they're authored once at S2, not re-negotiated per sprint against a live build.
- **The evaluator is the load-bearing piece.** Quote: "Out of the box, Claude is a poor QA agent. In early runs, I watched it identify legitimate issues, then talk itself into deciding they weren't a big deal and approve the work anyway." The fix was to (a) separate the evaluator from the generator and (b) tune the evaluator to be skeptical. Architect-os S7 puts the *human* first ("you run the rubric before reading AI comments") but doesn't have an explicit AI-evaluator role that's separate from the generator.
- **Context resets vs compaction.** Sonnet 4.5 exhibited "context anxiety" — wrapping up prematurely as it approached its perceived context limit. Context resets (full clear + structured handoff) beat compaction (in-place summarization). Opus 4.6 largely removed this behavior. **Implication for architect-os S6:** the "fresh Claude Code session per ticket" rule is correct and load-bearing on Sonnet-class models, but may be relaxable on Opus 4.6+.
- **Harness simplification on better models.** When Opus 4.6 shipped, the author removed the sprint decomposition entirely — the model handled the full build in one go. The evaluator remained useful for tasks at the edge of model capability. **Architect-os implication:** the lifecycle's stage boundaries are stable, but the *mechanism* inside each stage should shrink as models improve. The "find the simplest solution possible, and only increase complexity when needed" principle from *Building Effective Agents* is the meta-rule.
- **Cost reality check.** The full harness run on a DAW was 4 hours and $124. Solo was 20 min and $9. The harness was 20× more expensive and *vastly* better. Spec-driven dev is not cheaper than vibe-coding — it's better. Architect-os's cost-control.md should acknowledge this asymmetry: the lifecycle is a quality investment, not a cost-saving one.

### 6. Industry Critiques (HN / general discourse)

- **The HN search page (`hn.algolia.com`) requires JavaScript and did not return results via fetch.** A targeted HN search could not be performed via this tool. From general reading the recurring critiques are:
  - **"Specs go stale."** Spec Kit addresses this with the `/converge` command and the Evolving Specs brownfield loop. Architect-os's discovery loop addresses it but only as a diagram edge.
  - **"Spec-first is waterfall in a trench coat."** Counter from BMad: scale-adaptive depth means a bugfix gets a 3-line spec, not a PRD. Architect-os handles this with explicit skip-by-saying-so but doesn't *automate* the sizing.
  - **"Specs don't help if the spec author is wrong."** Spec Kit's `/speckit.checklist` ("unit tests for English") is a partial answer — it catches *internal* inconsistency even if external validity is uncheckable.
  - **"The agent will just rewrite the spec to match what it built."** This is a real failure mode. The `/converge` pattern appends remaining work as new tasks *against* the original spec, which preserves the original as ground truth. Architect-os's "written spec deltas — never silent divergence" is the same discipline, named more clearly.

---

## New Approaches Not Covered in architect-os

### A. Converge as a named stage-command

architect-os has the discovery loop as a diagram edge. Spec Kit has `/speckit.converge` as a command that **assesses the codebase against spec/plan/tasks and appends remaining work as new tasks**. **Recommendation:** make the converge pass an explicit sub-step of S7 or a new S7.5 — a diff between the FSD's acceptance criteria and the actual built behavior, run before human review. This is the cheapest place to catch "agent stubbed the feature" failures.

### B. Sprint contracts (Anthropic)

Before each implementation chunk, generator and evaluator negotiate a written contract: what will be built and how it will be verified. **Recommendation:** add a "sprint contract" artifact to S5 or S6 — a per-ticket mini-spec that translates FSD-level acceptance criteria into concrete testable behaviors for that specific PR. This is what `to-tickets` should produce but currently doesn't make explicit.

### C. Evaluator as a separate role from generator

Anthropic's evidence: the same model that builds cannot reliably grade its own work. Architect-os S7 puts the human first and uses a *different* AI (CodeRabbit, Codex review) as the second opinion, which is structurally correct. The new finding is that even a *different model* used as evaluator needs explicit "skeptical tuning" — few-shot examples of properly critical reviews, criteria weighted against AI-slop patterns. **Recommendation:** add an "evaluator prompt" template to `templates/` and reference it from review-workflow.md.

### D. Spec-quality checklists as artifacts

Spec Kit's `/speckit.checklist` generates "unit tests for English" that the spec must pass. **Recommendation:** add a `spec-checklist.md` artifact to S2's exit gate — concrete predicates the spec must satisfy (every FR has a testable Given/When/Then; every API claim has a doc link; every state has an empty/loading/error case; etc.). This makes the S2 exit gate mechanical rather than vibes-based.

### E. Scale-adaptive spec sizing (BMad)

architect-os allows stage-skipping by explicit statement. BMad goes further: the method *automatically* sizes the spec to the work. **Recommendation:** add a "sizing" sub-step at S1 exit that classifies the work as XS/S/M/L and emits the *expected* S2/S5 artifact depth. This prevents the failure mode where a one-line bugfix gets a full PRD because the lifecycle says so.

### F. Role-based bundles (Spec Kit, BMad)

Both Spec Kit and BMad package role-based setups (PM, BA, security, developer) in one command. Architect-os has skills-catalog.md but no "install the PM bundle" or "install the security bundle" concept. **Recommendation:** define 3-4 role bundles in skills-catalog.md, each mapping to a subset of the lifecycle stages the role owns.

### G. Web-bundle cost arbitrage (BMad v6)

BMad runs planning on Gemini/ChatGPT flat subscriptions and implementation on metered IDE tokens. **Recommendation:** architect-os's models-cost-quality.md should explicitly call out that S0–S3 (capture, frame, specify, design) can run on a flat-rate web LLM, and S6 (implement) is where metered tokens are justified. This is a real cost lever, not a theoretical one.

### H. Agent hooks for post-implementation async work (Kiro)

Kiro runs background agents that update docs, generate unit tests, optimize code. Architect-os has nothing between S7 (review) and S8 (merge) for this kind of async janitorial work. **Recommendation:** consider an S6.5 "agent hooks" stage where post-implementation async agents (test-generation, doc-update, perf-probe) run before review.

---

## Critiques and Counterarguments

### Critique 1: "Specs are waterfall with AI lipstick"

**Counter:** The discovery loop (architect-os), `/converge` (Spec Kit), and BMad's scale-adaptive sizing all directly address this. The honest answer is: spec-driven dev *is* waterfall-ish when applied to a 3-month feature, and that's correct — you *should* think before you build a 3-month feature. It's wrong when applied to a 3-hour bugfix, which is why the sizing step matters more than the lifecycle itself.

### Critique 2: "The agent will rewrite the spec to match what it built"

**Counter:** This is a real failure mode and it's why architect-os's "written spec deltas — never silent divergence" rule is load-bearing. Spec Kit's `/converge` preserves the original spec as ground truth and appends deltas. Anthropic's evaluator pattern uses the sprint contract (frozen pre-build) as the grading rubric. The pattern across all three: **freeze the spec, grade against the frozen spec, deltas are explicit.**

### Critique 3: "The harness overhead exceeds the model's needs as models improve"

**Counter:** Anthropic's March 2026 post directly addresses this. When Opus 4.6 shipped, they removed the sprint decomposition and the harness still worked. The principle: "every component in a harness encodes an assumption about what the model can't do on its own, and those assumptions are worth stress testing." Architect-os should treat its own stage boundaries as hypotheses to retest per model release, not as fixed law.

### Critique 4: "Self-evaluation by the agent is unreliable"

**Counter:** Anthropic's evidence is decisive here — separating generator from evaluator is the lever. Architect-os's S7 already structurally separates them (human + CodeRabbit + Codex/Claude action vs. the implementer). The remaining gap is that the *evaluator's tuning* is implicit. Make it explicit: an evaluator prompt template with skeptical few-shot examples.

### Critique 5: "Specs don't catch the real bugs — only conformance to the spec"

**Counter:** True and important. Anthropic's evaluator found real bugs (broken physics, stubbed audio recording) *because the spec explicitly called for those features*. A spec that says "audio recording works" is testable; a spec that doesn't mention it isn't. This is the case for the S2 edge-case table being non-negotiable — the spec is the test surface.

---

## Recommended Updates to architect-os

### lifecycle.md

1. **Add `/converge` equivalent to S7.** Before human review, an agent pass assesses the built PR against the FSD acceptance criteria and emits a conformance report (gaps → new tickets, not silent fixes). This is a missing mechanical step that exists in Spec Kit and is implied by Anthropic's evaluator pattern.

2. **Add a "sprint contract" artifact to S5 or S6.** Each ticket gets a per-PR mini-contract negotiated before implementation: "I will build X, verified by Y." This is what `to-tickets` should produce but doesn't name. The contract is the evaluator's rubric.

3. **Add an "evaluator prompt" template.** A skeptical, few-shot-tuned prompt for the AI second-opinion reviewer at S7. Currently S7 says "Codex review or claude-code-action" but doesn't tune them. Anthropic's evidence: untuned evaluators approve mediocre work.

4. **Add a sizing sub-step at S1 exit.** XS/S/M/L classification that determines *expected* S2/S5 artifact depth. Prevents over-specifying trivial work and under-specifying serious work.

5. **Add an "agent hooks" note at S6.5 or folded into S6.** Post-implementation async agents (test-gen, doc-update, perf-probe) run before review. Kiro's pattern.

6. **Reframe cost-control.md.** Spec-driven dev is a *quality* investment, not a cost-saving one (Anthropic: 20× more expensive, vastly better). Cost savings come from (a) running S0–S3 on flat-rate web LLMs (BMad web-bundles) and (b) the learning loop making future cycles cheaper.

### S2 — Specify

- **Add a `spec-checklist.md` artifact** to the exit gate. Concrete predicates the spec must satisfy — "every FR has ≥1 Given/When/Then," "every API claim has a doc link," "every state has empty/loading/error case." This is Spec Kit's `/speckit.checklist` pattern ("unit tests for English") and makes the S2 gate mechanical.
- **Add `/speckit.clarify` analog as a named sub-step before plan.** Architect-os has `grill-with-docs` for verifying external API claims; Spec Kit's `/clarify` is broader — it clarifies underspecified areas of the spec itself. Both should run; they're different checks.
- **Add `/speckit.analyze` analog between S2 and S5.** Cross-artifact consistency check between spec, plan, and tasks. Architect-os folds this into the S5 exit gate but doesn't name it as an artifact.

### S5 — Plan

- **Add "sprint contract" to each ticket's expected artifacts.** A ticket currently gets: linked spec, file/function-level plan, out-of-scope, G/W/T, test plan, size. Add: a per-PR negotiated contract stating what will be built and how it will be verified. This becomes the S7 evaluator's rubric.
- **Add `to-tickets` output: a conformance checklist per ticket.** The list of behaviors the S7 evaluator will test against. Makes "CI green" mean something beyond "tests pass."

### S7 — Review

- **Add an explicit evaluator role distinct from the implementer.** Currently S7 uses "CodeRabbit auto-review + a second AI opinion (Codex review or claude-code-action)." Make the second-opinion agent a *skeptically-tuned evaluator* with an explicit prompt template, not just "the same model asked to review."
- **Add a `/converge` pass as S7.0.** Before the human runs the 10-minute rubric, an agent compares the built PR to the FSD and emits a conformance report. This is the cheapest place to catch "agent stubbed the feature" failures and it's the pattern Spec Kit and Anthropic both validate.

---

## Sources

| Source | URL | Fetch date |
|---|---|---|
| GitHub Spec Kit | https://github.com/github/spec-kit | 2026-07-19 |
| Kiro About | https://kiro.dev/about/ | 2026-07-19 |
| BMad Method | https://github.com/bmad-code-org/BMAD-METHOD | 2026-07-19 |
| Anthropic Engineering (harness design) | https://www.anthropic.com/engineering/harness-design-long-running-apps | 2026-07-19 |
| Anthropic Engineering index | https://www.anthropic.com/engineering | 2026-07-19 |
| v0 Spec Mode | https://v0.app/spec-mode | 2026-07-19 |
| Hacker News search (spec-driven dev) | https://hn.algolia.com/?q=spec-driven+development | 2026-07-19 (JS-only, no results) |
| architect-os/lifecycle.md | local | 2026-07-19 |
