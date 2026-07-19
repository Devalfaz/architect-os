# PR Review Rubric — The 10-Minute Human Review

*You review the diff first, strictly before reading AI reviews. This rubric finds what AI reviewers systematically miss.*

---

## Why human-first

**Do not open CodeRabbit, Codex review, or any AI reviewer's comments until your own pass is complete.** The studied risk isn't classic anchoring — it's *trust drift* (failure mode #15): "the AI found nothing" quietly becomes a reason to skim, and AI reviewers are structurally silent on exactly what this rubric checks — intent, end state, architecture fit. Your pass and the AI pass catch disjoint bug classes; reading theirs first collapses yours into theirs.

---

## Pre-read gates (~1 minute, before the clock starts)

Reject **without a full read** if any of these fail — bouncing at the gate costs 1 minute; reviewing an ungated PR costs 30.

| Gate | Check | On failure |
|---|---|---|
| **Linked ticket** | PR has `Closes #N` and the ticket has a plan | Bounce — no ticket, no review |
| **Size** | ≤400 lines net (C9) unless pre-agreed | Bounce — `size:split-me` |
| **CI green** | Typecheck, lint, test, build all pass | Bounce — the diff is not evidence the code runs (hallucinated imports pass diff review; only execution catches them) |
| **Self-review posted** | C22 comment exists | Bounce — request it first |
| **Converge PASS** (M-size) | The [converge gate](skills/converge/SKILL.md) verdict is PASS — read the verdict line only, not the detail table | FAIL already auto-bounced; no verdict = run it first |
| **Reviewer independence (C36)** | The AI second-opinion reviewer is a different model family than the authoring agent | Fix the reviewer config before reviewing; if no cross-family reviewer is available, escalate this pass to 20 minutes |
| **Collateral-damage scan** | Every file in the diff's file list is justified by the ticket's plan | Unexplained files = bounce (failure mode #14 — agents touch what they traverse) |

---

## The 10-minute pass

### Minutes 1–2: End-state check (primary)

**Does the diff achieve the spec's stated end state?** Read the ticket's acceptance criteria, then ask of the diff: if I deployed this, would every Given/When/Then hold? List gaps in writing. This is the primary check — everything below is secondary, and no AI reviewer performs it. Your written output doubles as the spec-coverage confirmation:

> "The diff addresses [every criterion | all but X, deferred because Y]."

- ACs map to test cases?
- Edge cases handled? Check 2–3 from the table.
- Anything implemented that the spec didn't ask for?

### Minutes 3–4: Constitution scan
- C6: scope creep? Files not in plan?
- C7: unauthorized refactoring — including "harmless" adjacent edits?
- C8: new deps without ADR?
- C12: boundary validation?
- C31: secrets in code?

Constitution violations are **bounces**, not fix-requests. Cite the rule ID; the rule's severity tag (🔴/🟠/🟡/🔵) sets the response — any 🔴/🟠 = request changes.

### Minutes 5–6: Architecture & design
- Respects architecture boundaries?
- New abstractions justified?
- Makes repo easier or harder to understand?
- Follows existing patterns?

### Minutes 7–8: Error handling & edge cases
- What happens when this fails?
- Empty/null/undefined input?
- Happy path too happy?
- Async: awaited? caught? timeout?

### Minute 9: Security scan
- SQL injection? Parameterized?
- AuthZ checked per-operation?
- Input sanitized?
- Sensitive data in logs?

### Minute 10: Tests
- Tests exist? (Non-negotiable, C4)
- Test behavior or implementation?
- Would they fail if the feature broke? (Weak tests are green too — failure mode #4)
- Independent?

---

## What NOT to flag (leave to AI or CI)

Style/formatting (Prettier/ESLint), variable naming preferences, "extract to function" for ≤10 lines, optimizations on non-hot-paths, test file naming conventions, semicolons/whitespace/import ordering (all CI-enforced).

And when *you* do flag minor issues: batch every 🟡/🔵 finding into one comment, never individual threads. The nit-flood rule (failure mode #16) applies to humans too.

---

## Decision

### Approve
All pre-read gates passed, end state achieved, constitution clean, architecture consistent, error handling reasonable, tests present and behavioral.

### Fix-request (≤2 rounds, C24)
Specific, actionable, each citing a rule ID with its severity. Agent addresses with commit SHA or reasoned pushback. You resolve threads.

### Bounce (re-plan required)
Any pre-read gate failure, scope creep (C6/C7), architecture violation, missing tests (C4), 🔴 violation, 2+ fix rounds → back to S5.

---

## Review comment format

```markdown
## Review — [Approve / Fix Request / Bounce]

### Pre-read gates: [all passed / bounced at: gate]
### End state (2 min): [achieved / gaps: …]
### Constitution (2 min)
### Architecture (2 min)
### Error handling (2 min)
### Security (1 min)
### Tests (1 min)

### Verdict: [decision with notes — cite rule IDs with severities]
```

---

*This rubric is runnable in 10 minutes because the real quality work happened upstream in S2 (spec) and S5 (plan). If you're finding problems in review that take longer than 10 minutes, the problem isn't the review — it's the spec.*
