# Code Review Checklist — Agent Self-Review

*Every agent PR must include this checklist in the self-review comment. Human reviewers cross-reference against it during the rubric.*

---

## Before requesting human review, verify:

### ✅ Spec Fidelity (C1–C5)
- [ ] C1: All code implements what the FSD specifies — no silent divergence
- [ ] C2: Read the linked FSD section before writing code
- [ ] C3: Every acceptance criterion is mechanically verified (tests pass, behavior matches Given/When/Then)
- [ ] C4: Tests written first (TDD), all green
- [ ] C5: Every edge case from the FSD's edge-case table is handled

### ✅ Scope Discipline (C6–C10)
- [ ] C6: Every changed file is in the ticket's implementation plan — no extras
- [ ] C7: No unauthorized refactoring of adjacent code
- [ ] C8: No new dependencies without an ADR line-item (check `package.json` diff)
- [ ] C9: PR is ≤400 lines net new (check diff stat)
- [ ] C10: One ticket = one branch = one PR

### ✅ Code Quality (C11–C15)
- [ ] C11: Types/interfaces/schemas defined before logic
- [ ] C12: All external input validated at system boundary (Zod/Pydantic/etc.)
- [ ] C13: Errors handled as values — no uncaught throwables
- [ ] C14: Error logs include: what was attempted, input, actual error, where
- [ ] C15: No unexplained magic numbers in logic

### ✅ Testing (C16–C20)
- [ ] C16: Tests assert behavior (Given/When/Then), not implementation details
- [ ] C17: One concept per test — no multi-concern mega-tests
- [ ] C18: Tests are independent — no shared mutable state, no ordering dependencies
- [ ] C19: Each test fails for exactly one reason
- [ ] C20: No flaky tests — all pass consistently

### ✅ Communication (C21–C25)
- [ ] C21: Agent has not merged, closed, or deployed — human gates remain
- [ ] C22: This self-review is posted as a PR comment (see format below)
- [ ] C23: Any FSD discoveries documented as spec deltas, not silently handled
- [ ] C24: Fix rounds count: [0 / 1 / 2] — if 2, re-evaluate plan before requesting review
- [ ] C25: Areas of uncertainty explicitly flagged below

### ✅ Files & Structure (C26–C30)
- [ ] C26: File conventions followed (per AGENTS.md)
- [ ] C27: No dead code — unused imports, functions, variables removed
- [ ] C28: File names match primary exports
- [ ] C29: Imports organized: external → internal → relative
- [ ] C30: No hand-edits to generated files

### ✅ Security (C31–C35)
- [ ] C31: No secrets in code — only environment variables
- [ ] C32: Input sanitized before reaching database or filesystem
- [ ] C33: AuthN and AuthZ are separate checks
- [ ] C34: Dependencies pinned — no `latest` or `*` versions
- [ ] C35: No PII, credentials, tokens, or full identifiers in logs

### ✅ Review setup (C36 — human's responsibility, agent states the facts)
- [ ] C36: Authoring model family declared in the PR template, so the human can route a cross-family second reviewer (Claude authored → Codex review; Codex authored → claude-code-action/CodeRabbit)

---

## Areas of uncertainty

*Flag anything you're unsure about. "I did X because Y." Better to surface uncertainty than hide it.*

- [list or "None — all decisions were straightforward"]

---

## Deviations from the implementation plan

*Did you change any files not in the plan? Add any patterns not specified?*

- [list or "None — all files match the plan"]

---

## Model and cost

| Field | Value |
|---|---|
| Model used | [e.g., Claude Sonnet 5] |
| Session duration | [e.g., ~8 minutes] |
| API calls | [e.g., ~12 tool calls] |
| Fix cycles | [e.g., 1 (test → fix → green)] |

---

## Self-review comment template

*Paste this into the PR as a comment:*

```markdown
## 🤖 Agent Self-Review

### Files changed (with rationale)
- `path/to/file.ts` (new/modified): [What it does and why]

### Constitution compliance
- C1 ✓ — follows FSD §[section]
- C4 ✓ — TDD: tests written first, all green
- C6 ✓ — only planned files touched
- C7 ✓ — no scope creep
- C8 ✓ — no new dependencies
- C9 ✓ — [N] lines
- C12 ✓ — [validation approach] on [boundary]
- C22 ✓ — this self-review

### Uncertainty flags
- [list or "None"]

### Deviations
- [list or "None"]

### Model: [name] | Fix cycles: [N]
```
