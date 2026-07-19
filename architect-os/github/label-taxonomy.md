# Label Taxonomy Reference

*When to use each label. Print or pin. This is the decision guide for triage and ticket grooming.*

---

## Quick decision: which label in which situation

### "I just got a new issue"
→ Apply `needs-triage` if it's raw. If the reporter is internal and it's clear, go straight to sizing + area.

### "This is ready for the agent to work on"
→ Apply `ready-for-agent` + `ai:ready` + size label + area label + priority label.

### "This is too big for one ticket"
→ Apply `size:split-me`. Do not pass go. Decompose at S5.

### "This is a bug, not a feature"
→ Apply `bug`. If it's a regression (worked before, broken now): also `regression`. Triggers diagnosing-bugs workflow.

### "This is blocked on a decision"
→ Apply `needs-decision`. Reference the decision in a comment. Do not mark `ai:ready` until resolved.

### "An agent opened a PR for this"
→ Automation applies `in-review` + `ai:implemented`. After self-review: `ai:self-reviewed`.

### "I'm rejecting this"
→ Apply `wontfix`. Must include reason in closing comment.

---

## Size decision table

| How many files? | How many lines (net)? | Size label | Agent? |
|---|---|---|---|
| 1 | ≤50 | `size:XS` | Fire-and-forget (Copilot Workspace / Codex Cloud) |
| 1–3 | 50–150 | `size:S` | Agent-primary (Claude Code, fresh session) |
| 3–5 | 150–400 | `size:M` | Agent-primary, max size for single ticket |
| Any | >400 | `size:split-me` | Split first. Cannot go to agent as-is. |

---

## Priority decision table

| Situation | Priority |
|---|---|
| Production is down, data loss, security incident | `P0:now` |
| Core feature broken, no workaround, blocks release | `P1:next` |
| Broken but workaround exists, nice-to-have this sprint | `P2:soon` |
| Backlog, good first issue, tech debt | `P3:later` |
| Someday, icebox, "would be cool" | `P4:icebox` |

---

## Area mapping

| If the work touches... | Apply |
|---|---|
| Route handlers, API endpoints, request/response | `area:api` |
| Login, logout, sessions, permissions, Supabase auth | `area:auth` |
| React components, pages, CSS, shadcn/ui | `area:ui` |
| Schema, migrations, queries, Drizzle, seed data | `area:db` |
| CI workflows, deployment config, Docker, hosting | `area:infra` |
| Markdown files, ADRs, specs, README | `area:docs` |
| Test files, test utilities, E2E, Playwright config | `area:test` |
| N+1 queries, bundle size, Lighthouse, profiling | `area:perf` |
| Input validation, auth middleware, secrets, sanitization | `area:security` |

---

## Label application flow

```
Issue created → [template auto-applies] needs-triage
              → [you/agent triage] area:*, size:*, priority:*

S5 gated      → ready-for-agent, ai:ready

Branch created → [GitHub Actions auto] in-progress

PR opened     → [GitHub Actions auto] in-review, removes in-progress
              → [agent self-review] ai:self-reviewed

PR merged     → [GitHub auto] closes issue
              → [GitHub Actions auto] ai:implemented
```
