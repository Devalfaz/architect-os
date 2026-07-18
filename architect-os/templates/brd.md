# BRD Template — Business Requirements Document

*Decide whether this is worth building, in business terms, before any product thinking. For personal projects, a half page is sufficient. For SaaS products, invest more time. The non-goals section is what keeps downstream tickets from sprawling.*

---

```markdown
# BRD: [FEATURE / PRODUCT NAME]

<!-- status: draft | approved | rejected -->
<!-- date: YYYY-MM-DD -->
<!-- author: [name] -->

## Problem statement

[One paragraph: what problem exists, who has it, what evidence supports it. Written so someone unfamiliar with the project can understand it.]

## Target user / persona

[Who has this problem? Be specific — role, context, current workflow. "Compliance-conscious admins at companies with >50 employees" — not "users."]

## Desired outcome

[What changes in the user's life after this exists? Measurable if possible. "Admins can prove data portability compliance in under 2 minutes."]

## Success metric

[How will we know this worked? "Exports per week >20 within the first month" — not "users like it."]

## Non-goals

[What are we explicitly NOT building? This is the most important section for scope control. "No scheduled exports. No XLSX format. No async export jobs in v1."]

## Constraints

[Any hard constraints? "Must work with the existing auth system. Must not require new infrastructure. Data must never leave the EU region."]

## Go / no-go

- [ ] Go
- [ ] No-go
- [ ] Needs more research before decision

**Decision rationale:** [Why, in one sentence.]
```

---

*Save as `docs/product/<slug>/brd.md`. Grill with `/grill-with-docs` to pressure-test the non-goals and success metric. The gate: you can state problem, user, outcome, and non-goals in five sentences without reading the doc.*
