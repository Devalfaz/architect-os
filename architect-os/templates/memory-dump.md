# Memory Dump Template

*5 minutes at the end of every coding day. One file per day, all merged PRs in one file. This is the raw material that feeds the weekly distill and graph update.*

---

```markdown
# Memory Dump — YYYY-MM-DD

<!-- Daily dump. One file per day. Save to memory/dumps/YYYY-MM-DD.md -->

## What shipped today

### PR #[NNN]: [Title] — [merged / in review]

**What changed:**
- New: `path/to/file.ts` — [what it does]
- Modified: `path/to/file.ts` — [what changed]
- Deleted: `path/to/file.ts` — [why]

### PR #[NNN]: [Title] — [merged / in review]

...

## Decisions

- [Decision — with rationale if it differs from the spec]
- ...

## Surprises

*Things that didn't go as expected — for better or worse.*
- [Surprise 1: what was expected, what happened, what we did]
- ...

## Hallucinations caught

*APIs, patterns, or facts the agent got wrong that were caught before merge.*
- [Hallucination: what the agent claimed → what's actually correct]
- ...

## Graph delta candidates

*Nodes and edges that should be added/updated in the knowledge graph.*
- +Node: `type:id` — [label, description]
- +Edge: `source → target` (type: [relationship])
- ~Node: `id` — [what changed]
- -Edge: `source → target` — [why removed]

## Skill friction

*Workflow issues — skills that didn't work well, gaps, rough edges.*
- [Skill name]: [what went wrong or felt slow]
- None — everything worked smoothly.

## Notes

[Anything else worth remembering for the weekly distill or retro.]
```

---

*Save as `memory/dumps/YYYY-MM-DD.md`. One file per coding day. If no coding today, no dump. Dumps older than 2 weeks are archived during the weekly distill.*
