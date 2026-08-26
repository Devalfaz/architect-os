# GitHub Setup — The Execution System

*Issues, Projects, labels, rulesets, PR templates, branch protection. This is the infrastructure that turns specs into merged code.*

---

## Issue labels

**Canonical definitions: [github/labels.yml](github/labels.yml)** — names, colours,
and descriptions, machine-readable and synced by the `label-sync` workflow.
**When to use each: [github/label-taxonomy.md](github/label-taxonomy.md).**
Neither list is reproduced here; a third copy drifts, and the stale one wins
whichever gets read first.

The six families, so you know what exists:

| Family | Purpose | Examples |
|---|---|---|
| **Lifecycle** | where the issue is in the pipeline | `needs-triage` → `ready-for-agent` → `in-progress` → `in-review` |
| **Size** | expected artifact depth (S1 classification) | `size:XS` ≤50 lines · `size:S` 50–150 · `size:M` 150–400 · `size:split-me` >400 |
| **Area** | which part of the system | `area:api`, `area:auth`, `area:db`, `area:security`, … |
| **Priority** | scheduling | `P0:now` → `P4:icebox` |
| **Signal** | agent-pipeline state and issue kind | `ai:ready`, `ai:self-reviewed`, `bug`, `regression`, `spec-delta`, `needs-decision` |
| **Housekeeping** | triage hygiene | `duplicate`, `dependencies`, `breaking-change`, `help-wanted` |

Add or rename a label in `labels.yml` only — the sync workflow propagates it.

### Workflow
```
Issue created → needs-triage (auto)
Issue specified (S5) → ready-for-agent, ai:ready
Branch created → in-progress (auto)
PR opened → in-review (auto), removes in-progress
PR merged → closes issue, ai:implemented (auto)
```

---

## Issue templates (`.github/ISSUE_TEMPLATE/`)

### task.yml
Fields: Linked spec (required), Implementation plan file/function level (required), Out of scope (required), Acceptance criteria Given/When/Then (required), Test plan (required), Size dropdown (XS/S/M/split-me), Agent readiness checkboxes (plan reviewed, deps merged, no unmade decisions)

### bug.yml
Fields: What happened, Reproduction steps, Expected behavior, Actual behavior, Severity (P0–P3), Regression checkbox

---

## PR template (`.github/PULL_REQUEST_TEMPLATE.md`)

Sections: What (one sentence, closes #), Files changed, Self-review checklist (plan files, AC verified, tests pass, constitution checked, no new deps, ≤400 lines, uncertainty flagged), AI assistance disclosure, Screenshots, Rollback plan

---

## Branch naming

`feat/123-slug`, `fix/124-slug`, `refactor/125-slug`, `docs/126-slug`, `chore/127-slug`
One branch = one issue = one PR.

---

## Branch protection (main)

Ruleset targeting `refs/heads/main`:
- No deletion, no force push
- Pull requests required: 1 approval (default), dismiss stale reviews
- Required status checks: CI/typecheck, CI/lint, CI/test, CI/build
- Required linear history
- Squash merge only (enforced, not merge/rebase)

---

## GitHub Projects (Delivery board)

Create Projects v2 board "Delivery" with custom fields: Stage (S0–S9), Priority (P0–P4), Size (XS/S/M/split-me), Agent (claude-code/codex/copilot/human)

### Views
- **Board:** Backlog → Ready → In Progress → In Review → Done (with automations)
- **Current Sprint:** filtered by status, sorted by priority
- **Agent Queue:** filtered `label:ai:ready`, sorted by priority — the morning scan

---

## Agent interaction rules

### Agents CAN
- Create branches, open PRs (Closes #), post comments, apply labels (in-progress, in-review, ai:implemented, ai:self-reviewed), read issues and specs

### Agents CANNOT
- Merge PRs (human approval required), close issues (you or merge automation), mark own work ready without human gate, change branch protection, add secrets, deploy

### Agent self-review comment format
Must include: files changed with rationale, constitution compliance per rule, uncertainty flags, deviations from plan

---

## CI pipeline

GitHub Actions: typecheck, lint, test, build — all required status checks. Run on every PR to main.

```yaml
name: CI
on:
  pull_request:
    branches: [main]
jobs:
  typecheck: ...
  lint: ...
  test: ...
  build: ...
```

---

## What ships in `github/`

The folder **mirrors its destination**, so installing it is a copy, not a
reassembly. `.md` files are guides you read; everything else is payload.

```
architect-os/github/              →  your-app/
├── ISSUE_TEMPLATE/*.yml          →  .github/ISSUE_TEMPLATE/
├── workflows/ci.yml              →  .github/workflows/
├── workflows/label-sync.yml      →  .github/workflows/
├── rulesets/main-ruleset.json    →  .github/rulesets/   (import via UI/API)
├── PULL_REQUEST_TEMPLATE.md      →  .github/
├── CODEOWNERS                    →  .github/
├── labels.yml                    →  .github/
├── settings.yml                  →  .github/            (Probot Settings app)
└── .coderabbit.yaml              →  repo ROOT  ⚠ not .github/
```

⚠ **Two things that bite:** `.coderabbit.yaml` must sit at the **repo root** —
CodeRabbit does not read it from `.github/`. And `settings.yml` is repo metadata
for the Settings app; it is *not* `ISSUE_TEMPLATE/config.yml`, which is the
issue-chooser config and does not ship here.

## Setup checklist
- [ ] `cp -r architect-os/github/. .github/` then `mv .github/.coderabbit.yaml .`
- [ ] Create labels: `npx github-label-sync --labels .github/labels.yml <owner>/<repo>`
- [ ] Install branch protection ruleset on main (`rulesets/main-ruleset.json`)
- [ ] Create Delivery Projects v2 board with custom fields
- [ ] Set up Project automations
- [ ] Verify CI workflow runs (`.github/workflows/ci.yml`)
- [ ] Test PR to verify: CI, CodeRabbit, squash merge
- [ ] Verify needs-triage auto-applied on new issues
