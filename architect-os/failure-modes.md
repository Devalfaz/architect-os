# Failure Modes — The 18 Ways AI-Driven Development Goes Wrong

*Every one of these is real. Every one has happened to real projects. Every one has a mitigation built into the Architect OS. But the mitigations only work if you know what you're guarding against.*

---

## 1. Over-contexting

**What it looks like:** The agent can see the entire repo. It starts "helpfully" improving adjacent code, refactoring modules you didn't touch, updating patterns to be "more consistent." The PR diff is 2,000 lines. You spend an hour reviewing changes you didn't ask for.

**Why it happens:** AI models produce higher-quality output with focused context. But tool designers think "more context = better" and cram the whole repo into the window. The model doesn't know which context is relevant, so it treats all of it as fair game.

**OS mitigation:**
- C6: Implement the ticket, not the repo
- C7: No "while I'm in here"
- C9: PR ≤400 lines
- Fresh Claude Code session per ticket with explicit file list
- Narrow context isn't a cost optimization — it's a quality mechanism

---

## 2. Stale memory

**What it looks like:** An agent confidently uses an API that was removed three versions ago. It imports from a module that was refactored last month. It names entities that haven't existed since the domain model was updated. It writes code that looks right but references the wrong system.

**Why it happens:** The agent's training data is frozen. Without explicit, up-to-date memory, it falls back to what it was trained on — which could be 6–18 months out of date for your codebase.

**OS mitigation:**
- AGENTS.md with `last_verified` dates
- Memory freshness protocol (weekly verify)
- `docs/agents/verified-apis.md` with current API references
- Graph nodes carry `last_verified` — stale nodes flagged
- The weekly distill is the non-negotiable enforcement point

---

## 3. Hallucinated APIs / dependencies

