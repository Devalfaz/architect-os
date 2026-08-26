# FSD Template — Functional Specification Document

*The implementation contract. Write for a competent contractor with zero context. If a requirement needs the conversation to be understood, it isn't written yet. This is the document agents read during implementation — it is the most important artifact in the system.*

---

```markdown
# FSD: [FEATURE NAME]

<!-- status: draft | in-review | approved -->
<!-- date: YYYY-MM-DD -->
<!-- linked-prd: ../../product/<slug>/prd.md -->
<!-- generated-by: /to-spec or manually -->

## Scope

[What this spec covers. Which FR-ids from the PRD. Linked BRD for business context.]

## User flows

[Step-by-step for each user journey. Use Mermaid or numbered lists.]

### Flow 1: [Name]

1. User navigates to Settings > Export.
2. User clicks "Export CSV."
3. System generates CSV of all user-owned records.
4. Browser downloads the file.

## States & screens (for UI features)

| Screen / State | Description | Components |
|---|---|---|
| Export button (idle) | Button on settings page, grey when no data | `<Button>` from shadcn |
| Export button (loading) | Spinner, disabled | `<Button loading>` |
| Export error | Toast with error message | `<Toast variant="destructive">` |
| Empty state | "No data to export" message | `<EmptyState>` |

## API contracts

### `GET /api/export.csv`

**Query params:**
| Param | Type | Required | Description |
|---|---|---|---|
| `format` | `"csv"` | No | Default: csv |

**Response (200):**
```
Content-Type: text/csv
Content-Disposition: attachment; filename="export-{date}.csv"

