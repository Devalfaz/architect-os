# Failure Recovery Playbook — Symptom → Action

*When an agent goes off track, a PR goes sideways, or the workflow itself breaks. This is the field manual. Every symptom maps to a specific action, not a vague "try again."*

---

## Agent-level failures

### Symptom: Agent is stuck in a fix-loop (3+ test-fix cycles)

**Recognition:** The session transcript shows: write code → test fails → fix → test fails → fix → test fails. Repeating.

**Immediate action:** Kill the session.

**Root cause:** The approach is fundamentally wrong. Each fix is a local patch on a broken foundation.

**Recovery:**
1. Read the session transcript. What was the first test failure?
2. Check the ticket plan — was the initial approach correct, or did the agent misunderstand the requirement?
3. If the plan is correct but the approach failed: re-launch with a different strategy. "Try X approach instead of Y approach, because Y failed due to Z."
4. If the plan was wrong: bounce the ticket back to S5. Re-decompose.
5. Record in the ticket: "Agent failed 3 fix cycles on approach Y. Re-planned with approach X." This feeds the learning loop.

---

### Symptom: Agent edits files not in the plan

**Recognition:** The PR diff includes changes to files not in the ticket's plan list.

**Immediate action:** Kill the session. Do not continue — more unplanned edits will follow.

**Root cause:** C6/C7 violation. The agent expanded scope.

**Recovery:**
1. Revert all changes outside the plan.
2. Check: did the unplanned change fix a legitimate dependency? If yes, the ticket plan was incomplete — amend the plan, re-launch.
3. If no: the agent was "helpfully improving." File a separate tech-debt ticket for whatever it was trying to improve.
4. This is a constitution violation (C6). Flag it. If it happens frequently, your AGENTS.md needs stronger scope discipline language.

---

### Symptom: Agent adds a new dependency

**Recognition:** `package.json` diff shows a new package. No ADR line-item for it.

**Immediate action:** Revert the dependency add. Kill the session if the dependency was central to the approach.

**Root cause:** C8 violation. The agent chose a library without the gate.

**Recovery:**
1. Is the dependency justified? If yes, file an ADR line-item and amend the ticket plan. Re-launch with the ADR reference.
2. If not: the agent hallucinated a library or chose one unnecessarily. Re-launch with explicit instruction: "Do not add any new dependencies."
3. Record in gotchas: "Agent tried to add [library] for [purpose]. Better approach: [alternative]."

---

### Symptom: Agent outputs rapidly degrading quality

**Recognition:** The first 10–20 file edits are crisp. Then edits become sloppy, repetitive, or contradictory. The agent "forgets" what it already implemented.

**Immediate action:** Kill the session.

**Root cause:** Context window saturation. The session is too long.

**Recovery:**
1. Save current state (commit as WIP if needed).
2. Break remaining work into a subticket if scope was too large.
3. Launch a fresh session with remaining work. Use `/handoff` to compact context.
4. If this happens frequently: your tickets are too large. Downsize.

---

### Symptom: Agent hallucinates APIs or patterns

**Recognition:** Code uses a method that doesn't exist, imports a library not in `package.json`, or follows a pattern not in the codebase.

**Immediate action:** Stop the session. Point out the hallucination.

**Root cause:** The agent pattern-matched on training data, not your actual codebase.

**Recovery:**
1. If the API doesn't exist: verify the correct API in `docs/agents/verified-apis.md`. Have the agent use it.
2. If the pattern is wrong: point to the correct pattern in the codebase (a specific file) or AGENTS.md.
3. Record the hallucination in the memory dump: "Agent hallucinated [API/pattern]. Correct is [actual]."
4. If this happens frequently: your AGENTS.md or verified-apis.md needs updating.

---

## PR/review-level failures

### Symptom: PR is >400 lines

**Recognition:** The diff stat shows 600+ lines.

**Immediate action:** Do not review. Close PR with comment: "Size violation (C9). Split into ≤400-line tickets."

**Recovery:**
1. Re-decompose at S5. Split into 2–3 tickets.
2. Re-implement each with a fresh agent session.
3. If this happens frequently: your decomposition bar is too low. Raise it — smaller tickets.

---

### Symptom: PR touches files outside plan, but the changes are good

**Recognition:** Agent improved adjacent code. Changes are correct and valuable. But out of scope.

**Immediate action:** Bounce the PR. Do not merge, even if the changes are good.

