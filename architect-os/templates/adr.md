# ADR Template — Architecture Decision Record

*One per irreversible choice. Each ADR must include an agent instruction line and a mechanical compliance check. ADRs document decisions that are expensive to reverse — use of a library, choice of architecture style, data model decision.*

**When to write one:** *the moment you reject a credible alternative.* If nothing
real was rejected, it isn't a decision — it's a structural fact, and it belongs
in [architecture.md](architecture.md), not here. That test is why a healthy repo
has few ADRs and one long architecture doc, not the reverse.

**An ADR is frozen, not living.** Once accepted, you never edit it to match
reality — you write a new one that supersedes it. The old file stays as
provenance: *what did we believe on date X, and what changed our mind.* Same
model as the knowledge graph's `valid_to` / `invalidated_by`
([repo-memory.md](../repo-memory.md)) — `superseded_by` **is** invalidation, one
layer up. Editing an accepted ADR destroys the only record of the reasoning.

---

```markdown
# ADR-NNNN: [TITLE]

<!-- status: proposed | accepted | deprecated | superseded -->
<!-- date: YYYY-MM-DD -->
<!-- supersedes: ADR-NNNN (if applicable) -->
<!-- superseded_by: ADR-NNNN (if applicable) -->
<!-- last_verified: YYYY-MM-DD -->

## Context

[What problem does this solve? What forces are at play? What constraints exist?
Write so someone joining a year from now understands why this decision was made.]

## Decision

[What did we decide? Be specific. Not "we chose React" — "we chose React 19 with Server Components as the primary rendering strategy."]

## Alternatives considered

| Alternative | Pros | Cons | Why rejected |
|---|---|---|---|
| [Alternative 1] | | | |
| [Alternative 2] | | | |

## Consequences

### What becomes easier?

### What becomes harder?

### What risks does this introduce?

## Agent instruction

[One line that agents executing code must follow. "All new API routes MUST use Zod validation for request parsing."]

## Compliance check

[A mechanical way to verify this decision is being followed. "ESLint rule `require-zod-validation` fails if a route handler body isn't passed through `.parse()`" or "Architecture lint rule `no-cross-boundary-imports` prevents imports from `src/server/` into `src/app/`"]

## Migration plan (if applicable)

[If this decision requires changes to existing code, how will those changes be made? Incrementally? Big-bang? What's the strangler fig pattern?]
```

---

*Save as `docs/adr/NNNN-<slug>.md`. Number sequentially from 0001. ADR-0001 is always the tech stack. ADR-0002 is always the architecture style. Approved ADRs are merged to main and referenced in AGENTS.md. Deprecated ADRs carry a `superseded_by` pointer — never delete.*