column1,column2,column3
value1,value2,value3
...
```

**Response (401):** `{ "error": "Unauthorized" }`

**Response (413):** `{ "error": "Export exceeds 100,000 row limit" }`

**Response (500):** `{ "error": "Export generation failed" }`

## Data model changes

[What database changes are needed? New tables, columns, indexes, migrations.]

## Requirements (EARS)

*Numbered FR-ids in EARS notation. Write these **before** the edge-case table —
walking all five patterns is what generates the edge cases, instead of hoping
you remember them.*

| Pattern | Keyword | Template | What it surfaces |
|---|---|---|---|
| Ubiquitous | *(none)* | The `<system>` shall `<response>` | always-true invariants |
| State-driven | **While** | While `<precondition>`, the `<system>` shall `<response>` | mode and state behaviour |
| Event-driven | **When** | When `<trigger>`, the `<system>` shall `<response>` | the happy path |
| Optional feature | **Where** | Where `<feature is included>`, the `<system>` shall `<response>` | flags, tiers, platforms |
| **Unwanted behaviour** | **If / Then** | If `<trigger>`, then the `<system>` shall `<response>` | **the edge cases** |
| Complex | *(combined)* | While `<precondition>`, when `<trigger>`, the `<system>` shall `<response>` | conditional behaviour |

**FR-1** — The export service shall produce RFC-4180-conformant, UTF-8 CSV. *(ubiquitous)*

**FR-2** — When the user selects "Export CSV", the system shall generate a CSV of all records the user owns. *(event-driven)*

**FR-3** — While an export is generating, the system shall display the export button disabled with a spinner. *(state-driven)*

**FR-4** — If the user owns more than 100,000 records, then the system shall return 413 with "Export exceeds 100,000 row limit". *(unwanted)*

**FR-5** — If the database connection fails during generation, then the system shall return 500 and log the error with user id and row count. *(unwanted)*

**FR-6** — If the request carries no valid session, then the system shall return 401. *(unwanted)*

**FR-7** — Where object-storage offload is enabled, the system shall stream to a signed URL instead of an inline response. *(optional feature)*

> **The discipline:** walk all five patterns for every flow. **A flow with no
> If/Then requirements has not been thought about yet** — that is the same
> failure the edge-case table's "empty = not thought about" rule catches, but
> caught earlier and systematically rather than by memory.

**How the three artifacts chain:** EARS requirement → edge-case row → Given/When/Then criterion. EARS states *what must be true*; G/W/T states *how you would test it*. Every **If/Then** requirement should appear as an edge-case row **and** at least one acceptance criterion — if it doesn't, one of the three is incomplete.

## Edge-case table

*Every cell must be filled. Empty = not thought about ≠ none exist. Every
If/Then requirement above should have a row here.*

| Scenario | Expected behavior | Handled by |
|---|---|---|
| User has zero records | Return empty CSV with headers only | Service: `records.length === 0` → headers-only |
| User has >100k records (v1 cap) | Return 413 error with message | Service: pre-count query, early return |
| Concurrent export requests | Each request generates independently; second request gets fresh data | Stateless GET, no locking needed |
| Unicode characters in records | BOM-prefixed CSV, UTF-8 encoded | `serializeRow` util |
| Special CSV characters (commas, quotes) | Properly escaped per RFC 4180 | CSV library |
| User not authenticated | Return 401 | Auth middleware |
| Database connection fails mid-export | Return 500, log error with context | try/catch in route handler |

## Acceptance criteria (Given/When/Then)

*Keyed to FR-ids. Each criterion is mechanically testable.*

**FR-1: Export button**

> GIVEN a logged-in user on the Settings page
> WHEN the page loads
> THEN an "Export CSV" button is visible

> GIVEN a logged-in user with zero records
> WHEN they click "Export CSV"
> THEN a CSV file downloads containing only column headers

**FR-2: Export all user-owned records**

> GIVEN a logged-in user with 50 records
> WHEN they click "Export CSV"
> THEN a CSV file downloads containing all 50 records within 5 seconds

**FR-3: Row limit enforcement**

> GIVEN a logged-in user with 150k records
> WHEN they click "Export CSV"
> THEN a 413 error is displayed: "Export exceeds 100,000 row limit"

## External API verification

*Every external API claim must carry a doc link and verification date.*

| API / Library | Claim | Doc link | Verified |
|---|---|---|---|
| `csv-stringify` (stdlib) | `stringify(records, {header: true})` | [nodejs.org/api](https://nodejs.org/api/) | 2025-01-15 |
| Supabase `auth.getSession()` | Returns session with `access_token` | [supabase.com/docs](https://supabase.com/docs) | 2025-01-15 |

## Test plan (spec-level)

*Guides the ticket-level test plans. What must be tested, at what level.*

| Layer | What to test |
|---|---|
| Unit | `ExportService.exportToCsv()`: 0 rows, typical, cap exceeded, unicode, special chars |
| Integration | `GET /api/export.csv`: auth, no auth, empty data, cap, db failure |
| E2E | Happy path: login → settings → export → verify CSV contents |

## Implementation plan (frozen at S5)

*Filled by `to-tickets` at S5, then FROZEN — this section plus the acceptance criteria are what the `converge` gate grades the built PR against. Changes after freezing are spec deltas (C23) recorded in the revision history, never silent edits. For M-size features, mirror to `docs/specs/<slug>/plan.md` so the plan survives any session.*

| Ticket | Files (file → function/change) | Depends on | Size |
|---|---|---|---|
| #231 | `src/server/export/service.ts` → `ExportService.exportToCsv()` (new); `src/server/export/service.test.ts` (new) | — | M |
| #232 | `src/app/settings/page.tsx` → export section | #231 | S |

## Revision history

| Date | Change | Author |
|---|---|---|
| 2025-01-15 | Initial spec | Agent (reviewed by [name]) |
```

---

*Save as `docs/specs/<slug>/fsd.md`. Grill with `/grill-with-docs` before approval. Every external API claim must be verified against actual documentation. The gate: every acceptance criterion is mechanically testable; **every flow has at least one If/Then requirement**; the edge-case table is filled; you would bet a day of work on this spec being right.*

*On EARS: the [Easy Approach to Requirements Syntax](https://alistairmavin.com/ears/) came out of Rolls-Royce analysing airworthiness regulations for jet-engine control (IEEE RE'09, Mavin et al.). It is borrowed here for one reason — the five patterns are a **generator**, not just a format. Kiro ships it as its default AC notation on the claim that it catches edge cases developers miss by hand. Adopted 2026-08-26; retest per the [subtraction ritual](../rituals-and-metrics.md).*