**Root cause:** C6 violation, even if well-intentioned.

**Recovery:**
1. Extract out-of-scope changes into a separate ticket, gated at S5.
2. "But it was good!" is not a defense. Scope discipline is the system's safety property. Exceptions erode it.

---

### Symptom: 3 rounds of fixes and still not right

**Recognition:** Round 1: fixed X. Round 2: fixed Y emerged. Round 3: fixing Y broke X again.

**Immediate action:** Bounce. C24: 2 rounds max.

**Recovery:**
1. Close the PR. Do not merge.
2. Go back to the spec (S2) or plan (S5). What was underspecified?
3. Amend the spec. Re-decompose if needed.
4. File a spec delta documenting what was discovered.
5. This is not a failure of the agent — it's the discovery loop working. The fix-loop bound prevents it from becoming expensive.

---

### Symptom: CodeRabbit false positive rate >50%

**Recovery:**
1. Tune `.coderabbit.yaml` — more specific `high_level_summary`, expanded `path_filters`, `knowledge_base.learnings` entries.
2. If FPR remains high: consider whether CodeRabbit is the right tool for this codebase language/pattern.

---

## Workflow-level failures

### Symptom: Spec deltas are rising

**Recognition:** Memory dumps show 3+ spec deltas per feature, up from 1–2.

**Recovery:**
1. Review recent FSDs. Are edge-case tables filled? AC mechanically testable? External APIs verified?
2. Tighten the S2 gate: spend more time there.
3. Consider running `grill-with-docs` longer before signing off on the spec.

### Symptom: Cost per ticket rising without quality improvement

**Recovery:**
1. Check per-ticket costs. Which tickets are the cost drivers?
2. Route those tickets to a cheaper model next time.
3. If fix rounds are increasing: fix spec quality, not routing.

### Symptom: Reviewing 5+ PRs per day, quality dropping

**Recovery:**
1. Reduce WIP limit from 2 to 1 concurrent agent run.
2. Delay launching new agents until current PRs are reviewed.
3. Consider: are XS tickets worth the review overhead? Batch them or skip AI for trivial changes.

### Symptom: Graph >20% stale

**Recovery:**
1. Block 30 minutes on calendar for weekly distill. Treat as a hard meeting.
2. If 30 min isn't enough: the graph is too large. Prune low-value nodes.
3. Automate more via `/graph-update` skill.

### Symptom: You haven't written code yourself in 2+ weeks

**Recovery:**
1. Pick one ticket each sprint to implement yourself.
2. If you can't implement it: your understanding of the codebase has decayed. Re-learn.
3. Architect-orchestrator, not absentee-landlord. You must remain capable of writing what you approve.

---

## Emergency procedures

### Broken main
1. Revert squash commit immediately. `git revert <SHA>` on main, push.
2. Diagnose: what test would have caught this? Why didn't it?
3. Fix the test gap, then re-implement with the test in place.
4. Retro: how did this pass review? Was the rubric skipped?

### Security incident (secrets leaked, injection found, etc.)
1. Revoke the secret immediately (rotate keys, invalidate tokens).
2. Revert the commit if the code is the vulnerability.
3. Audit: what other code follows the same vulnerable pattern? File tickets.
4. Add a CI check to catch this class of vulnerability.

### Agent produces malicious or dangerous code
*(Extremely rare — possible with prompt injection or training data poisoning)*
1. Kill the session immediately.
2. Do not run or merge any code from that session.
3. Report to the platform with the session transcript.
4. Audit: was there untrusted input in the context?

---

## The meta-recovery

### Symptom: The workflow itself feels broken

**Recognition:** Spending more time managing the workflow than coding. Rituals feel like chores. Metrics going wrong direction. Questioning whether this is worth it.

**Action:**
1. Drop to the lightweight profile for one sprint. Skip everything except: one ticket = one PR, file-level plan in the issue, human diff review, tests before merge.
2. After the sprint, evaluate: do you miss the ceremony? If no, stay lightweight.
3. If burnout: scale back. The OS should make you more effective, not more tired.
4. Monthly retro is the forum: "The workflow itself is the problem."

---

*This playbook exists because things will go wrong. AI agents are powerful but unpredictable. The system is robust but requires maintenance. When something breaks, don't improvise — follow the playbook. If you encounter a symptom that isn't here, add it after recovery.*
