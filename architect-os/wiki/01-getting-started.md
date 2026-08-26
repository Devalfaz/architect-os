# 01 — Getting Started

*Two installations: once per machine, once per repo. Then a calibration period where you deliberately run only half the system.*

---

## Step 0 — Pick your stack (one decision, up front)

The OS runs on either of two named stacks ([adoption-plan.md](../adoption-plan.md) § The two stacks):

- **Default stack** — Claude Code + Anthropic models. Most mature agent loop (skills, subagents, hooks, Routines). ~$232–249/mo.
- **Open-frontier stack** — OpenCode + DeepSeek/OpenRouter. Same OS, ~$32–65/mo, younger harness.

Pick per repo, record it in ADR-0001, don't mix mid-ticket. Unsure? Start with the default stack — every walkthrough in the docs assumes it; switch a repo later if the bill argues (migration is cheap: everything is markdown).

## Step 1 — Machine setup (once, ~1 hour)

Follow the machine checklist in [README.md](../README.md), which per stack means:

1. Install the harness (Claude Code on a Max plan, or OpenCode + DeepSeek/OpenRouter keys — [harness-matrix.md](../harness-matrix.md) § Setup checklist).
2. Install `gh` CLI, authenticated. Install Codex CLI — it's the standing C36 cross-family reviewer, not an optional extra.
3. Install skills to `~/.claude/skills/` (or `.opencode/skills/`): Matt Pocock's set (`npx skills@latest add mattpocock/skills`) + the OS-native skills ([skills-catalog.md](../skills-catalog.md)) including [`converge`](../skills/converge/SKILL.md).
4. Put `~/.claude/skills` under git. Non-negotiable — skills without versioning rot invisibly (failure mode #12... check [failure-modes.md](../failure-modes.md)).
5. Install the CodeRabbit GitHub App on your account.

## Step 2 — Repo setup (per repo, ~1 hour the first time, <30 min after)

1. **Pick a profile** — [profiles/](../profiles/README.md). Flutter → [flutter.md](../profiles/flutter.md); Next.js → [web-nextjs.md](../profiles/web-nextjs.md); neither → copy the closest and adapt.
2. Run the repo checklist in [README.md](../README.md): create repo (squash-only), copy `github/` → `.github/` **applying the profile's CI section**, sync labels, import the [branch ruleset](../github/rulesets/main-ruleset.json) (update required-check names to the profile's job names), create the Delivery project ([guide](../github/projects-setup.md)).
3. Drop `.coderabbit.yaml` at repo root ([base](../github/.coderabbit.yaml) + the profile's path-instructions block).
4. Write `AGENTS.md` from the [template](../templates/AGENTS.md) + the profile's seed section. `ln -s AGENTS.md CLAUDE.md`. Have the agent draft it from the code, then **you correct it** — the corrections are the value.
5. Create `docs/` + `memory/` trees; commit an empty graph conforming to the [schema](../memory/repo-graph.schema.json).
6. Write ADR-0001: stack + profile + deviations. Write ADR-0002 if the architecture style isn't the profile default.
7. Hello-world PR through the **full** pipeline (converge → rubric → CodeRabbit → merge) before any feature work — you're testing the pipeline, not the code.

## Step 3 — Calibration (weeks 1–4)

Do **not** attempt the whole lifecycle on day 1. Per the [adoption plan](../adoption-plan.md) 30/60/90: run only S5→S9 (tickets → implement → review → merge → dump) on one repo for the first month. The front half (BRD/PRD/FSD discipline) joins in month two. Exit criteria before adding more ceremony: first-pass acceptance ≥40%, zero merges without the rubric, dumps exist for every merge day.

## Existing repos

Same as Step 2, plus: have the agent draft `architecture.md` and an initial repo graph from the code (you correct both); expect AGENTS.md to be wrong for two weeks (every wrong claim it makes becomes a gotcha entry — that's the system learning); start with S5→S9 only, and only on well-understood areas of the codebase until the graph fills in.
