# Harness Matrix — Which Tool, Which Layer, and Why

*Every tool comparison in one place, cited to primary sources. The question isn't "which tool is best" — it's "which tool for which job at which stage."*

---

## The shape of the market (mid-2025)

Three categories of AI coding tools exist, and the mistake is comparing across categories:

| Category | Example tools | What they do | Architecture |
|---|---|---|---|
| **IDE extensions** | Copilot Chat, Cursor, Copilot agent mode | Add AI inside your editor | Session-bound, synchronous, human-initiated |
| **CLI agents** | Claude Code, Codex CLI | Full agent in the terminal — plan, edit files, run commands, self-correct | Fresh session per run, file-grounded, autonomous loop |
| **Cloud/platform agents** | Copilot Workspace, Copilot coding agent, Traycer, CodeRabbit | Hosted services triggered by platform events (issue opened, PR created) | Fire-and-forget or interactive web UX, not tied to developer's machine |

The Architect OS uses **CLI agents as the primary harness** (they read real files, run real tests, leave a full audit trail) and **platform agents for async/always-on tasks** (PR review, XS tickets). IDE extensions are support infrastructure, not the main event.

---

## Claude Code

**Source:** [docs.anthropic.com](https://docs.anthropic.com/en/docs/claude-code/overview), Anthropic official documentation and release notes.

### What it is

Anthropic's terminal-based agent. It's not a chat client — it's an autonomous agent that reads files, writes code, runs commands, tests its own work, and self-corrects in a loop until the task is done. Runs Claude Opus 4.x and Sonnet 4.x models.

### Key capabilities

- **Plan mode:** read-only exploration and structured planning before writing code. Claude reads the repo, proposes an approach, you approve before it touches anything.
- **Agentic loop:** read file → plan → edit → run test → read error → fix → re-run test → green. Loops autonomously within guardrails.
- **Skills system:** `.claude/skills/` directory holds reusable workflow definitions (user-invoked and model-invoked). Matt Pocock's skills ecosystem plus custom OS-native skills.
- **CLAUDE.md / AGENTS.md:** project-level instructions consumed at session start. The agent entrypoint — ~150 lines of constitution, conventions, and domain language.
- **Context isolation:** fresh session per ticket. No session-to-session memory (by design — memory lives in files, not in the agent).
- **Permission model:** you approve file writes, terminal commands, and network access per session or per operation.
- **Model routing:** defaults to Claude Sonnet for most tasks, escalates to Opus for complex reasoning. Configurable via `--model`.

### Pricing

| Plan | Price | What you get |
|---|---|---|
| Claude Pro | $20/mo | Limited Claude Code usage, rate-limited |
| Claude Max | $100–$200/mo | 5×–20× usage of Pro. $100 = 5×, $200 = 20× |
| Claude Max (API key) | $200/mo + API usage | Max plan + bring-your-own API key for unlimited API access. API costs: Opus 4.6 $5/$25 per 1M input/output; Sonnet 4.6 $3/$15; Haiku 4.5 $1/$5 |

**Source:** Anthropic pricing page; Reuters report on Max plan launch, April 2025.

### Where it fits in Architect OS

| Stage | Role |
|---|---|
| S1 Frame (BRD) | Primary — interactive grilling + drafting |
| S2 Specify (PRD/FSD) | Primary — frontier model, spec creation + grill-with-docs |
| S3 Design | Primary + Figma MCP |
| S4 Architect | Primary — plan mode for options, ADR drafting |
| S5 Plan (tickets) | Primary — to-tickets + wayfinder, reads real files |
| S6 Implement | Primary — one fresh session per ticket, narrow context |
| S7 Review | Secondary — self-review pass via code-review skill |
| S9 Learn | Primary — mechanical, can use cheap model |

### Best for

Every stage that requires reasoning about the actual codebase in a fresh, narrow-context session. This is the default harness for Architect OS.

### Limitations

- No async fire-and-forget — you launch a session and it runs synchronously (or backgrounded). Can't be triggered by a GitHub webhook.
- Context windows — a fresh session means fresh context. If the agent needs info from a previous session, it must be in AGENTS.md, docs, or the ticket body.
- Human gates required — the agent will keep going until it's done or blocked. You must set explicit completion criteria.

---

## OpenAI Codex CLI & Codex Cloud

**Source:** [platform.openai.com/docs](https://platform.openai.com/docs), OpenAI official documentation.

### What it is

OpenAI's terminal-based coding agent, analogous to Claude Code. Runs GPT-5.2 and o3 models. Its architectural distinction: native multi-sandbox execution (can run code in isolated environments for safety) and strong reasoning model support.

### Key capabilities

- **Agentic loop:** read → plan → edit → test → fix, similar to Claude Code.
- **Multi-model routing:** can use o3 for reasoning-heavy tasks (planning, architecture), GPT-5.2 for implementation, GPT-5 mini for mechanical work.
- **Sandbox execution:** runs terminal commands in isolated sandboxes by default — safer for untrusted code paths but adds latency.
- **Cloud (async) mode:** Codex can run in the cloud (Codex Cloud), triggered via API or webhook. This is the closest alternative to "Copilot coding agent" for async fire-and-forget.
- **Review mode:** `codex review` command does a structured code review of a PR, similar to CodeRabbit but ad-hoc rather than always-on.

### Pricing

| Model | Input ($/1M tokens) | Output ($/1M tokens) |
|---|---|---|
| GPT-5.2 | $1.75 | $14.00 |
| GPT-5.2 Pro | $21.00 | $168.00 |
| o3 | $10.00 | $40.00 |
| o4-mini | $1.10 | $4.40 |
| GPT-5 mini | ~$0.15–$0.60 | ~$0.60–$2.40 |

**Source:** OpenAI platform pricing page, mid-2025.

### Where it fits in Architect OS

| Stage | Role |
|---|---|
| S6 Implement | Alternative — second implementation opinion when you want to compare approaches |
| S7 Review | Supplementary — `codex review` as second AI reviewer, alongside CodeRabbit |
| S6 Async | Alternative — Codex Cloud can be the async fire-and-forget for XS tickets if Copilot coding agent isn't available |

### Best for

- **Second-opinion implementation:** run the same ticket through Claude Code AND Codex, compare the approaches, pick the better one. Doubles cost but can surface blind spots.
- **Code review second pass:** `codex review` catches different things than CodeRabbit because it uses different models.
- **Async execution:** Codex Cloud for fire-and-forget XS tickets when you're not on GitHub Enterprise (where Copilot Workspace would be the native option).

### Claude Code vs Codex CLI — the practical difference

| Dimension | Claude Code | Codex CLI |
|---|---|---|
| Primary model | Claude Sonnet/Opus 4.x | GPT-5.2 / o3 |
| Reasoning strength | Opus excels at nuanced trade-off analysis | o3 excels at structured, multi-step decomposition |
| Code generation | Sonnet — crisp, idiomatic, fewer hallucinations | GPT-5.2 — broad language support, more verbose |
| Skills ecosystem | Mature (Pocock skills + custom MD files) | Less mature, file-based instructions |
| Context grounding | Reads files directly, strong at following patterns | Strong but more prone to "creative" deviations |
| Cost per session | ~$3–$15 (Sonnet) to ~$25–$50 (Opus) for a typical ticket | ~$2–$8 (GPT-5.2) to ~$15–$40 (o3) |

Practical rule: use Claude Code as primary (better at following your conventions and maintaining narrow scope); use Codex as the second opinion when you're unsure about an approach.

---

## GitHub Copilot (Full Platform)

**Source:** [docs.github.com/copilot](https://docs.github.com/copilot), GitHub official documentation, GitHub Blog September 2025.

### What it is

GitHub's layered AI platform — not one tool, but four:

1. **Copilot IDE** (completions + Chat + agent mode preview)
2. **Copilot Workspace** (browser-based Issue → Plan → Spec → PR, Enterprise only)
3. **Copilot coding agent** (async cloud agent: assign issue → PR appears, prototype/announced)
4. **Copilot Code Review** (inline AI PR review, Enterprise only)

### Key capabilities (by layer)

**Copilot Chat + @workspace:**
- `@workspace` participant uses embeddings-based codebase search
- `/tests`, `/fix`, `/explain`, `/doc` slash commands
- Multi-model: GPT-4o, Claude 3.5 Sonnet, Gemini 2.0 Flash (user-selectable in Pro+)
- Model selection affects chat only — completions use a proprietary model

**Copilot Workspace (Enterprise, $39/mo/user):**
- Opens from any Issue → plan phase → spec phase → implement phase → PR
- Human gates at plan and spec stages
- Full repo indexing, multi-file edits, terminal access for validation
- The most architecturally aligned Copilot product for Architect OS

**Copilot coding agent (prototype/announced):**
- Fully autonomous: assign issue → branch → implement → PR
- Context: issue body + `.github/copilot-instructions.md` + codebase embeddings
- Self-review comment posted on PR
- **Not yet GA for Pro/Business tiers** — Enterprise or limited preview only

**Copilot Code Review (Enterprise):**
- Inline PR comments, bug/performance/style/security flagging
- Manual trigger ("Review with Copilot") or auto on PR open
- GPT-4o model, optimized for classification tasks

### Pricing

| Tier | Price | Key features |
|---|---|---|
| Copilot Free | $0 | 2K completions/mo, 50 chat messages/mo |
| Copilot Pro | $10/mo | Unlimited completions + chat, multi-model, @workspace |
| Copilot Business | $19/user/mo | Pro + org policies, IP indemnity |
| Copilot Enterprise | $39/user/mo (requires GHEC) | Business + Workspace, Code Review, knowledge bases |

**Source:** GitHub Copilot pricing page, mid-2025.

### Where it fits in Architect OS

| Stage | Copilot component | Role |
|---|---|---|
| S5 Plan | Copilot Workspace (Enterprise) | Interactive plan-first decomposition |
| S6 (XS) | Copilot Workspace / coding agent | Fire-and-forget for well-specified small tickets |
| S6 (exploration) | Copilot Chat @workspace | IDE exploration before coding |
| S6 (watching) | Copilot agent mode (VS Code Insiders) | IDE-native agent with human watching |
| S7 Review | Copilot Code Review (Enterprise) | Third-pass AI reviewer |

### The Enterprise gap for solo professionals

This is the most important practical finding: **Copilot Workspace, coding agent, and Code Review all require Enterprise ($39/mo + GitHub Enterprise Cloud) plus VS Code Insiders for agent mode.** On Pro ($10/mo) or Business ($19/mo), you get:

- Inline completions
- Copilot Chat with @workspace and @github
- Multi-model chat
- **No** Workspace, **no** coding agent, **no** Code Review

For a solo professional on Pro: Copilot is IDE support infrastructure, not a harness. Claude Code handles the heavy lifting.

### Reality check: the "Copilot coding agent" doesn't exist as a product

GitHub has demoed an autonomous Issue→PR agent but the publicly documented product is sparse and described as "coming soon." The closest operational equivalent today is **Copilot Workspace** (interactive, plan-first, human-gated) — which actually aligns better with Architect OS principles than a fully autonomous agent would.

**Source:** GitHub Copilot documentation, docs.github.com/copilot, mid-2025.

---

## Cursor

**Source:** cursor.com documentation and blog, mid-2025.

### What it is

A VS Code fork built from the ground up as an AI-first IDE. Unlike Copilot (which layers AI on top of VS Code), Cursor rewrites the editing experience around AI interaction.

### Key capabilities

- **Tab (completions):** Jump-to-next-suggestion completions that predict entire multi-line edits and cursor-position changes — not just text insertions.
- **Agent mode:** Terminal-based agent similar to Claude Code/Codex CLI, but embedded in the IDE. Plan → edit → test loop with terminal access.
- **Composer:** A chat interface that can apply edits directly to files. Simpler than agent mode for single-file changes.
- **.cursorrules:** Project-level instructions (analogous to CLAUDE.md), plus `.cursor/rules/` directory for domain-specific rules.
- **Model selection:** GPT-4o, Claude 3.5 Sonnet, Gemini 2.0 Flash, o1-preview. You choose per-feature (completions vs chat vs agent).
- **Context:** `@Codebase` for repo-wide RAG, `@Docs` for documentation indexing, `@Web` for live web search.
- **Inline editing:** Select code, Cmd+K, describe the change — Cursor edits in place without leaving the file.

### Pricing

| Tier | Price |
|---|---|
| Hobby | Free (limited usage) |
| Pro | $20/mo |
| Pro+ (with o1/o3) | $40/mo |
| Business | $40/user/mo |

**Source:** cursor.com/pricing, mid-2025.

### Cursor vs Claude Code — the architectural distinction

| Dimension | Cursor | Claude Code |
|---|---|---|
| Where you work | IDE — graphical editor with AI layered in | Terminal — text-only, filesystem-native |
| Primary interaction model | You type code, AI completes/assists | You specify, AI does most of the typing |
| Agent characteristics | IDE-embedded, you watch and intervene inline | Autonomous, runs in terminal, you review the diff |
| Context scope | @Codebase (RAG), @Docs, @Web, open files | Fresh session: AGENTS.md + ticket + planned files only |
| Audit trail | Diff in editor, less structured | Full transcript of every command and file edit |
| Learnability | Shallow — works like VS Code with AI features | Steeper — requires comfort with terminal and agent paradigms |

### Where it fits in Architect OS

Cursor is not a replacement for Claude Code in this OS. It's the **IDE layer** — where you do exploration, light edits, and review. The OS uses Claude Code as the harness for heavy work because Claude Code provides:

- Fresh, narrow context per ticket (Cursor's @Codebase is the opposite — broad context)
- Full transcripts as audit trails
- Programmatic skills that can be versioned and shared
- Deterministic context assembly (you specify exactly which files the agent sees)

Practical: use Cursor for reading code, small edits, and reviewing diffs in a visual editor. Use Claude Code for S5–S6 because narrow context + auditable sessions is the OS's quality mechanism. If you prefer an IDE feel and are willing to trade some control, Cursor agent mode can replace Claude Code for S6 on S/M tickets — but you lose the session transcript.

---

## CodeRabbit

**Source:** [docs.coderabbit.ai](https://docs.coderabbit.ai), CodeRabbit official documentation.

### What it is

An AI code review platform that integrates as a GitHub/GitLab/Bitbucket App. It's always-on — reviews every PR automatically, posts inline comments, responds to threaded replies, and re-reviews on new commits.

### Key capabilities

- **Multi-model pipeline:** Claude (Opus/Sonnet) for deep analysis, GPT-4o for cross-validation, fine-tuned models for summarization.
- **Summarization:** Walk-through summaries and release-notes-style changelogs automatically generated.
- **Sequence diagrams:** Visual diff understanding generated from code changes.
- **Conversational:** Reply to CodeRabbit's comments and it responds. `@coderabbitai review`, `@coderabbitai summarize`, `@coderabbitai skip`.
- **`.coderabbit.yaml`:** Repo-level config file in version control — review profile, path filters, ignore patterns, tone instructions, knowledge base sources.
- **Incremental review:** Only reviews new commits on a PR, not the full diff each push.
- **Auto-labeling:** `size/M`, `security`, `breaking-change` labels applied automatically.

### Pricing

| Tier | Price |
|---|---|
| Free | $0 — public repos only, limited reviews |
| Pro | ~$12–15/user/month — unlimited public + private |
| Team | ~$20–24/user/month — analytics, knowledge base |
| Enterprise | Custom — SSO, audit logs, on-prem, SLA |

**Source:** docs.coderabbit.ai, mid-2025.

### Where it fits in Architect OS

| Stage | Role |
|---|---|
| S7 Review | Primary AI reviewer — second pass after human rubric review |

CodeRabbit runs automatically when a PR opens. The human reviews first (rubric), then reads CodeRabbit's findings, then optionally a second AI opinion (Codex review or claude-code-action).

### Recommended .coderabbit.yaml for Architect OS

```yaml
language: "typescript"
reviews:
  profile: "assertive"
  max_files: 30
  request_changes_workflow: false
  auto_review:
    enabled: true
    drafts: false
    auto_incremental_review: true
    ignore_title_patterns:
      - "WIP:"
      - "[skip review]"
  path_filters:
    exclude:
      - "**/*.test.ts"
      - "**/*.spec.ts"
      - "**/__snapshots__/**"
      - "docs/**"
      - "generated/**"
      - "memory/dumps/**"
  high_level_summary: |
    Review priorities (in order):
    1. Constitution C-rule violations — flag by rule ID (C1–C35)
    2. Security vulnerabilities and unsafe patterns
    3. Missing error handling / null safety
    4. Test coverage gaps in new logic
    5. API contract violations vs. the FSD spec
    DO NOT flag: style/formatting (Prettier/ESLint in CI), variable naming preferences, "extract to function" for ≤10 line blocks.
    PRs are capped at 400 lines — flag violations.

tone_instructions: |
  Concise. One finding = one comment. Include the specific constitution rule ID if applicable.
  For bugs: show the failing case. For suggestions: show the fix as a code diff.

knowledge_base:
  sources:
    - "AGENTS.md"
    - "docs/architecture/architecture.md"
  learnings:
    - "All API routes require Zod validation (C12)"
    - "TDD required: tests before implementation (C4)"
    - "No new dependencies without an ADR line (C8)"
```

### CodeRabbit vs other AI reviewers

| Dimension | CodeRabbit | Copilot Code Review | Codex CLI (`codex review`) |
|---|---|---|---|
| Availability | All repos (free for public) | Enterprise only ($39/mo) | API — pay per review |
| Setup | Click-install GitHub App | Built into GitHub (Enterprise) | CLI tool + API key |
| Review depth | Deep, multi-model | Good, single-model (GPT-4o) | Good, focused |
| Summarization | ✅ Excellent | ❌ None | ❌ None |
| Conversational | ✅ Thread replies | ⚠️ Limited | ❌ One-shot |
| Config file | `.coderabbit.yaml` (repo) | UI settings | CLI flags |
| Best for | "Set and forget" always-on review | Teams already on Copilot Enterprise | Quick second opinion |

For solo professionals on any tier: CodeRabbit is free for public repos, Pro for private. This is the most accessible AI reviewer.

---

## Traycer

**Source:** traycer.ai (official site), cross-referenced with Architect OS references.

### What it is

A managed, product-form alternative for the **S5 ticket decomposition** step. Positioned as the same outcome as running `to-tickets` in Claude Code, but through a hosted product UI rather than a terminal session.

### Key capabilities (inferred from Architect OS positioning)

- Reads an approved FSD + architecture docs
- Reads the actual repo to name real files in ticket plans
- Produces GitHub Issues with file/function-level implementation plans
- Manages dependency ordering between tickets
- Likely includes team collaboration features, saved project context, and Linear/Jira integration

### Where it fits in Architect OS

| Stage | Role |
|---|---|
| S5 Plan | Alternative harness — replaces Claude Code + `to-tickets` for ticket decomposition |

The human gate (plan review) is the same regardless of tool. Traycer is the product alternative when you want a managed UI vs a CLI session.

### Traycer vs Claude Code + `to-tickets`

| Factor | Claude Code + `to-tickets` | Traycer |
|---|---|---|
| How it works | Raw Claude Code session, reads live repo files | Hosted product with UI, repo connection |
| Context quality | Reads actual files from live repo | Unknown — likely indexed snapshots |
| Human gate | You review every plan (same) | You review every plan (same) |
| Integration | GitHub Issues via `gh` CLI | GitHub/GitLab/Jira/Linear (inferred) |
| Pricing | Claude Code subscription + API usage | Unknown (contact sales) |
| Best for | Solo professionals who want full control | Teams wanting managed UX |

### Verdict for Architect OS

Traycer is a valid alternative at S5. The OS defaults to Claude Code because it reads live files (the lifecycle explicitly says "it must read actual code to name actual files") and has no additional subscription. If you find ticket decomposition is your bottleneck and prefer a product UX, evaluate Traycer — but verify it reads live repo files, not stale snapshots.

---

## Spec Kit (GitHub)

**Source:** [github.com/github/spec-kit](https://github.com/github/spec-kit), [GitHub Blog Sept 2025](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/).

### What it is

GitHub's open-source CLI toolkit for spec-driven development. Installs as `specify` CLI + slash commands (`/speckit.*`) into your coding agent of choice. Works with Claude Code, Copilot, Gemini CLI, and 30+ other agents.

### Key capabilities

The four-phase workflow:

1. **`/speckit.specify`** — high-level requirements → detailed spec (user journeys, what/why, not tech stack)
2. **`/speckit.plan`** — spec + tech stack/constraints → technical implementation plan
3. **`/speckit.tasks`** — plan → actionable, dependency-ordered task list
4. **`/speckit.implement`** — tasks → working code, one task at a time

Plus optional commands:

- `/speckit.constitution` — project governing principles
- `/speckit.clarify` — clarify underspecified areas before planning
- `/speckit.analyze` — cross-artifact consistency analysis
- `/speckit.checklist` — quality checklists (like "unit tests for English")
- `/speckit.taskstoissues` — convert tasks to GitHub Issues
- `/speckit.converge` — assess codebase against spec/plan, append remaining work

### Customization

- **Extensions:** Add new commands and workflows (e.g., Jira integration, V-Model traceability, post-implementation review)
- **Presets:** Override templates and terminology (e.g., compliance-oriented spec format, Agile vs Waterfall, localized workflow)
- **Bundles:** Role-based curated sets (product manager bundle, security researcher bundle, developer bundle)
- **Templates:** Resolved at runtime with priority: project-local overrides > presets > extensions > core defaults

### Pricing

Free and open-source (MIT license). No paid tier. The `specify` CLI is a Python tool installed via `uv` or `pipx`.

### Spec Kit vs Architect OS — how they relate

Architect OS and Spec Kit are the same philosophy implemented at different levels:

| Dimension | Spec Kit | Architect OS |
|---|---|---|
| Scope | Spec → Plan → Tasks → Implement | Idea → BRD → PRD → FSD → Design → Architecture → Plan → Implement → Review → Release → Learn |
| Specification format | Spec Kit's own `.specify/` templates | Full BRD, PRD, FSD, ADR with domain modeling |
| Human gates | At spec and plan phases | At every stage, with explicit exit criteria |
| Review system | None (implementation is the end) | Two-stage: human rubric + AI review |
| Memory system | `.specify/memory/` directory | Four-layer: AGENTS.md → docs → graph → dumps |
| Skills integration | Extensions/presets/community bundles | Full Pocock skills + OS-native skills + skill versioning |
| Execution system | Tasks list (markdown) | GitHub Issues + Projects + rulesets |
| Agent harness | Agent-agnostic (supports 30+) | Opinionated: Claude Code primary, others mapped per stage |

Architect OS is the complete lifecycle. Spec Kit is the spec→implement vertical slice. They're complementary: you could use Spec Kit's `/speckit.*` commands as the tooling inside Architect OS's S2–S5 stages. The key difference is that Architect OS adds the front half (business framing, design, architecture) and back half (review, release, learning), plus the memory system that makes every cycle smarter.

### Where it fits in Architect OS

| Stage | Spec Kit mapping |
|---|---|
| S2 Specify | `/speckit.specify` → generates spec from high-level description |
| S4 Architect | `/speckit.plan` → generates technical plan from spec + constraints |
| S5 Plan | `/speckit.tasks` + `/speckit.taskstoissues` → decomposes into issues |
| S6 Implement | `/speckit.implement` → executes tasks |

If you want less ceremony than the full Architect OS lifecycle, Spec Kit alone is a solid lightweight alternative. If you want the full system, Spec Kit commands can be embedded as tools within the OS stages.

---

## BMAD (bmad.dev)

**Source:** bmad.dev (official site, limited public documentation).

### What it is

A persona-based AI development methodology. BMAD structures the development process around **role separation**: distinct personas (analyst → PM → architect → developer) with artifact handoffs and human sign-off between each. Story files carry full context through the chain.

### Key characteristics

- **Persona-based:** Each role has a dedicated AI persona with specific instructions and constraints
- **Artifact chain:** Business analysis document → product requirements → architecture decisions → story files → implementation
- **Context preservation:** Story files carry full context from planning all the way to dev — the agent implementing a story has the complete upstream rationale
- **Heavy ceremony:** More process weight than Architect OS or Spec Kit. Suitable for regulated environments or large teams with role specialization.

### BMAD vs Architect OS

| Dimension | BMAD | Architect OS |
|---|---|---|
| Role separation | Enforced — different personas, human sign-off between each | Optional — heavyweight profile adds BMAD-style separation |
| Process weight | Heavy — designed for enterprise teams | Modular — three profiles (light/default/heavy) |
| Agent model | Persona files + story files as context carriers | Narrow context per ticket, memory in files and graph |
| Best for | Regulated industries, large teams, compliance-heavy work | Solo professionals, small teams, startup velocity |

The Architect OS heavyweight profile explicitly mentions BMAD-style role separation as an option. The OS's default profile is lighter — one architect role (you) across all stages, with agents executing between gates.

---

## The One-Glance Matrix

| Layer | Default | Alternative | Skip when |
|---|---|---|---|
| S0 Capture | None / Claude chat | — | — |
| S1 Frame (BRD) | Claude Code | — | Lightweight profile: skip BRD |
| S2 Specify (PRD/FSD) | Claude Code (frontier model) | Spec Kit `/speckit.specify` | Lightweight: acceptance criteria in issue body |
| S3 Design | Claude Code + Figma MCP | v0, cursor Composer | Headless work; lightweight profile |
| S4 Architect | Claude Code (plan mode) | Spec Kit `/speckit.plan` | — |
| S5 Plan (tickets) | Claude Code + `to-tickets` | Traycer, Spec Kit `/speckit.tasks` | Lightweight: plan in issue body |
| S6 Implement | Claude Code (fresh session per ticket) | Codex CLI (2nd opinion), Copilot Workspace (Enterprise) | — |
| S6 (XS async) | — (manual launch) | Copilot coding agent (Enterprise), Codex Cloud | Lightweight: manual is fine for XS |
| S7 Review (AI) | CodeRabbit | Codex review, Copilot Code Review (Enterprise) | Lightweight: CodeRabbit only |
| S7 Review (human) | You + rubric | — | Never skip |
| S8 Release | CI/CD (Vercel/Cloudflare) | — | — |
| S9 Learn | Claude Code (cheap model) | — | Lightweight: skip graph update, just dump |

---

## Setup checklist (from this matrix)

- [ ] Claude Code installed, signed in on Max plan ($200/mo recommended for heavy usage; Pro at $20/mo works for light)
- [ ] Codex CLI installed (`npm install -g @openai/codex`) for second-opinion runs
- [ ] CodeRabbit GitHub App installed on all repos
- [ ] `gh` CLI authenticated
- [ ] Optional: Spec Kit CLI installed (`uv tool install specify-cli`) if you want the `/speckit.*` commands alongside Architect OS stages
- [ ] Optional: Cursor installed for IDE-level exploration and review
- [ ] Optional: Copilot Pro ($10/mo) for inline completions and IDE Chat
- [ ] Optional: Copilot Enterprise ($39/mo + GHEC) if you need Workspace, coding agent, and Code Review

### Model routing defaults

| Task | Model | Rationale |
|---|---|---|
| Spec generation (S2) | Claude Opus 4.x | Highest reasoning quality — not the stage to save money |
| Architecture decisions (S4) | Claude Opus 4.x or o3 | Deep trade-off analysis |
| Ticket decomposition (S5) | Claude Sonnet 4.x | Reads real files, produces structured plans |
| Implementation (S6) | Claude Sonnet 4.x (primary), GPT-5.2 (alternative) | Sonnet is crisp and fast; GPT-5.2 for broader language coverage |
| Self-review (S6) | Claude Sonnet 4.x | Same model catches its own patterns |
| Learning/dumps (S9) | Claude Haiku 4.5 or GPT-5 mini | Mechanical distillation, cheapest viable model |

**Source for model routing:** Anthropic and OpenAI official documentation on model capabilities; pricing verified against docs.x.ai, platform.openai.com, ai.google.dev, docs.anthropic.com.
