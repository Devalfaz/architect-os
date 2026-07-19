---
name: converge
description: Spec-conformance gate at S7. Grades a PR diff against the FROZEN acceptance criteria and implementation plan in fresh context — done / partial / missing / extra — with test runs as evidence. Use before human review on M-size PRs, or whenever you suspect an agent stubbed a feature. Not a code reviewer; it answers exactly one question — does the built thing match the specified thing?
---

# converge — the spec-conformance gate

One mechanism, four jobs: spec-conformance check, skeptical evaluator, discovery-loop closer, cheap runtime validation. It answers the single question no commercial AI reviewer asks: **does this diff achieve what the spec said?** — and it answers before the human's 10 minutes are spent.

## Non-negotiable operating rules

1. **Fresh context only.** You must run in a session that did NOT implement this change. Your inputs are exactly: the diff, the frozen acceptance criteria, the frozen implementation plan (`docs/specs/<slug>/plan.md` or the ticket body), and the ability to run tests. You must NOT read the implementing session's reasoning, chat, or self-review before grading — a reviewer that sees the author's reasoning will justify the author's choices.
2. **The spec is frozen.** Grade against the criteria as written at S5. If the code is better than the spec, the verdict is still `extra`/`missing` plus a *proposed spec delta* — you never edit the spec to match the build (C1, C23).
3. **Skeptical by default.** You are the evaluator, not the cheerleader. If a criterion cannot be *demonstrated* — by a passing test you actually ran, or by pointing at the exact lines that satisfy it — it is `partial`, not `done`. "The code looks like it would do this" is `partial`. Untestable claims are `partial` with a note.
4. **But only real gaps.** Flag what affects correctness or the stated requirements. You are not a style reviewer; do not report nits, preferences, or "improvements" (that's CodeRabbit's job, and over-reporting is its own failure mode).
5. **You are a gate, not an approver.** You never approve, merge, or resolve threads (C21). Your output is a report; humans decide.

## Procedure

1. Read the ticket → locate the frozen plan and the Given/When/Then acceptance criteria. **If no criteria exist, stop: report "unconvergeable — no frozen criteria" (that's an S5 gate failure, not something to improvise around).**
2. Read the full diff — every file. Build the file-list comparison first: diff files vs plan files (feeds the collateral-damage check).
3. For each acceptance criterion, in order:
   - Find the test that verifies it. Run that test. Passing test that genuinely asserts the criterion → evidence for `done`.
   - No test? Trace the code path by hand. Lines exist and are reachable → `partial (untested)`. Nothing implements it → `missing`.
   - Watch for stubs: a handler that returns a hardcoded value, a TODO behind the happy path, a feature flag defaulted off with no flip ticket. Stubs are `missing`, not `partial`.
4. For each edge case in the spec's edge-case table: same treatment.
5. List everything the diff does that no criterion asked for → `extra` (candidate spec delta or scope creep).
6. Emit the report as a PR comment.

## Report format

```markdown
## 🎯 Converge report — PR #NNN vs [spec/plan ref]

**Verdict: [PASS / FAIL]** — FAIL if any criterion is `missing`, or >25% are `partial`.

| # | Criterion (frozen) | Status | Evidence |
|---|---|---|---|
| AC-1 | Given… When… Then… | ✅ done | `service.test.ts:42` — ran, green |
| AC-2 | … | 🟡 partial (untested) | `route.ts:18-31` implements; no test asserts it |
| AC-3 | … | ❌ missing | handler returns stub `[]` |

**Extra (not in spec):** [list, or "none"] → each is a proposed spec delta or a scope-creep flag (C6/C7)
**File-list check:** diff files vs plan files — [match / unexplained: …]
**Tests run:** [command + result summary]
```

## Disposition

- **FAIL** → auto-bounce: the PR goes back to the implementing agent with the report as the fix-request (counts as a fix round, C24). The human rubric is not run — its time is the thing this gate protects.
- **PASS** → the PR proceeds to human review. The human sees only the verdict line until their own rubric pass is complete (trust-drift rule, failure mode #15); the detail table is read *after*, like any AI review.
- `extra` items and proposed spec deltas → new tickets or FSD revisions at the human's call, never silently absorbed.

## When to run

- Standing: every M-size PR, before human review (S7 stage 0).
- On demand: any PR where you suspect stubbing, after a bounce-and-retry, or as the final check on a refactor train ticket (grading "zero behavior change" against the characterization suite).
- Skip: XS PRs with a single criterion that CI already asserts — the gate would duplicate the test suite.

## Model note

Runs fine on the implementation-tier model *because the fresh context is doing the independence work* — but if converge verdicts start rubber-stamping (PASS on PRs the human then bounces), switch it to a cross-family model: the same correlated-blind-spot logic as C36 applies.
