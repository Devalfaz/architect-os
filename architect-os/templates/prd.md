# PRD Template — Product Requirements Document

*Convert intent into numbered requirements. Each requirement is testable. The PRD is what you show stakeholders; the FSD is what agents implement from.*

---

```markdown
# PRD: [FEATURE NAME]

<!-- status: draft | in-review | approved -->
<!-- date: YYYY-MM-DD -->
<!-- linked-brd: ../brd.md -->
<!-- linked-fsd: ../../specs/<slug>/fsd.md -->

## Overview

[Two sentences: what are we building and why? Non-technical. A PM should understand this.]

## Functional requirements

Each requirement is numbered (FR-1, FR-2...) and independently testable.

| ID | Requirement | Priority | Notes |
|---|---|---|---|
| FR-1 | [Requirement — what the system must do, not how] | P0 | [Clarification, edge cases] |
| FR-2 | | P1 | |
| ... | | | |

## Non-functional requirements

| ID | Requirement | Target |
|---|---|---|
| NFR-1 | Performance: CSV export for 50k rows | ≤30 seconds |
| NFR-2 | Accessibility: all UI components | WCAG AA |
| ... | | |

## User stories

Format: "As a [role], I want [capability], so that [benefit]."

1. As an admin, I want to export my organization's data as CSV, so that I can prove compliance during audits.
2. ...

## Out of scope

[What are we explicitly NOT building in this version? Linked to the BRD's non-goals.]

## Dependencies

[What must exist before this can be built? Other features, infrastructure, third-party APIs, team availability.]

## Open questions

[Anything unresolved that blocks spec writing?]
```

---

*Save as `docs/product/<slug>/prd.md`. This is the input to the FSD. The PRD says what — the FSD says how.*
