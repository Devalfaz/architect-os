# Memory Freshness Protocol

*Stale memory is worse than no memory because it's confidently wrong.*

---

## The core rule

Every document, graph node, and research note carries `last_verified`. Unverified memory is a hypothesis, not a fact.

---

## Write-time invalidation (automatic — the important half)

The weekly distill is the *confirmation* step, not the *discovery* step. Waiting for Friday to flag a known-wrong fact means every agent session until Friday reads a lie. So:

**The rule: any agent that discovers a memory contradicts the current code invalidates it in the same session.** Discovery moments: a graph node names a file that doesn't exist, an AGENTS.md claim fails against `package.json`, an ADR's compliance grep comes back dirty, a "verified API" throws at typecheck.

What "invalidate immediately" means, by artifact:

| Artifact | Immediate action (same session) | Weekly distill then does |
|---|---|---|
| Graph node/edge | Set `valid_to: <today>` + `invalidated_by: <what proved it wrong>` — the fact becomes history, never deleted (schema v1.1) | Confirms the invalidation, adds the corrected replacement node |
| AGENTS.md claim | Inline `⚠️ STALE (found wrong 2026-07-20: <one line>)` next to the claim | Rewrites the claim, clears the flag |
| ADR | Comment on the ADR issue/file: compliance check failing | Human decides: enforce or supersede |
| verified-apis.md | Strike the entry with the failing evidence | Re-verify against installed version |

Two constraints keep this safe:

- **Invalidation ≠ correction.** The agent marks the fact wrong with evidence; it does not write the replacement truth mid-ticket (that's scope creep, C6/C7, and mid-session "corrections" are how memory poisoning happens). The dump's graph-delta block carries the proposed replacement; the distill reviews it.
- **Invalidated facts stay as history.** `valid_to` + `invalidated_by` preserve provenance — you can always ask "what did we believe on date X, and what changed our mind." Deletion says nothing; tombstones teach.

---

## Verification schedule

| Artifact | Frequency | Trigger |
|---|---|---|
| AGENTS.md | Weekly | Weekly distill |
| architecture.md | Monthly or on architecture change | ADR merged, significant refactor |
| ADRs | Monthly or when constrained code changes | File in constrained area modified |
| Research | On expiry date | Expiry date passed |
| verified-apis.md | Weekly or when deps change | package.json/lockfile diff |
| project-gotchas.md | Weekly | Weekly distill |
| Graph nodes | Weekly | Weekly distill |
| Graph edges | Weekly | Weekly distill |
| Memory dumps | Never (raw material, timestamped) | N/A |

---

## How to verify

### AGENTS.md
Check every claim. File conventions vs actual directory. Gotchas: new from dumps? Old ones fixed? Tech versions vs package.json. Update `last_verified`.

### Architecture docs
Walk the overview. New modules undocumented? ADR statuses correct? Mermaid diagrams match code? Update `last_verified`.

### ADRs
Compliance check still passing? Code in constrained area still follows decision? Mark deprecated ADRs with `superseded_by` pointer — never delete.

### Research
Sources still accessible? Code samples still work? If valid: update dates. If stale: `⚠️ STALE` banner, move to archive if obsolete.

### Verified APIs
Each API claim checked against actual installed version docs. Test code snippets. Flag changed APIs.

### Knowledge graph
Nodes: still exist? Edges: still accurate? Confirm any write-time invalidations from the week (`valid_to` set mid-session) and add their corrected replacement nodes. Orphans and wrong facts get `valid_to` + `invalidated_by`, not deletion. Add new from dumps. Update `last_verified` on all checked.

---

## Staleness flags

```
⚠️ STALE — last_verified: 2025-01-01 (more than 30 days ago)
This claim may be inaccurate. Verify before relying on it.
```

Stale flags tell agents to be suspicious. Deleted memory says nothing.

---

## Weekly distill checklist (30 min)

- [ ] Read all week's memory dumps
- [ ] Identify graph deltas from each dump
- [ ] Update repo-graph.json (add/update nodes and edges, update last_verified)
- [ ] Verify AGENTS.md freshness
- [ ] Check research/ for expired — flag or re-verify
- [ ] Update project-gotchas.md from dump surprises and hallucinations
- [ ] Check verified-apis.md if dependencies changed
- [ ] Run /graph-update skill → graph delta PR
- [ ] Review and merge graph delta PR
- [ ] Archive stale dumps (>2 weeks) or keep if unique insights

---

*Memory staleness is the silent killer of AI-assisted development. The weekly distill is the vaccine. Don't skip it.*
