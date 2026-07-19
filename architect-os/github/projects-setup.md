# GitHub Projects Setup Guide

*Setting up the "Delivery" Projects v2 board with automations, custom fields, and views.*

---

## Step 1: Create the project

1. Go to your repo → Projects → New project
2. Choose "Board" layout
3. Name: `Delivery`
4. Description: `Architect OS delivery board — S5 through S8 tracking`

---

## Step 2: Add custom fields

Settings → Custom fields → New field:

| Field name | Type | Options |
|---|---|---|
| **Stage** | Single select | S0 Capture, S1 Frame, S2 Specify, S3 Design, S4 Architect, S5 Plan, S6 Implement, S7 Review, S8 Release, S9 Learn |
| **Priority** | Single select | P0:now, P1:next, P2:soon, P3:later, P4:icebox |
| **Size** | Single select | XS, S, M, split-me |
| **Agent** | Single select | claude-code, codex, copilot, human |

---

## Step 3: Configure board columns

| Column | Meaning | Auto-add when... |
|---|---|---|
| **Backlog** | Unprioritized work | Default for new issues |
| **Ready** | `ai:ready` — gated, executable | Label `ai:ready` applied |
| **In Progress** | Being implemented | Branch created (linked to issue) |
| **In Review** | PR open, review active | PR opened (linked to issue) |
| **Done** | Merged and deployed | PR merged (linked to issue) |

Settings → Workflows:
- **Item added to project:** Set Stage to "S5 Plan" if label is `ready-for-agent`
- **Item reopened:** Move to Ready column
- **Item closed:** Move to Done column
- **Pull request opened:** Move linked issue to In Review
- **Pull request merged:** Move linked issue to Done

---

## Step 4: Create views

### View 1: Board (default view)
Layout: Board, grouped by Status. Columns as configured above.

### View 2: Current Sprint
Layout: Table. Filter: `Status: Ready, In Progress, In Review`. Sort: Priority ascending (P0 first).

Visible fields: Title, Priority, Size, Agent, Stage, Assignees, Labels.

### View 3: Agent Queue
Layout: Table. Filter: `Label: ai:ready`. Sort: Priority ascending.

This is the morning scan — what agents should work on today.

### View 4: By Stage (pipeline)
Layout: Board, grouped by Stage. See where everything is in the S0–S9 pipeline.

### View 5: By Area
Layout: Board, grouped by area label. See what's in progress per domain.

---

## Step 5: Test the board

1. Create a test issue from `task.yml` template
2. Verify it appears in Backlog with `needs-triage` label
3. Apply `ready-for-agent` + `ai:ready` → verify it moves to Ready
4. Create a branch `feat/999-test` → verify issue moves to In Progress
5. Open a PR referencing "Closes #999" → verify issue moves to In Review
6. Squash merge the PR → verify issue moves to Done and auto-closes

---

## Step 6: Daily workflow

1. **Morning:** Open "Agent Queue" view → pick 1–2 tickets → launch agents
2. **Midday:** Check "Current Sprint" view → any PRs needing review?
3. **Afternoon:** Move merged items from Done to archive (optional, quarterly cleanup)

---

## Optional: Add a "Spec Queue" view for S0–S2 work

Create an additional view filtered to Stage: S0/S1/S2. This is your idea → spec pipeline — separate from implementation tracking.
