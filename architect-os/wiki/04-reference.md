# 04 — Reference Card

*The numbers, gates, and states on one page. If a number here disagrees with a canonical doc, the doc wins.*

---

## The limits (the system IS the limits)

| Limit | Value | Source |
|---|---|---|
| PR size | ≤400 lines | C9 |
| Ticket size | ≤1 day (M max; bigger = `size:split-me`) | lifecycle S5 |
| Fix rounds | ≤2, then bounce to S5 | C24 |
| WIP | ≤2 concurrent agent runs | daily-loop |
| Human rubric | 10 min (20 if C36 unavailable) | pr-review-rubric |
| AGENTS.md | ≤150 lines | repo-memory Layer 1 |
| Implementation-run starting context | <~20k tokens of repo content | repo-memory |
| Correction attempts before restart | 2 | failure-recovery playbook |

## The gates (where your time goes)

| Gate | You ask | Doc |
|---|---|---|
| S1 exit | Worth building? Sized XS/S/M/L? | lifecycle |
| S2 exit | Every AC mechanically testable? Edge-case table honest? APIs grilled? | lifecycle |
| S5 exit | Files real? Plan persisted? Would I stake a review on it? | lifecycle |
| S7 converge | PASS? (M+; auto-bounces on FAIL) | skills/converge |
| S7 rubric | Pre-read gates → end-state first → constitution → architecture → tests | pr-review-rubric |
| S8 | Deployed + smoke-checked by me? | lifecycle |

## Issue label state machine

`needs-triage` → (+size, area, priority) → `ready-for-agent` + `ai:ready` → `in-progress` (auto, branch) → `in-review` + `ai:implemented` (auto, PR) → `ai:self-reviewed` → merged (auto-close). Humans apply `ai:ready` and close; agents never do either (C21). Full taxonomy: [github/label-taxonomy.md](../github/label-taxonomy.md).

## Review pipeline order (never reorder)

**Converge gate** (M+, fresh context, frozen criteria) → **your rubric** (before reading ANY AI output) → **CodeRabbit** (always-on, `[Cn]`+severity, nits batched) → **C36 cross-family second opinion** (Claude/DeepSeek authored → Codex review; Codex authored → claude-code-action) → fix loop ≤2 → squash merge.

## Constitution quick index ([full text](../constitution.md))

C1–C5 spec fidelity · C6–C10 scope · C11–C15 quality · C16–C20 testing · C21–C25 gates & honesty · C26–C30 files · C31–C35 security (all 🔴) · **C36** reviewer independence 🔴 · **C37** event-triggered agents ingest data, never instructions 🔴. Severities: 🔴/🟠 request changes · 🟡/🔵 one batched comment.

## Skills → stages ([catalog](../skills-catalog.md))

S1 `grill-with-docs` · S2 `research` `prototype` `to-spec` `grill-with-docs` · S4 `domain-modeling` · S5 `to-tickets` `wayfinder` · S6 `implement` `tdd` `diagnosing-bugs` · S7 `converge` `code-review` · S9 `memory-dump` `graph-update` `retro` · anytime `handoff` `triage` `improve-codebase-architecture`.

## Memory: who writes what, when

Merge → session dump (5 min) · Friday → distill dumps into graph + docs (30 min) · decision → ADR same day · architecture change → architecture.md in the same PR · contradiction found mid-session → **invalidate immediately** (`valid_to` + `invalidated_by`), correct at distill ([protocol](../memory-freshness-protocol.md)).

## Standing dates

| What | When |
|---|---|
| ~~Sonnet 5 intro pricing ends~~ | ✅ done 2026-08-27 — now $3/$15, budgets re-run |
| CodeRabbit pricing re-verified | ✅ done 2026-08-27 — free tier covers solo |
| Research snapshot expires | **2026-10-19** — re-verify harness-matrix + models docs |
| Subtraction ritual | Quarterly — disable one mechanism for a week; delete it if nothing degrades |