**What it looks like:** The agent imports `fast-csv` (doesn't exist). Uses a method signature that was deprecated two major versions ago. Assumes a function exists because "it should." The code compiles — it's just calling APIs that don't exist.

**Why it happens:** The model patterns-match on common library patterns. If 90% of CSV libraries have a `parse()` method, the model assumes yours does too. It doesn't verify — it completes.

**OS mitigation:**
- S2: `grill-with-docs` verifies every API claim against actual documentation
- `docs/agents/verified-apis.md` documents what's actually installed
- Constitution C8: no new dependencies without ADR line
- `package.json` version pinning + lockfile committed
- Hallucinated APIs caught at typecheck (TypeScript strict mode)

**The diff-review blind spot:** a hallucinated import *looks* plausible in a diff — the function name pattern-matches what the library "should" export. Static review (human or AI) systematically misses this class; only typecheck, tests, or actually running the code catches it. That's why CI-green is a pre-read gate in the rubric, not a nice-to-have: the diff is not evidence the code runs.

---

## 4. Weak tests

**What it looks like:** The PR has tests. They're green. They also mock every dependency, assert implementation details, and test nothing that matters. A month later, a refactor breaks all the tests without changing behavior. The test suite provides false confidence.

**Why it happens:** Agents are good at writing tests that look like tests. They're bad at writing tests that actually verify behavior. Mocking is easy to generate; behavioral testing requires understanding intent.

**OS mitigation:**
- C16: Test behavior, not implementation
- C17: One concept per test
- C19: Test should fail for exactly one reason
- TDD skill enforces behavior-first testing
- Human review checks: are these tests catching what they should?
- Mutation spot-check: deliberately break the implementation, verify tests catch it

---

## 5. Giant PRs

**What it looks like:** A ticket says "implement user dashboard." The agent implements: dashboard page, charts, filters, export, settings panel, notification widget. 2,500 lines. You review for two hours. You miss three bugs. One of them reaches production.

**Why it happens:** Poor ticket decomposition. "Implement user dashboard" is not a ticket — it's a feature. The agent can't refuse; it tries to implement everything.

**OS mitigation:**
- C9: PR ≤400 lines (non-negotiable)
- S5 ticket decomposition: `to-tickets` breaks features into tracer-bullet slices
- Size labels: XS, S, M — `size:split-me` prevents large tickets from reaching agents
- Human gate at S5: you review plan before implementation, not diff after

---

## 6. Architecture drift

**What it looks like:** Each agent PR is individually fine. But after 20 PRs, the codebase no longer follows its own architecture. Business logic leaks into route handlers. The hexagonal architecture has become a hexagon with holes. New patterns appear without ADRs.

**Why it happens:** Agents solve the ticket in isolation. They don't maintain architectural consistency across sessions. Each agent session is a fresh start — without architectural memory, they drift.

**OS mitigation:**
- AGENTS.md codifies architecture decisions
- ADRs with "agent instruction" lines — agents read these every session
- Mechanical compliance checks (lint rules, import boundaries, architecture tests)
- `improve-codebase-architecture` skill runs regularly to detect drift
- Monthly retro: "Is the architecture still consistent?"

---

## 7. Security issues

**What it looks like:** SQL injection in a raw query. API key accidentally logged. User input reaches the database unsanitized. Authorization checked at the route but not at the data layer. Secrets committed to the repo.

**Why it happens:** Training data includes vulnerable code. Models learn patterns indiscriminately — they don't distinguish secure from insecure unless explicitly guided.

**OS mitigation:**
- C31: Secrets never in code
- C32: Input sanitized at boundaries (Zod/Pydantic)
- C33: AuthN and AuthZ are separate, checked per-operation
- C34: Dependencies pinned and auditable
- C35: No sensitive data in logs
- CodeRabbit security scanning on every PR
- `grill-with-docs` verifies auth library API usage

---

## 8. Agent loops / fix-loops

**What it looks like:** The agent hits a test failure. Tries a fix. Test still fails. Tries another fix. Test still fails. Five rounds later, the code is worse than when it started, the session cost is $15, and the problem is still not solved.

**Why it happens:** The agent can't step back and question its approach. Each fix is a local patch on a fundamentally wrong solution. The feedback loop (test fail → try fix → test fail) cycles without convergence.

**OS mitigation:**
- C24: Fix loops bounded at 2 rounds
- Cost ceilings: per-ticket budget limits kill runaway sessions
- Kill signal: 3+ test-fix cycles without green = kill, re-plan
- Bounce back to S5 if fix loop bounds are hit
- The problem is almost always the plan, not the implementation

---

## 9. Review fatigue

**What it looks like:** By the 5th PR of the day, you start approving without reading. By the 15th, you're skimming diffs in 30 seconds. Quality drops. Bugs slip through. The review pipeline becomes a rubber stamp.

**Why it happens:** AI makes it too easy to produce PRs. You're generating 3–5 PRs per day, every day. Reviewing them properly takes 30–50 minutes. Your brain can't sustain attention at that volume.

**OS mitigation:**
- PR size ≤400 lines = reviewable in 10 minutes
- WIP limit: 2 concurrent agent runs
- The 10-minute rubric makes review fast and repeatable
- Human gate at S5 catches problems before they become diff problems
- Monthly metrics track review time — if it's rising, reduce WIP

---

## 10. Silent spec divergence

**What it looks like:** The FSD says "max 100k rows, sync-only." The agent discovers during implementation that async would be better. It silently implements async with a queue. You discover this in review. Now you're debating architecture decisions mid-review instead of mid-planning.

**Why it happens:** Agents make reasonable technical judgments. But without a gate, those judgments accumulate into an implementation that diverges from spec. And nobody notices until review.

**OS mitigation:**
- C1: The spec is the source of truth
- C23: Discovery → spec delta, not silent fix
- S2 grill-with-docs: catches spec ambiguities before implementation
- S5 plan review: catches architectural decisions before they become code
- S7 review checks spec conformance explicitly — the rubric's *first* axis is the end-state check

**The review-stage face of this failure:** AI reviewers judge "is this good code," not "does this implement the linked ticket." A PR can be well-written, secure, tested — and ship the wrong behavior. No commercial reviewer checks ticket conformance by default; the human end-state check in the rubric is the only gate that asks "does the diff achieve what the spec said."

---

## 11. Context window decay

**What it looks like:** The agent produces excellent code for the first 10 minutes. Then quality degrades. It starts repeating itself. It forgets what it already did. It re-implements functionality that already exists in the same session.

**Why it happens:** Context windows fill up. As the session grows (files read, edits made, terminal output, conversation history), older context gets compressed or dropped. The agent loses track of what it's done.

**OS mitigation:**
- Fresh session per ticket (C10)
- Narrow context: only planned files, AGENTS.md, spec section
- `/handoff` skill bridges between sessions
- Kill signal: output quality degradation → new session
- Sessions are cheap to restart; stale sessions are expensive to continue

---

## 12. Abandoned craftsmanship

**What it looks like:** Six months in, you realize you haven't written a line of your own code in weeks. You can't explain how the authentication system works. You don't know where the caching layer is. You've become a review-bot — approving code you couldn't write yourself.

**Why it happens:** The workflow optimizes for agent productivity. It doesn't optimize for your own understanding. Over time, you lose touch with the codebase. When something complex arises, you can't write it yourself. Your review quality degrades because you don't understand the context.

**OS mitigation:**
- You still write some code: complex domain logic, architecture-sensitive code, performance-critical paths. Agents handle the mechanical 80%.
- Human gate at S5: you review plans, which forces you to understand the architecture
- Human gate at S7: you review diffs, which forces you to understand the implementation
- Monthly retro: "Do I still understand this system?"
- The goal is architect-orchestrator, not absentee-landlord

---

## 13. Correlated-model review blind spots

**What it looks like:** Claude Code authors the PR. A Claude-based reviewer approves it. Both missed the same authorization bug — because both are the same model family with the same blind spots. The review ran, the checkmark is green, and it verified nothing.

**Why it happens:** A reviewer built on the author's model inherits the author's failure distribution. The auditor shares scaffolding with the audited. One vendor's data (Greptile, Apr 2026 — single-vendor, not independently verified, but structurally sound regardless) measured Claude-authored code running weak on IDOR/tenancy checks at ~1.75× the human rate, and Cursor-authored code on n+1 queries at ~3.45× — bug classes a same-family reviewer is least likely to catch.

**OS mitigation:**
- C36: the AI second-opinion reviewer must be a different model family than the author — same-family review counts as no review
- Standing routing: Claude authors → Codex review; Codex authors → claude-code-action/CodeRabbit ([review-workflow.md](review-workflow.md))
- Rubric pre-read gate: independence check before the clock starts

**Tripwire:** you notice the "second opinion" and the author resolve to the same vendor. Fix the config before reviewing.

---

## 14. Collateral-damage edits

**What it looks like:** The diff contains a changed line in a file the ticket never mentioned — a reordered import, a rephrased comment, a whitespace "fix," a renamed variable three modules away. Humans don't do this, so human reviewers don't look for it. Real changes hide in the noise.

**Why it happens:** Long-running agents touch what they traverse. Each edit is individually defensible ("improved consistency"); collectively they pollute the diff, defeat blame, and occasionally change behavior.

**OS mitigation:**
- C7 explicitly bans "harmless" adjacent edits, not just refactors
- Rubric pre-read gate: collateral-damage scan — every file in the diff justified by the ticket's plan, unexplained files = bounce
- `git diff --word-diff` when a file's changes look suspiciously broad

**Tripwire:** diff file count > plan file count.

---

## 15. Trust drift — "the AI approved it"

**What it looks like:** You approve faster on PRs where CodeRabbit found nothing. Over weeks, "AI reviewer is quiet" becomes a proxy for "PR is fine." Then a quiet PR ships a business-logic bug that no static reviewer could have caught, and you realize you haven't truly read a diff in a month.

**Why it happens:** Deference is cheaper than attention. The AI's sign-off *feels* like evidence, but AI reviewers catch mechanical issues — they are structurally silent on intent, architecture fit, and spec conformance, which is exactly what the human pass exists for.

**OS mitigation:**
- Rubric rule: do not open AI review comments until your own pass is complete
- The rubric's first axis is the end-state check — a question no AI reviewer answers
- Monthly retro question: "Did I approve anything this month where I deferred to the AI reviewer against my own judgment?"

**Tripwire:** your median review time on AI-quiet PRs drops below half your median on AI-noisy PRs.

---

## 16. Nit flood & address-rate collapse

**What it looks like:** Every PR gets 15 AI review comments. You address three, wave through the rest, and start resenting the reviewer. Within a month you're ignoring all of them — including the occasional critical one buried at comment #11.

**Why it happens:** Raw LLM review comment volume outstrips human attention. Industry data (Greptile, single-vendor): ~19% of unfiltered LLM review comments get addressed; with noise suppression the rate reaches ~55%. Volume without ranking destroys the channel.

**OS mitigation:**
- Severity tags on every constitution rule: 🔴/🟠 as individual comments, all 🟡/🔵 batched into one summary comment ([.coderabbit.yaml](github/.coderabbit.yaml))
- CodeRabbit Learnings + false-positive replies prune recurring noise
- **Address rate** as a tracked metric: comments addressed ÷ comments emitted. Below ~35% for two consecutive weeks → prune rules and re-tune the reviewer, don't push harder

**Tripwire:** you catch yourself resolving AI threads without reading them.

---

## 17. Generated-code silent skip

**What it looks like:** A migration file, a codegen output, or your `memory/repo-graph.json` ships a wrong change — and the AI reviewer never saw it. Its default path filters excluded it as "generated," and nothing told you.

**Why it happens:** Review tools ship default exclusions (`**/*.generated.*`, lock files, JSON blobs) to cut noise. Sensible for actual codegen; silently wrong for generated-*looking* files that carry real decisions — graph memory, config, templates.

**OS mitigation:**
- [.coderabbit.yaml](github/.coderabbit.yaml) explicitly re-includes `docs/**`, `templates/**`, `memory/*.json`
- C30 keeps true generated code tool-owned, so the re-include list stays short
- The rubric's collateral-damage scan reads the *full* file list, including files AI review skipped

**Tripwire:** a reviewed-and-merged PR contains a file no review comment ever referenced.

---

## 18. Same-vendor autofix loop

**What it looks like:** The AI reviewer flags an issue and offers a one-click autofix. You click it. The fix is generated by the same model that just reviewed — if the review was wrong, the fix is wrong the same way. The fix lands unreviewed because "the reviewer wrote it."

**Why it happens:** Reviewer-generated fixes (autofix, generated docstrings, generated tests) feel pre-approved. They're not — they're new unreviewed code from a vendor that just demonstrated a blind spot on this exact PR.

**OS mitigation:**
- Rule: reviewer autofixes are treated as a new unreviewed change — they pass through the same rubric + cross-family check as any commit (noted in [.coderabbit.yaml](github/.coderabbit.yaml))
- C21: agents propose, humans decide — applies to reviewer-agents too

**Tripwire:** a commit authored by the review bot lands without a human review comment on it.

---

## Summary: the failure-mode taxonomy

| Failure mode | Root cause | Stage affected | Primary mitigation |
|---|---|---|---|
| Over-contexting | Broad context | S6 | C6, C7, narrow sessions |
| Stale memory | No freshness protocol | All | Weekly distill, `last_verified` |
| Hallucinated APIs | Model guessing | S6 | S2 grill-with-docs, typecheck |
| Weak tests | Implementation-coupled tests | S6 | C16, C17, TDD skill |
| Giant PRs | Poor decomposition | S5 | C9, `to-tickets`, size labels |
| Architecture drift | No cross-session memory | S4–S6 | AGENTS.md, ADRs, lint rules |
| Security issues | Training data patterns | S6 | C31–C35, CodeRabbit |
| Agent loops | Local optimization trap | S6 | C24, cost ceilings, kill signals |
| Review fatigue | Too many PRs | S7 | WIP limits, rubric readability |
| Silent spec divergence | No discovery gate | S2–S6 | C1, C23, spec deltas |
| Context window decay | Session too long | S6 | Fresh sessions, `/handoff` |
| Abandoned craftsmanship | Over-delegation | All | Human gates, write complex code yourself |
| Correlated-model review | Same-family author+reviewer | S7 | C36, cross-family routing |
| Collateral-damage edits | Agent touches what it traverses | S6–S7 | C7, file-list scan gate |
| Trust drift | Deference to AI sign-off | S7 | Human-pass-first rule, retro check |
| Nit flood | Comment volume > attention | S7 | Severity batching, address-rate metric |
| Generated-code skip | Default reviewer path filters | S7 | Explicit re-includes in `.coderabbit.yaml` |
| Same-vendor autofix | Reviewer fixes feel pre-approved | S7 | Autofix = new unreviewed change |

---

*These failure modes are not hypothetical — they're the battle scars of teams that have adopted AI coding tools at scale. The Architect OS doesn't eliminate them; it mitigates them with specific, enforceable mechanisms. The mitigations work when the rituals are followed. They fail when the rituals are skipped.*

*Fanning out adds a second taxonomy on top of these 18: the 12 multi-agent failure modes (F1–F12 — coordinator bottlenecks, telephone games, shared-state races, cost explosion) live in [multi-agent.md](multi-agent.md).*
