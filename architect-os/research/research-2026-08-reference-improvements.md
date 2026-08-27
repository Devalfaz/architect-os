---
status: snapshot
last_verified: 2026-08-27
expires: 2026-11-27
---
<!-- SNAPSHOT: findings below were verified against the linked upstream source files on last_verified. -->

# Research Report: Reference Improvements for Architect OS

*Four GitHub repositories were inspected on 2026-08-27. This is an evidence
report, not a change to architect-os policy. Recommendations are deliberately
scoped to practices that can be adopted in the existing workflow.*

## Executive Summary

Architect-os already has unusually strong foundations for narrow-context work,
human gates, dated research, and skill evaluation. The references suggest four
concrete upgrades:

1. **Make skill authoring behavior-tested by default.** Superpowers treats a
   skill like production code: establish a failing baseline, write the minimum
   instruction, then re-run the scenario and close loopholes.
2. **Make skill metadata and trigger behavior testable.** Superpowers requires
   skill invocation before action and uses trigger-only descriptions; taste-skill
   also demonstrates explicit install-name metadata. Architect-os has the
   catalog, but not a repository-level contract for checking these interfaces.
3. **Add a reusable design-system/preflight workflow for UI work.** UI UX Pro
   Max combines domain search, industry-specific rules, anti-patterns, and a
   responsive/accessibility checklist. Taste-skill adds brief inference,
   explicit design dials, redesign audit mode, and strict scope boundaries.
4. **Adopt action-first progress reporting as a default output shape.**
   i-have-adhd provides a compact protocol: lead with the next action, number
   bounded steps, restate state, show wins, and end with one concrete next step.

The highest-leverage near-term work is to add a small skill-evaluation contract
and a UI-specific preflight template. The communication rules are low-cost and
can be folded into existing agent-facing guidance without a new subsystem.

## Current Architect-os Baseline

The repository already has:

- A dated `research/` evidence layer with `last_verified` and `expires` fields.
- A skills catalog with trigger guidance, OS-native skills, and a documented
  edit-time evaluation loop using fresh sessions and 2–3 scenarios.
- A `converge` gate that grades implementation against frozen acceptance
  criteria and plans before human review.
- Human-first review, bounded fix loops, cross-family review, and explicit
  evidence-before-completion rules.
- Context discipline: load on demand, one ticket's context, capped reads, and
  fresh sessions for tickets.

This means the recommendations below are refinements and reusable artifacts,
not a proposal to replace the lifecycle.

## Findings and Adoption Opportunities

### 1. Test skills as behavioral artifacts

**Evidence.** Superpowers explicitly calls skill writing “Test-Driven
Development applied to process documentation.” Its mapping is concrete:
pressure scenario = test case, agent violation without the skill = RED,
compliance with the skill = GREEN, and new rationalizations = refactor. It also
states that without observing the baseline failure, the author cannot know
whether the skill teaches the right behavior.

