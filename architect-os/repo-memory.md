# Repo Memory — The Four-Layer Architecture

*Agents start every session with zero context. Without memory, every session is a new engineer on their first day. With memory, every session builds on all previous ones.*

---

## Why agents need explicit memory

Agents know programming from training data, but not:
- Your project's domain language
- Architecture decisions and why
- Which APIs are actually installed (vs hallucinated)
- Past bugs and fixes
- File-to-concern mapping
- Unwritten conventions

---

## Layer 1: AGENTS.md — The entrypoint

**File:** `AGENTS.md` (root) + symlink `CLAUDE.md → AGENTS.md`
**Purpose:** First thing every agent reads. ≤150 lines.

**Contents:** Project one-liner, Commands the agent can't guess, Domain language (5–10 terms), Binding architecture decisions (3–5, each linking its ADR), Constitution summary (5 most violated rules), Gotchas (from dumps), Do-not-touch list, Active constraints.

**Deliberately excluded** (Anthropic's include/exclude guidance — the [template](templates/AGENTS.md) carries the full table): tech-stack lists (readable from `package.json`), file-tree conventions (readable from the tree), API docs (link instead), anything that changes frequently. The pruning test for every line: *"would removing this cause the agent to make a mistake?"* Bloated entrypoints make agents ignore the rules that matter — if a rule keeps being violated, the file is too long, not too gentle.

**Freshness:** Verified weekly. `last_verified` checked at distill.

---

## Layer 2: The docs tree

```
docs/
├── product/        ideas, BRDs, PRDs
│   └── <slug>/     brd.md, prd.md
├── specs/          FSDs (implementation contracts)
│   └── <slug>/     fsd.md
├── adr/            numbered decision records
├── architecture/   architecture.md + diagrams
├── research/       spikes with expiry dates
└── agents/         verified-apis.md, project-gotchas.md, retro-YYYY-WW.md, skill-changelog.md
```

### ADR format
Title, Status (proposed/accepted/deprecated/superseded), Context, Decision, Alternatives, Consequences, Agent instruction line, Compliance check (mechanical), last_verified

### Research expiry
Every file in `docs/research/` has `research_date` and `expiry_date`. After expiry: stale — re-verify or archive.

---

## Layer 3: The knowledge graph

**File:** `memory/repo-graph.json`

### Node types
`module` (logical grouping), `file` (source file), `concept` (domain term), `decision` (ADR), `api` (endpoint), `test` (test suite), `workflow` (cross-cutting flow)

### Node schema
```json
{
  "id": "file:src/server/auth/service.ts",
  "type": "file",
  "label": "AuthService",
  "properties": {
    "description": "...",
    "exports": [],
    "dependencies": [],
    "test_coverage": "high",
    "last_modified": "2025-01-14",
    "last_verified": "2025-01-15"
  }
}
```

### Edge types
`depends_on`, `implements`, `tests`, `owns`, `calls`, `constrained_by`, `related_to`, `supersedes`, `caused_by`

### Temporal validity & provenance (schema v1.1)
Nodes and edges optionally carry `valid_from` / `valid_to` / `invalidated_by` / `sources`. A fact with `valid_to` set is **history, not truth** — it stays in the graph for provenance ("what did we believe on date X, and what changed our mind") but agents must not act on it. Invalidation happens at *write time*, the moment an agent discovers the contradiction — not at the weekly distill ([protocol](memory-freshness-protocol.md)). `sources` points every fact back to its origin: a dump file, an ADR, a spec.

### Ego-network loading
Agent loads: file nodes in ticket plan + all edges from those nodes + connected concepts/ADRs → ~10–50 nodes, **excluding anything with `valid_to` set**. Programmatic navigation, not context dump.

---

## Layer 4: Session memory dumps

**Directory:** `memory/dumps/` — `YYYY-MM-DD.md`

### Dump fields
What changed, Decisions, Surprises, Hallucinations caught, Graph delta candidates, Skill friction

### Dump → Graph pipeline
- **Daily (5 min):** One dump per coding day
- **Weekly (30 min):** `/graph-update` reads dumps, produces graph delta PR, you review and merge
- **Monthly (in retro):** Review graph coverage — orphans? stale edges? concepts without files?

---

## Memory freshness protocol

See [memory-freshness-protocol.md](memory-freshness-protocol.md).

1. Every doc and graph node has `last_verified`
2. AGENTS.md: verified weekly. Architecture: monthly. ADRs: when constrained code changes. Research: expiry date. Graph: weekly distill.
3. Stale memory is flagged, not deleted. Stale = hypothesis, not fact.
4. Weekly distill is the enforcement point.

---

## The summary

| Layer | What | When read | Freshness |
|---|---|---|---|
| AGENTS.md | Entrypoint, ≤150 lines | Every session start | Weekly |
| docs/ | Specs, ADRs, architecture | Per ticket (linked) | Monthly / on change |
| repo-graph.json | Nodes + edges | Per ticket (ego-network) | Weekly from dumps |
| memory/dumps/ | Daily summaries | Weekly distill → graph | Daily write, weekly distill |

No layer alone is sufficient. Together: compound memory that gets smarter every cycle.
