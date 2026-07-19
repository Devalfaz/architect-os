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
     └── Cross-family second opinion (C36) — standing on M/security PRs
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

**Interaction:** Valid findings → include in fix-request. False positives → reply `@coderabbitai false positive: [reason]`. Config: [.coderabbit.yaml](github/.coderabbit.yaml) — rules cited as `[Cn]` with severity, 🟡/🔵 batched, `docs/**`/`templates/**`/`memory/*.json` explicitly re-included (failure mode #17).

**Finishing Touches / autofix:** reviewer-generated fixes (autofix, docstrings, generated tests) are a **new unreviewed change** from the same vendor that just reviewed (failure mode #18). They go through the rubric like any commit — never merged on the reviewer's say-so.

### Second AI opinion — cross-family by construction (C36)

**The rule (C36, 🔴):** when both author and reviewer are AI, they must not share a model family. Same-family review counts as no review — the reviewer inherits the author's blind spots.

**Routing table:**

| PR authored by | Standing second opinion | Alternatives | Never |
|---|---|---|---|
| Claude Code (default) | **Codex review** — `codex review --pr <NUM> --repo <owner/repo>` | Copilot code review (if seat exists) | claude-code-action on a Claude model |
| Codex | claude-code-action (Anthropic model) | CodeRabbit alone + 20-min rubric | Codex review |
| Cursor agent | Codex review or claude-code-action | — | Cursor Bugbot |
| Human (no agent) | any — C36 doesn't apply | — | — |

**When it runs:** standing on every M-size PR and anything touching `area:auth` / `area:security` paths; on-demand elsewhere (agent-flagged uncertainty, performance-critical hot paths, weekly spot-check). Whenever it runs, the family rule is hard. If no cross-family reviewer is available, the rule is not waived — the human rubric escalates to the 20-minute pass.

**Track it:** unique catches per reviewer per month. If the second opinion catches nothing CodeRabbit + rubric didn't for a full month, drop it to weekly spot-checks — but never drop the family rule.

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
Monthly: avg review time, fix round distribution, CodeRabbit FP rate, second AI value, **address rate** (AI review comments addressed ÷ emitted — below ~35% for two consecutive weeks means prune rules and re-tune the reviewer, not push harder; see failure mode #16).

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