Source: [superpowers `writing-skills/SKILL.md`](https://github.com/obra/superpowers/blob/main/skills/writing-skills/SKILL.md#tdd-mapping-for-skills)

**Architect-os gap.** `skills-catalog.md` already requires 2–3 fresh-session
scenarios after an active skill edit, but it does not provide a standard
scenario format, baseline/pass evidence fields, or a single command/checklist
for running them. The practice is present as guidance, not yet as a durable
artifact contract.

**Adopt.** Add a `templates/skill-eval.md` template and require each actively
maintained skill to record: task prompt, expected behavior, baseline failure,
skill version, post-change result, and unresolved loopholes. Keep evaluation
focused on observable agent behavior, not prose quality. Add the template to
the catalog's existing edit-time loop; do not add a new lifecycle gate for
one-off documentation.

**Priority:** High. **Cost:** Small. **Success signal:** every maintained skill
has at least two reproducible scenarios, including one scenario that failed
without the skill.

### 2. Treat skill discovery as an interface

**Evidence.** Superpowers makes `using-superpowers` a session-start skill and
requires invocation before any response or file exploration when a skill might
apply. Its skill-writing guide says frontmatter descriptions should state only
when to use the skill, begin with a concrete trigger, and avoid summarizing the
workflow because agents may follow the shortcut instead of reading the body.
It also recommends search terms covering symptoms, synonyms, tools, and error
messages.

Sources: [superpowers `using-superpowers/SKILL.md`](https://github.com/obra/superpowers/blob/main/skills/using-superpowers/SKILL.md),
[superpowers `writing-skills/SKILL.md`](https://github.com/obra/superpowers/blob/main/skills/writing-skills/SKILL.md#skill-discovery-optimization-sdo)

**Additional evidence.** Taste-skill distinguishes a skill's folder name from
its frontmatter install name and documents the exact install name users pass to
the CLI. That distinction is useful, but the repository also has an open issue
reporting mismatches between the documented names and local script keys.

Sources: [taste-skill README](https://github.com/Leonxlnx/taste-skill/blob/main/README.md#installing),
[taste-skill naming mismatch issue](https://github.com/Leonxlnx/taste-skill/issues/76)

**Architect-os gap.** The catalog describes when to fire skills, but there is no
mechanical check for description shape, unique names, or agreement between the
catalog, folder, and frontmatter. This is especially relevant because
architect-os treats third-party skills as executable instructions.

**Adopt.** Define a small skill metadata contract: unique hyphenated name,
trigger-only description beginning with `Use when`, explicit “not for” scope
when ambiguity is likely, and catalog entry matching the frontmatter name.
Validate it with a lightweight script or CI check. Include a negative trigger
case in the behavior eval so a skill is not loaded for adjacent work.

**Priority:** High. **Cost:** Small. **Success signal:** metadata CI catches
duplicate/mismatched names and each skill has a positive and negative trigger
scenario.

### 3. Generate design guidance from the brief, then preflight it

**Evidence.** UI UX Pro Max's base skill starts by mapping scenarios to a
workflow: new pages go through design-system generation, component work uses a
focused domain search, and existing UI review starts from a checklist. Its
generator searches product type, style, color, landing patterns, and typography,
then emits a pattern, style, colors, typography, effects, anti-patterns, and a
pre-delivery checklist. The repository includes industry-specific reasoning
rules and reports 119 UX guidelines across accessibility, resilient text, and
cancellable interactions.

Sources: [UI UX Pro Max README](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill/blob/main/README.md#how-design-system-generation-works),
[UI UX Pro Max base skill template](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill/blob/main/src/ui-ux-pro-max/templates/base/skill-content.md#how-to-use-this-skill)

Its checklist is concrete: contrast, visible focus, reduced motion, reflow at
375/768/1024/1440px, and no clipping of labels. Its resilient-text guidance
also requires accessible handling for wrapped chips, truncated values, and
color-independent badge meaning.

Source: [UI UX Pro Max README, pre-delivery and resilient-text guidance](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill/blob/main/README.md#resilient-text-and-compact-ui)

**Architect-os gap.** `visual-appendix.md` and the prototype skill support UI
exploration, and the constitution contains general quality rules, but there is
no reusable brief-to-design-system output contract or compact UI preflight that
can be attached to a design brief/FSD.

**Adopt.** Add a `templates/ui-preflight.md` (or a UI section to the existing
design brief template) with four fields: audience/use case, design-system or
style choice, known anti-patterns, and verification matrix. Require evidence
for responsive widths, keyboard focus, contrast, reduced motion, text/chip
reflow, and semantic state. Keep it as a checklist attached to UI work, not a
global requirement for backend tickets.

**Priority:** High for frontend repos. **Cost:** Small to medium. **Success
signal:** UI PRs link a completed preflight and record failures as follow-up
tickets rather than silently accepting visual regressions.

### 4. Infer design intent and expose controlled variation

**Evidence.** Taste-skill requires a one-line “Design Read” before code. It
extracts page kind, user-provided vibe, references, audience, existing brand
assets, and quiet constraints; accessibility, public-sector, regulated, and
trust-first constraints override aesthetic preference. It then maps the read to
three explicit dials: `VARIANCE`, `MOTION`, and `DENSITY`, with presets for
landing pages, portfolios, public-sector services, and redesign-preserve versus
redesign-overhaul.

Sources: [taste-skill `SKILL.md`, brief inference](https://github.com/Leonxlnx/taste-skill/blob/main/skills/taste-skill/SKILL.md#0-brief-inference-read-the-room-before-anything-else),
[taste-skill dial inference](https://github.com/Leonxlnx/taste-skill/blob/main/skills/taste-skill/SKILL.md#1-brief-inference-design-read--dial-values)

**Architect-os gap.** The design brief template captures requirements, but it
does not force an explicit audience/design-language read or distinguish a
preserve redesign from an overhaul. This leaves an agent more room to invent a
default aesthetic than the spec-driven philosophy otherwise allows.

**Adopt.** Add optional UI-only fields to `templates/design-brief.md`: “Design
Read,” audience, existing assets, quiet constraints, and three 1–10 dials. Make
the dials descriptive controls, not a claim that aesthetic quality is
quantifiable. For redesigns, require an audit before implementation and a
decision to preserve or overhaul.

**Boundary.** Taste-skill explicitly excludes dashboards, dense product UI,
data tables, multi-step forms, code editors, native mobile, and realtime
collaboration. Do not apply its landing-page defaults to those surfaces; route
them to the appropriate design system or product-specific guidance.

Source: [taste-skill out-of-scope section](https://github.com/Leonxlnx/taste-skill/blob/main/skills/taste-skill/SKILL.md#13-out-of-scope)

**Priority:** Medium. **Cost:** Small. **Success signal:** design briefs expose
the intended audience and constraint hierarchy before implementation begins.

### 5. Make progress communication action-first

**Evidence.** i-have-adhd defines ten output rules: lead with the next action,
number multi-step work, end with one concrete next step, suppress tangents,
restate state each turn, use specific time estimates, make wins visible, and
keep lists to five items. Its source explains the mechanism: working memory is
limited, starting is hard, vague time estimates fail, and visible progress
matters. It also says task/plan tools should carry the state for multi-step
work.

Sources: [i-have-adhd README](https://github.com/ayghri/i-have-adhd/blob/main/README.md#the-rules),
[i-have-adhd `SKILL.md`](https://github.com/ayghri/i-have-adhd/blob/main/skills/i-have-adhd/SKILL.md#what-adhd-changes-about-reading)

**Architect-os fit.** Architect-os already uses incremental updates, task
states, bounded lists, and explicit gates. The missing piece is a concise
default output shape for agent updates and final reports; the current guidance
can still permit a useful action to be buried beneath context.

**Adopt.** Add to agent-facing communication guidance: first line = current
action or result; use numbered steps for more than one action; state
`done / in progress / blocked`; surface one next action; keep rationale after
the action. Treat the style as a default, not a diagnosis or mandatory persona,
and allow the user to request a different style.

**Priority:** Medium. **Cost:** Tiny. **Success signal:** progress updates make
the current state and next action scannable without reading the full transcript.

### 6. Preserve scope boundaries and honest output completion

**Evidence.** Taste-skill says each rule is contextual, instructs the agent to
read the brief before pulling rules, and has an explicit out-of-scope section.
Its final preflight is non-optional and includes design-read, dial, design
system, redesign audit, theme/color/shape consistency, contrast, and wrapping
checks. Its README also separates implementation skills from image-generation
reference skills and says each skill does one job.

Sources: [taste-skill `SKILL.md`](https://github.com/Leonxlnx/taste-skill/blob/main/skills/taste-skill/SKILL.md#tasteskill-anti-slop-frontend-skill),
[taste-skill final preflight](https://github.com/Leonxlnx/taste-skill/blob/main/skills/taste-skill/SKILL.md#14-final-pre-flight-check),
[taste-skill README](https://github.com/Leonxlnx/taste-skill/blob/main/README.md#skills)

Superpowers makes the analogous engineering claim through a workflow catalog:
brainstorming, planning, TDD, debugging, parallel agents, review, and branch
finishing are separate skills, with evidence-over-claims as a core philosophy.

Source: [superpowers README](https://github.com/obra/superpowers#available-skills)

**Architect-os fit.** This reinforces C6 scope control, the frozen-plan
`converge` gate, C24's bounded fix loop, and the existing “skills are
executable instructions” warning. The practical addition is to make “out of
scope” and “done means every preflight box passes” standard fields in new
skill/template design.

**Priority:** Medium. **Cost:** Small. **Success signal:** fewer skills that
apply broad defaults outside their domain, and fewer reports that claim
completion while omitting a required verification step.

## Recommended Order

1. Add skill metadata checks and `templates/skill-eval.md`; this strengthens
   the existing catalog with a testable contract.
2. Add `templates/ui-preflight.md` and the brief/design-read fields for
   frontend project profiles.
3. Add the action-first update shape to the agent template and daily loop.
4. Review the first month of skill evaluations and UI preflights; promote only
   practices with observable improvement into constitution-level rules.

## Source Index

- [obra/superpowers](https://github.com/obra/superpowers)
- [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)
- [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill)
- [ayghri/i-have-adhd](https://github.com/ayghri/i-have-adhd)

## Caveats

- Repository README claims and feature counts are treated as maintainer/vendor
  claims unless they describe directly inspectable source behavior.
- Taste-skill is explicitly marked experimental in its README; adopt its
  inference and preflight patterns, not its aesthetic defaults wholesale.
- i-have-adhd is an output-style skill, not clinical guidance. The proposed
  communication rules are productivity heuristics and should remain user
  overridable.
- Re-verify this report after 2026-11-27, consistent with architect-os research
  freshness rules.
