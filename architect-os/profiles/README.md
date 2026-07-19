# Profiles — Per-Project-Type Configuration

*How one OS serves Flutter apps, Next.js apps, CLIs, and APIs without forking itself. A profile is an **overlay**, not a fork: the lifecycle, constitution, rubric, and rituals never change per project — only the stack, tools, MCPs, and type-specific rules do.*

---

## The four layers

Configuration lives in exactly four places. When you wonder "where do I put this rule?", walk this table top-down and stop at the first layer that fits:

| Layer | Lives in | Changes per… | Contains |
|---|---|---|---|
| **1. OS core** | `architect-os/` (this repo) | never | Lifecycle S0–S9, constitution C1–C37, rubric, review pipeline, memory architecture, rituals, templates |
| **2. Profile** | `architect-os/profiles/<type>.md` | project *type* | Stack + versions, harness/stack choice, MCP set, constitution **annex** (type-specific rules), CI jobs, review path-instructions, test strategy, starter gotchas |
| **3. Repo** | the project repo itself | project | `AGENTS.md` (instantiated from template + profile), `ADR-0001` (stack + profile named), `.mcp.json` / `opencode.json`, `.coderabbit.yaml`, `.github/`, repo-specific skills |
| **4. Session** | the ticket | ticket | The narrow context: ticket + linked FSD section + planned files |

Two hard rules keep this sane:

1. **The constitution is never edited per project.** Type-specific rules are an **annex** with their own prefix (`F1…` for Flutter, `W1…` for web, `A1…` for API) declared in the profile and cited in reviews exactly like C-rules. C-rules always apply; annex rules apply where the profile is active. Amending C1–C37 itself remains an ADR-gated event (see constitution.md).
2. **`harness-matrix.md` is reference, not configuration.** It's the market map — you read it to *choose*; the choice is *recorded* in the profile (per type) and `ADR-0001` (per repo). If you find yourself editing the matrix to configure a project, stop — you want a profile.

## Instantiating a repo from a profile

```
1. Pick the profile (or write a new one from an existing one — they're one file).
2. Run the repo setup checklist (README.md) as usual.
3. Where the checklist says "copy github/" → copy base, then apply the profile's
   "CI & checks" section (add/replace jobs, update ruleset required checks to match).
4. Where it says "write AGENTS.md" → start from templates/AGENTS.md, fill the
   profile's "AGENTS.md seed" section (commands, gotchas, annex pointer).
5. Append the profile's "path instructions" block to .coderabbit.yaml.
6. Create .mcp.json (Claude Code) or opencode.json (OpenCode) from the profile's
   MCP + model-routing section.
7. Record it: ADR-0001 names the stack AND the profile ("This repo uses
   profiles/flutter.md @ <commit>"). Deviations from the profile are ADR
   line-items, same as any architecture decision.
```

## What a profile may and may not override

| May override | May NOT override |
|---|---|
| Stack, language, framework versions | Lifecycle stages and gates |
| Harness + model routing (within the two named stacks) | C1–C37 (annex rules add, never subtract) |
| MCP servers | PR ≤400 lines, WIP ≤2, fix rounds ≤2 |
| CI jobs and required-check names | Human-review-first ordering, C36 family rule |
| Test strategy (what "tests" means for this type) | Memory architecture (AGENTS.md/docs/graph/dumps) |
| `area:*` labels, review path-instructions | Severity semantics 🔴🟠🟡🔵 |

## Current profiles

| Profile | For | Status |
|---|---|---|
| [flutter.md](flutter.md) | Flutter/Dart mobile (+ desktop/web targets) | active |
| [web-nextjs.md](web-nextjs.md) | Next.js full-stack web (the tech-stack.md default) | active |

Writing a new one: copy the closest existing profile; keep every section heading (they're the contract); delete what doesn't apply rather than leaving boilerplate. A profile that's grown past ~150 lines is probably two profiles.
