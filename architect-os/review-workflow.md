# Review Workflow — The Full Two-Stage Pipeline

*From PR open to squash merge. Human first, AI second, fix loop bounded at two rounds.*

---

## The pipeline

```
PR opened (CI green, self-review posted)
     │
     ▼
STAGE 1: Human review (10-minute rubric)
     │
     ├── Bounce ──→ Re-plan (S5) or re-spec (S2)
     │
     ├── Fix request ──→ Agent addresses → STAGE 1 again (max 2 rounds)
     │
     ▼
STAGE 2: AI review (automatic, parallel)
     │
     ├── CodeRabbit (always-on)
     └── Second AI opinion (Codex review) — optional
     │
     ▼
Agent addresses AI findings → Human resolves threads
     │
     ▼
Approved → Squash merge
```

---

## Stage 1: Human review

**Timing:** Review within 24 hours (same day for active tickets).

**What you use:** The [10-minute rubric](pr-review-rubric.md).

**What you do NOT do:** Read CodeRabbit's review first. Read agent's self-review before forming your own impression. This prevents anchoring.

**After your review:**
1. Post review comment in rubric format.
2. Read agent's self-review comment. Did it flag anything you missed? (Yellow flag if it caught something you didn't see.)
3. Read CodeRabbit's review.
4. Consolidate into single fix-request action set for agent.

---

## Stage 2: AI review

### CodeRabbit (primary, always-on)

Reviews every PR automatically. Best at: mechanical bugs, security patterns, code quality issues, missing error handling, performance anti-patterns. Bad at: design drift, architectural consistency, correct abstraction questions, business logic correctness.

**Interaction:** Valid findings → include in fix-request. False positives → reply `@coderabbitai false positive: [reason]`.

### Second AI opinion (optional)

Use on: security-sensitive paths, performance-critical hot paths, M-size with complex logic, agent-flagged uncertainty, weekly spot-check.

Run: `codex review --pr <NUM> --repo <owner/repo>` or claude-code-action.

**Decision rule:** Run once a week on one M-size PR. Track unique catches. After a month: is it worth $0.50–$2 per review?

---

## The fix loop (bounded at 2 rounds)

### Round 1
You post consolidated fix request from: your rubric, CodeRabbit, second AI. Agent addresses every finding with commit SHA or reasoned pushback. **You resolve threads** — agent never resolves threads.

### Round 2
Smaller fixes. New issues emerging in round 2 → something wrong.

### Bounce conditions (C24)
- Round 3 needed
- New issues emerge in round 2 unrelated to original fix requests
- Constitution violation not caught in round 1
- Agent shows confusion or hedging
- Diff keeps growing (scope creep during fixes)

**On bounce:** Close PR. Back to S5. Re-decompose. Old PR closed, not merged.

---

## Thread resolution protocol

- **You** resolve threads. Always.
- CodeRabbit resolves own threads on your confirmation reply.
- Agent never resolves. Addresses with commit or pushback; you assess and resolve.

---

## Approval and merge

### Criteria
- [ ] Your rubric posted
- [ ] CodeRabbit findings addressed
- [ ] Second AI opinion addressed (if run)
- [ ] All threads resolved by you
- [ ] CI green
- [ ] PR template complete
- [ ] Commit history clean

### Merge
Squash merge only. PR title = conventional commit. Delete branch after merge.

---

## Review metrics (track in rituals)

Per-PR: time to first review, fix rounds, CodeRabbit valid/FP ratio, second AI unique catches, bounce rate.
Monthly: avg review time, fix round distribution, CodeRabbit FP rate, second AI value.

---

## Anti-patterns

| Anti-pattern | Fix |
|---|---|
| Reading AI review first | Always rubric first |
| "LGTM" reviews | Be explicit about what you skipped and why |
| Fix loop death spiral (4+ rounds) | Bounce at 2 rounds |
| Fixing agent's code yourself | Fix via agent or file new ticket |
| Approving because "tests pass" | Trust the rubric, not green checkmarks |
| Letting review pile up >24h | Review same-day or next-morning |
