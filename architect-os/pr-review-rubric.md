# PR Review Rubric — The 10-Minute Human Review

*You review the diff first, strictly before reading AI reviews. This rubric finds what AI reviewers systematically miss.*

---

## Why human-first

Reading AI reviews first anchors you on mechanical issues and blinds you to design drift, over-engineering, and missed requirements. Human-first review prevents approving PRs that are mechanically sound but architecturally wrong.

---

## The 10-minute pass

### Minutes 1–2: Constitution scan
- C6: scope creep? Files not in plan?
- C7: unauthorized refactoring?
- C8: new deps without ADR?
- C9: PR >400 lines?
- C12: boundary validation?
- C31: secrets in code?

Constitution violations are **bounces**, not fix-requests.

### Minutes 3–4: Spec conformance
- Changed files match implementation plan?
- ACs map to test cases?
- Edge cases handled? Check 2–3 from the table.
- Anything out of scope implemented?

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
- One concept per test?
- Independent?

---

## What NOT to flag (leave to AI)

Style/formatting (Prettier/ESLint), variable naming preferences, "extract to function" for ≤10 lines, optimizations on non-hot-paths, test file naming conventions, semicolons/whitespace/import ordering (all CI-enforced)

---

## Decision

### Approve
All constitution passes, spec conformance yes, architecture consistent, error handling reasonable, tests present and behavioral

### Fix-request (≤2 rounds, C24)
Specific, actionable. Agent addresses with commit SHA or reasoned pushback. You resolve threads.

### Bounce (re-plan required)
Scope creep (C6/C7), architecture violation, missing tests (C4), constitution violation, 2+ fix rounds → back to S5

---

## Review comment format

```markdown
## Review — [Approve / Fix Request / Bounce]

### Constitution (1 min)
### Spec conformance (2 min)
### Architecture (2 min)
### Error handling (2 min)
### Security (1 min)
### Tests (2 min)

### Verdict: [decision with notes]
```

---

*This rubric is runnable in 10 minutes because the real quality work happened upstream in S2 (spec) and S5 (plan). If you're finding problems in review that take longer than 10 minutes, the problem isn't the review — it's the spec.*
