# Harness Matrix — Which Tool, Which Layer, and Why

_Every tool comparison in one place, cited to primary sources. Rewritten July
2026 around the **open-frontier + OpenRouter** thesis: intelligent models for
planning, cheap flash models for coding, different model families for review.
The question isn't "which tool is best" — it's "which model for which job at
which stage, at what cost."_

---

## The shape of the market (July 2026)

The market reorganized around two axes: **who runs the agent** (harness) and
**which model powers it** (frontier). The mid-2025 matrix conflated these. A CLI
agent is now a _harness_ — the model behind it is a _routing decision_. The
Architect OS uses **open-frontier models via OpenRouter / direct APIs** as the
primary power source, with **OpenCode / Aider / Cline** as the harnesses, and
**CodeRabbit + OpenRouter-routed reviewers** as the review layer.

| Category                     | Example tools                                                                  | What they do                                                                                                                             | Architecture                                         |
| ---------------------------- | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| **CLI agents (BYO-key)**     | OpenCode, Aider, Cline, Continue                                               | Full agent in terminal/IDE — plan, edit, test, self-correct. Accept any OpenAI-compatible endpoint, OpenRouter, or direct provider APIs. | Fresh session per run, file-grounded, model-agnostic |
| **Vendor-locked CLI agents** | Claude Code, Codex CLI                                                         | Terminal agents tied to vendor models (Anthropic / OpenAI). Claude Code now accepts DeepSeek via its Anthropic-compatible API.           | Vendor-locked but widening                           |
| **IDE extensions**           | Copilot Chat, Cursor, Cline (VS Code)                                          | AI inside your editor                                                                                                                    | Session-bound, synchronous                           |
| **Cloud/platform agents**    | Copilot Workspace, Copilot coding agent, Devin, CodeRabbit, claude-code-action | Hosted services triggered by platform events                                                                                             | Fire-and-forget or interactive web UX                |

**The Architect OS thesis (updated):** use **BYO-key CLI agents** as the primary
harness (model-agnostic, reads real files, full audit trail, no vendor lock-in),
route **intelligent open models** to planning stages, **cheap flash models** to
implementation, and **different model families** to review (uncorrelated
auditor). Vendor-locked agents are optional escape hatches, not the default.

---

## The cost-quality frontier (July 2026)

The single most important change since mid-2025: open-frontier models closed the
quality gap with Anthropic/OpenAI while staying 10–40× cheaper. The routing
problem is no longer "which vendor" — it's "which tier at which stage."

### The open-frontier model landscape

| Model                         | Input $/1M | Output $/1M | Context | Best for                                                     | Notes                                                                                                                 |
| ----------------------------- | ---------- | ----------- | ------- | ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| **DeepSeek V4 Pro**           | $0.435     | $0.87       | 1M      | Planning, spec generation, architecture, reasoning           | Thinking mode, tool calls, JSON. Anthropic-compatible API at `api.deepseek.com/anthropic`. 28× cheaper than Opus 4.8. |
| **DeepSeek V4 Flash**         | $0.14      | $0.28       | 1M      | Implementation, coding, self-review                          | Thinking + non-thinking modes, tool calls, FIM. 35× cheaper than Sonnet 5. Cache hit input: $0.0028/1M.               |
| **GLM 5.2** (Zhipu)           | ~$0.50     | ~$2.00      | 200k    | Code review, general reasoning, uncorrelated second opinion  | Open frontier. Strong on Chinese + English. Good reviewer because different training corpus from DeepSeek.            |
| **GLM 4.5 Air**               | ~$0.10     | ~$0.40      | 128k    | Cheap implementation, mechanical edits                       | Flash-tier. Extremely cheap.                                                                                          |
| **Qwen 3 Coder** (Alibaba)    | ~$0.20     | ~$0.60      | 256k    | Code generation, completion, refactoring                     | Open weights. Strong on code. Available via OpenRouter and Alibaba Cloud.                                             |
| **Qwen 3 235B**               | ~$0.15     | ~$0.60      | 256k    | General reasoning, planning fallback                         | Open weights, self-hostable.                                                                                          |
| **Llama 4 Maverick** (Meta)   | ~$0.20     | ~$0.60      | 1M      | Long-context review, uncorrelated auditor                    | Open weights. 1M context. Via OpenRouter or self-hosted.                                                              |
| **Kimi K2** (Moonshot)        | ~$0.60     | ~$2.50      | 256k    | Reasoning, planning alternative to DeepSeek V4 Pro           | Agentic-strong. Open weights.                                                                                         |
| **Gemini 2.5 Flash** (Google) | $0.075     | $0.30       | 1M      | Mechanical work, dumps, classification, cheap implementation | Cheapest viable model. 1M context.                                                                                    |
| **Gemini 2.5 Pro** (Google)   | $1.25      | $10.00      | 2M      | Long-context planning, massive repo analysis                 | 2M context. Mid-tier price.                                                                                           |
| **Grok 4** (xAI)              | ~$3.00     | ~$15.00     | 256k    | Reasoning alternative, X/Twitter data access                 | Mid-tier. Optional.                                                                                                   |

### The vendor frontier (for comparison, used as escape hatches)

| Model           | Input $/1M    | Output $/1M | When to use                                                                                     |
| --------------- | ------------- | ----------- | ----------------------------------------------------------------------------------------------- |
| Claude Opus 4.8 | $5.00         | $25.00      | Only when DeepSeek V4 Pro fails a planning task — highest reasoning ceiling, 11× more expensive |
| Claude Sonnet 5 | $2.00 (intro) | $10.00      | Only when DeepSeek V4 Flash produces low-quality code on a hard ticket — 35× more expensive     |
| GPT-5.6-sol     | $5.00         | $30.00      | Escape hatch for OpenAI-ecosystem tasks                                                         |

**Source:** DeepSeek pricing verified at
api-docs.deepseek.com/quick_start/pricing (fetched 2026-07-19). Other pricing
from OpenRouter, provider pricing pages, and `research-2026-07-update.md` in
this directory.

### The routing principle

> **Intelligent models for planning (where mistakes are expensive), cheap flash
> models for coding (where iterations are cheap), different model families for
> review (where correlation is the enemy).**

Planning mistakes cost 10× the diff they produce. Coding mistakes cost 1× the
diff. Review mistakes from correlated models cost the bug rate of the author's
blind spots. Route accordingly.

---

## OpenRouter — the routing layer

**Source:** [openrouter.ai/docs](https://openrouter.ai/docs/quickstart),
`research-2026-07-update.md`.

### What it is

A unified API across 400+ models from all major providers and the open-frontier
ecosystem. One API key, one billing surface, automatic fallbacks, and an MCP
server at `https://mcp.openrouter.ai/mcp` that lets any coding agent pull live
model and pricing data.

### Key capabilities for Architect OS

- **Unified API:** `OPENAI_BASE_URL=https://openrouter.ai/api/v1` works with
  OpenCode, Aider, Cline, Continue, and any OpenAI-compatible client. One key,
  all models.
- **Latest aliases:** `~openai/gpt-latest`, `~anthropic/claude-sonnet-latest`,
  `~deepseek/deepseek-latest` — always resolve to newest version. No model-name
  churn.
- **Automatic fallbacks:** if DeepSeek is down, fall back to GLM 5.2 → Qwen 3 →
  Llama 4. Configure per-route.
- **Provider routing:** some models have multiple providers (e.g., Llama 4 via
  Meta, Together, Groq). OpenRouter picks cheapest/fastest by default; you can
  pin.
- **MCP server:** `https://mcp.openrouter.ai/mcp` — agents can query live
  pricing, model capabilities, and routing options at runtime.
- **Agent SDK:** `@openrouter/agent` for tool-use loops.
- **Pricing:** OpenRouter adds a small markup (typically 5–20%) over provider
  pricing. For the cheapest models this is negligible; for frontier models,
  compare against direct API.

### Pricing

Pay-as-you-go, credits. No subscription tier. You fund the account and spend
against any model. Free tier exists for some models (Llama, Gemini Flash via
Google).

### Where it fits in Architect OS

OpenRouter is the **model routing layer** — it sits between the harness
(OpenCode/Aider/Cline) and the models. The harness doesn't know or care which
model it's talking to; OpenRouter handles the routing.

| Stage                   | OpenRouter route                             | Why                                                     |
| ----------------------- | -------------------------------------------- | ------------------------------------------------------- |
| S2 Specify              | `deepseek/deepseek-v4-pro`                   | Intelligent planning, cheap                             |
| S4 Architect            | `deepseek/deepseek-v4-pro`                   | Deep reasoning, cheap                                   |
| S5 Plan                 | `deepseek/deepseek-v4-pro`                   | Reads real files, structured plans                      |
| S6 Implement            | `deepseek/deepseek-v4-flash`                 | Fast, cheap coding                                      |
| S6 (hard ticket)        | `zhipuai/glm-4.5-air` or `qwen/qwen-3-coder` | Fallback if Flash struggles                             |
| S7 Review (2nd opinion) | `zhipuai/glm-5.2`                            | **Different family from author** — uncorrelated auditor |
| S7 Review (3rd opinion) | `meta-llama/llama-4-maverick`                | Third family, long context                              |
| S9 Learn                | `google/gemini-2.5-flash`                    | Cheapest viable, mechanical                             |

---

## OpenCode — primary harness

**Source:** opencode.ai documentation, `research-2026-07-tools.md`.

### What it is

A terminal-based, model-agnostic coding agent. Reads files, writes code, runs
commands, tests its own work, self-corrects in a loop. Accepts any
OpenAI-compatible API, OpenRouter, Anthropic, OpenAI, Google, or direct provider
endpoints. The harness the Architect OS is built on.

### Key capabilities

- **Model-agnostic:** configure via `opencode.json` with any provider. DeepSeek,
  GLM, Qwen, Llama, Gemini, Claude, GPT — all work. Switch models per-stage via
  config.
- **Subagents:** `task` tool dispatches parallel subagents with `subagent_type`
  (general, explore). Each runs in isolated context. Native multi-agent.
- **Skills:** `.opencode/skills/` directory, same convention as Claude Code.
  Reusable workflows, model-invoked and user-invoked.
- **AGENTS.md / opencode.json:** project-level instructions + config consumed at
  session start. Model routing per-stage defined here.
- **Context isolation:** fresh session per ticket, narrow context. Same
  principle as Claude Code, no vendor lock.
- **MCP support:** connect any MCP server (Figma, Notion, GitHub, OpenRouter
  pricing).
- **Plan mode:** read-only exploration before writing code.
- **Hooks:** deterministic pre/post actions (lint on edit, format on commit).

### Pricing

Open source.
**$0 for the harness.** You pay only for model API usage (DeepSeek, OpenRouter, etc.). This is the core economic advantage over Claude Code ($200/mo
Max plan) and Codex ($20/mo ChatGPT).

### Where it fits in Architect OS

| Stage                | Role                                      | Model routed               |
| -------------------- | ----------------------------------------- | -------------------------- |
| S1 Frame (BRD)       | Primary — interactive grilling + drafting | DeepSeek V4 Pro            |
| S2 Specify (PRD/FSD) | Primary — spec creation + grill-with-docs | DeepSeek V4 Pro            |
| S3 Design            | Primary + Figma MCP                       | DeepSeek V4 Pro            |
| S4 Architect         | Primary — plan mode, ADR drafting         | DeepSeek V4 Pro            |
| S5 Plan (tickets)    | Primary — to-tickets + wayfinder          | DeepSeek V4 Pro            |
| S6 Implement         | Primary — one fresh session per ticket    | DeepSeek V4 Flash          |
| S7 Review            | Secondary — self-review pass              | GLM 5.2 (different family) |
| S9 Learn             | Primary — mechanical                      | Gemini 2.5 Flash           |

### Best for

Every stage. This is the default harness for Architect OS. Model-agnostic, no
vendor lock, no subscription, routes to the cheapest viable model per stage.

### Configuration sketch

```jsonc
// opencode.json
{
  "provider": {
    "deepseek": {
      "npm": "@ai-sdk/openai-compatible",
      "options": { "baseURL": "https://api.deepseek.com" },
      "models": {
        "deepseek-v4-pro": { "name": "deepseek-v4-pro" },
        "deepseek-v4-flash": { "name": "deepseek-v4-flash" }
      }
    },
    "openrouter": {
      "npm": "@ai-sdk/openai-compatible",
      "options": { "baseURL": "https://openrouter.ai/api/v1" },
      "models": {
        "glm-5.2": { "name": "zhipuai/glm-5.2" },
        "llama-4-maverick": { "name": "meta-llama/llama-4-maverick" },
        "gemini-2.5-flash": { "name": "google/gemini-2.5-flash" }
      }
    }
  },
  "model": "deepseek/deepseek-v4-flash"
}
```

---

## Aider — alternative CLI harness

**Source:** [docs.aider.ai](https://docs.aider.ai/),
`research-2026-07-memory.md`.

### What it is

A terminal-based pair-programming agent. Stronger than OpenCode on one axis:
**automatic repo-map construction** via tree-sitter + PageRank-style symbol
ranking. Weaker on others: less mature subagent/multi-agent story.

### Key capabilities

- **Repo map:** auto-builds a tree-sitter knowledge graph of the repo, ranked by
  import/dependency structure, sized to a token budget. This is the
  `repo-graph.json` Layer 3 from `repo-memory.md` — Aider builds it
  automatically, architect-os hand-curates it.
- **OpenRouter native:** `--model openrouter/zhipuai/glm-5.2` just works. Deep,
  first-class support.
- **Multi-model:**
  `--model-deepseek deepseek-v4-pro --model-openrouter zhipuai/glm-5.2` — use
  different models for different sub-tasks in one session.
- **Architect mode:** separate "architect" model (planning) and "editor" model
  (implementation) in one run. Maps directly to the Architect OS routing
  principle.
- **Git-native:** every edit is a commit. Full audit trail by construction.

### Pricing

Open source. **$0 for the harness.** Pay only for model API usage.

### Where it fits in Architect OS

| Stage        | Role                                         | Why                                                           |
| ------------ | -------------------------------------------- | ------------------------------------------------------------- |
| S6 Implement | Alternative to OpenCode                      | Repo map gives better symbol-level context on large repos     |
| S4 Architect | Alternative — repo map surfaces architecture | Auto-built graph is the "improve-codebase-architecture" input |

### Aider architect mode = the routing principle, built in

```bash
aider --architect-model deepseek/deepseek-v4-pro \
      --model deepseek/deepseek-v4-flash \
      --editor-model deepseek/deepseek-v4-flash
```

The architect model plans (Pro), the editor model implements (Flash). This is
the Architect OS S5→S6 split in one command.

---

## Cline — IDE-native harness

**Source:** cline.bot (formerly Claude Dev), `research-2026-07-tools.md`.

### What it is

A VS Code extension that turns the editor into an agent surface. Reads files,
writes code, runs terminal commands, MCP support. Model-agnostic via OpenRouter
and direct provider APIs.

### Key capabilities

- **VS Code-native:** works in the editor you already use. No terminal context
  switch.
- **OpenRouter native:** first-class OpenRouter support. Route to any model.
- **MCP support:** connect Figma, GitHub, Notion, databases.
- **`@`-mentions:** reference files, folders, URLs in prompts.
- **Plan vs act mode:** plan (read-only) → approve → act (write). Same gate
  structure as OpenCode plan mode.

### Pricing

Open source. **$0 for the harness.** Pay only for model API usage. VS Code
extension marketplace install.

### Where it fits in Architect OS

Cline is the **IDE layer** — where you do exploration, light edits, and review
diffs visually. Use OpenCode for S5–S6 heavy work (narrow context, auditable
sessions). Use Cline for reading code, small edits, and visual review. Same
division as the old Cursor recommendation, but model-agnostic.

---

## DeepSeek — the primary model provider

**Source:**
[api-docs.deepseek.com](https://api-docs.deepseek.com/quick_start/pricing),
verified 2026-07-19.

### The two models

| Model                 | Input (cache miss) | Input (cache hit) | Output   | Context | Concurrency |
| --------------------- | ------------------ | ----------------- | -------- | ------- | ----------- |
| **deepseek-v4-flash** | $0.14/1M           | $0.0028/1M        | $0.28/1M | 1M      | 2500        |
| **deepseek-v4-pro**   | $0.435/1M          | $0.003625/1M      | $0.87/1M | 1M      | 500         |

### Key capabilities

- **Thinking mode:** both models support thinking (default) and non-thinking
  modes. Thinking mode = extended reasoning before output. Toggle per-request.
- **Tool calls:** full function calling. Agents can use tools.
- **JSON output:** structured output for spec generation, ticket decomposition.
- **FIM (Fill-in-Middle):** code completion via prefix-suffix-middle. Fast
  inline completions.
- **Context caching:** automatic KV cache. Cache hit input is **50× cheaper**
  than cache miss ($0.0028 vs $0.14). Keep AGENTS.md and spec sections stable to
  maximize cache hits.
- **Anthropic-compatible API:** `https://api.deepseek.com/anthropic` — Claude
  Code, claude-code-action, and any Anthropic-ecosystem tool can use DeepSeek as
  a backend. **This is the correlated-vendor escape hatch:** run the Claude Code
  harness with DeepSeek models, getting Anthropic's agent loop + DeepSeek's
  uncorrelated model.
- **1M context window:** fits entire repos or long specs. No need for aggressive
  context engineering on most tickets.

### Pricing comparison vs vendor frontier

| Task                                          | DeepSeek V4 Pro              | Claude Opus 4.8         | Multiple        |
| --------------------------------------------- | ---------------------------- | ----------------------- | --------------- |
| Spec generation (S2, ~50k input, ~10k output) | $0.022 + $8.70 = **$8.72**   | $0.25 + $250 = **$250** | **28× cheaper** |
| Architecture (S4, ~100k input, ~20k output)   | $0.044 + $17.40 = **$17.44** | $0.50 + $500 = **$500** | **28× cheaper** |

| Task                                        | DeepSeek V4 Flash          | Claude Sonnet 5       | Multiple        |
| ------------------------------------------- | -------------------------- | --------------------- | --------------- |
| Implementation (S6, ~30k input, ~5k output) | $0.004 + $1.40 = **$1.40** | $0.06 + $50 = **$50** | **35× cheaper** |
| Self-review (S7, ~20k input, ~3k output)    | $0.003 + $0.84 = **$0.84** | $0.04 + $30 = **$30** | **35× cheaper** |

**Monthly estimate (15–20 tickets):** DeepSeek API spend ~$15–40/mo vs Claude
Max $200/mo. OpenRouter for review adds ~$5–10/mo. **Total:
~$20–50/mo vs ~$232–249/mo.** 80–90% cost reduction at near-equivalent quality
for most tasks.

### Where it fits in Architect OS

DeepSeek is the **primary model provider**. V4 Pro for all planning stages (S1,
S2, S4, S5). V4 Flash for all implementation (S6) and self-review. The
Anthropic-compatible API means you can also use it as a backend for Claude Code
and claude-code-action when you want the Anthropic agent loop without the
Anthropic price.

---

## Open frontier models for review (GLM, Qwen, Llama, Gemini)

**Source:** `research-2026-07-review.md`, OpenRouter model catalog, provider
pricing pages.

### The reviewer-independence principle

From `insights-and-issues-2026-07.md` issue I2 and the Greptile auditor post:
**the AI reviewer must not share model family with the author.** If DeepSeek
authored the PR, a DeepSeek reviewer shares the same blind spots. Route review
to a _different family_ — GLM, Llama, Qwen, or Gemini.

### GLM 5.2 (Zhipu) — primary review model

- **Why:** different training corpus from DeepSeek (Chinese + English academic,
  different web crawl). Strong reasoning. Catches different bugs.
- **Cost:** ~$0.50/$2.00 per 1M. A typical review (20k input, 3k output) costs
  ~$0.16.
- **Via:** OpenRouter (`zhipuai/glm-5.2`) or Zhipu direct API.
- **Where:** S7 second-opinion review, S4 architecture debate (council pattern).

### Llama 4 Maverick (Meta) — long-context review

- **Why:** 1M context, open weights, can self-host. Different family from both
  DeepSeek and GLM. Third opinion when review is contentious.
- **Cost:** ~$0.20/$0.60 per 1M via OpenRouter. Cheaper than GLM.
- **Where:** S7 third opinion, large-PR review (1M context fits the whole diff +
  surrounding code).

### Qwen 3 Coder (Alibaba) — code-specialized review

- **Why:** trained heavily on code. Strong at catching code-specific bugs (type
  mismatches, API misuse, edge cases). Different family.
- **Cost:** ~$0.20/$0.60 per 1M.
- **Where:** S7 code-specific bug hunting, S6 fallback implementation when
  DeepSeek Flash struggles.

### Gemini 2.5 Flash (Google) — mechanical and dump

- **Why:** $0.075/$0.30 per 1M. Cheapest viable model with 1M context. Different
  family.
- **Cost:** a memory dump (10k input, 2k output) costs ~$0.014.
- **Where:** S9 memory dumps, classification, mechanical distillation, S7 quick
  skim.

---

## CodeRabbit — always-on AI reviewer

**Source:** [docs.coderabbit.ai](https://docs.coderabbit.ai),
`research-2026-07-review.md`.

### What it is

An AI code review platform that integrates as a GitHub/GitLab/Bitbucket App.
Always-on — reviews every PR automatically, posts inline comments, responds to
threaded replies, re-reviews on new commits.

### Key capabilities (updated July 2026)

- **Multi-model pipeline:** Claude (Opus/Sonnet) for deep analysis, GPT-4o for
  cross-validation, fine-tuned models for summarization. **Note:** CodeRabbit
  uses vendor frontier models under the hood — this is the one place you're
  paying for premium models, but it's a subscription, not per-token.
- **Path Instructions** (`reviews.path_instructions` in `.coderabbit.yaml`) —
  glob-scoped instructions. The constitutional hook: each rule Cn can be
  expressed as a path instruction. CodeRabbit reports violations against the
  rule.
- **Learnings** — learns from PR chat conversations and auto-applies to future
  reviews. The loop that turns resolved comments into permanent rule updates.
- **`@coderabbitai emit path instructions`** — sweeps last 7 days of review
  feedback, opens a PR merging suggestions into `.coderabbit.yaml`. The
  automation hook for keeping the constitution current.
- **Change Stack** — reorganizes PR file list into structured layer-by-layer
  walkthrough. Gives the human a reading order for the 10-minute rubric.
- **Knowledge Base** — cross-repo analysis, MCP server connections.

### Pricing

| Tier | Price                                           |
| ---- | ----------------------------------------------- |
| Free | $0 — public repos only, limited reviews         |
| Pro  | ~$12–15/user/month — unlimited public + private |
| Team | ~$20–24/user/month — analytics, knowledge base  |

### Where it fits in Architect OS

| Stage     | Role                                                  |
| --------- | ----------------------------------------------------- |
| S7 Review | Primary always-on AI reviewer — runs on every PR open |

CodeRabbit runs automatically. The human reviews first (rubric), then reads
CodeRabbit's findings, then the OpenRouter-routed second opinion (GLM 5.2 or
Llama 4) runs via `gh` CLI or a GitHub Action.

### Recommended `.coderabbit.yaml` for Architect OS

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
  path_instructions:
    - path: "**/*.ts"
      instructions: |
        Review priorities (in order):
        1. Constitution C-rule violations — flag by rule ID (C1–C36)
        2. Security vulnerabilities and unsafe patterns
        3. Missing error handling / null safety
        4. Test coverage gaps in new logic
        5. API contract violations vs. the FSD spec
        DO NOT flag: style/formatting (Prettier/ESLint in CI),
        variable naming preferences, "extract to function" for ≤10 line blocks.
        PRs are capped at 400 lines — flag violations.

tone_instructions: |
  Concise. One finding = one comment. Include the specific constitution
  rule ID if applicable. For bugs: show the failing case.
  For suggestions: show the fix as a code diff.

knowledge_base:
  sources:
    - "AGENTS.md"
    - "docs/architecture/architecture.md"
  learnings:
    - "All API routes require Zod validation (C12)"
    - "TDD required: tests before implementation (C4)"
    - "No new dependencies without an ADR line (C8)"
    - "Reviewer must not share model family with author (C36)"
```

---

## claude-code-action — for uncorrelated review via Bedrock/Vertex

**Source:**
[github.com/anthropics/claude-code-action](https://github.com/anthropics/claude-code-action),
`research-2026-07-review.md`.

### What it is

Anthropic's GitHub Action for running Claude Code on PRs and issues. Runs on
your own runner. Supports Anthropic direct, AWS Bedrock, Google Vertex AI,
Microsoft Foundry.

### The key trick: non-Claude models via Bedrock/Vertex

claude-code-action supports Bedrock and Vertex as backends. Bedrock hosts
non-Claude models (Llama, Mistral, Nova). **Run claude-code-action against
Bedrock with a Llama 4 or Nova model** — you get the Claude Code agent loop
(which is excellent for review) but with a non-Claude model. This is the
cheapest way to get an uncorrelated auditor with a mature agent harness.

Alternatively, use the **DeepSeek Anthropic-compatible API**
(`api.deepseek.com/anthropic`) as the backend for claude-code-action. Claude
Code's agent loop runs, but DeepSeek V4 Pro powers the reasoning. Completely
uncorrelated from a Claude-authored PR.

### Where it fits in Architect OS

| Stage                   | Role                                    | Model                    |
| ----------------------- | --------------------------------------- | ------------------------ |
| S7 Review (2nd opinion) | Uncorrelated auditor via Bedrock        | Llama 4 Maverick or Nova |
| S7 Review (alternative) | Uncorrelated via DeepSeek Anthropic API | DeepSeek V4 Pro          |

### Pricing

Free (open source GitHub Action). You pay for the model API usage. Bedrock Llama
4: ~$0.20/$0.60 per 1M. DeepSeek V4 Pro via Anthropic API: $0.435/$0.87 per 1M.

---

## Vendor-locked agents — optional escape hatches

### Claude Code (optional, for when you need the absolute best agent loop)

**Source:**
[docs.anthropic.com/en/docs/claude-code](https://docs.anthropic.com/en/docs/claude-code/overview),
`research-2026-07-tools.md`.

Claude Code is still the most mature agent harness — subagents, skills, hooks,
plan mode, agent teams, Claude Code on the web. But it's vendor-locked to
Anthropic models by default.

The DeepSeek escape hatch: Claude Code can use DeepSeek via the
Anthropic-compatible API (`https://api.deepseek.com/anthropic`). Configure
`ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic` and
`ANTHROPIC_API_KEY=<deepseek-key>`. Claude Code's agent loop runs unchanged, but
DeepSeek V4 powers the reasoning. **You get Anthropic's harness quality at
DeepSeek's price.**

| Plan                            | Price                              | What you get                      |
| ------------------------------- | ---------------------------------- | --------------------------------- |
| Claude Pro                      | $20/mo                             | Limited Claude Code, rate-limited |
| Claude Max (20×)                | $200/mo                            | Heavy usage                       |
| Claude Max + API                | $200/mo + API usage                | BYO key for unlimited             |
| **DeepSeek-backed Claude Code** | **$0 + DeepSeek API (~$15–40/mo)** | **Same harness, 80–90% cheaper**  |

**Where it fits in Architect OS:** Optional. Use Claude Code + DeepSeek backend
when you need Anthropic's specific agent loop features (a particular skill, a
specific MCP integration) that OpenCode doesn't yet support. For 90% of work,
OpenCode + DeepSeek direct is sufficient and cheaper.

### OpenAI Codex CLI & Codex Web (optional second opinion)

**Source:** [github.com/openai/codex](https://github.com/openai/codex),
`research-2026-07-tools.md`.

OpenAI's terminal agent, now Rust-based (99.5k stars, v0.144.6 Jul 18 2026).
Four surfaces: Codex CLI, Codex IDE (VS Code/Cursor/Windsurf), Codex Web
(chatgpt.com/codex), Codex App. Bundled with ChatGPT
Plus/Pro/Business/Enterprise. Supports `AGENTS.md` cross-tool convention.

**Where it fits in Architect OS:** Optional third implementation opinion when
both DeepSeek Flash and GLM Flash struggle on a hard ticket. Codex Web is async
fire-and-forget alternative. Pricing bundled with ChatGPT subscription
($20–200/mo) — use only if you already pay for ChatGPT.

---

## GitHub Copilot — IDE support layer

**Source:** [docs.github.com/copilot](https://docs.github.com/copilot),
`research-2026-07-tools.md`.

### What it is

GitHub's layered AI platform: Copilot IDE (completions + Chat + agent mode),
Copilot Workspace (Issue → Plan → Spec → PR), Copilot coding agent (async Issue
→ PR), Copilot Code Review (now at Business tier, not just Enterprise).

### Key update July 2026

**Copilot Code Review is now available at Business tier ($19/user/mo), not just
Enterprise.** This changes the math: an uncorrelated OpenAI-family reviewer is
now accessible without Enterprise seats.

### Pricing

| Tier       | Price       | Key features                                          |
| ---------- | ----------- | ----------------------------------------------------- |
| Free       | $0          | 2K completions/mo, 50 chat messages/mo                |
| Pro        | $10/mo      | Unlimited completions + chat, multi-model, @workspace |
| Business   | $19/user/mo | Pro + Code Review (new), org policies, IP indemnity   |
| Enterprise | $39/user/mo | Business + Workspace, coding agent, knowledge bases   |

### Where it fits in Architect OS

| Stage                   | Copilot component              | Role                                                                |
| ----------------------- | ------------------------------ | ------------------------------------------------------------------- |
| S6 (exploration)        | Copilot Chat @workspace (Pro)  | IDE exploration before coding                                       |
| S6 (IDE agent)          | Copilot agent mode (Pro)       | IDE-native agent with human watching                                |
| S7 Review (3rd opinion) | Copilot Code Review (Business) | **OpenAI-family third reviewer** — uncorrelated from DeepSeek + GLM |

### The three-family reviewer stack

With DeepSeek authoring + GLM/Llama reviewing via OpenRouter + Copilot Code
Review as third pass, you have **three different model families** reviewing
every PR. This is the strongest reviewer-independence configuration available
without enterprise spend.

---

## Cursor — optional IDE-first alternative

**Source:** [cursor.com](https://cursor.com), `research-2026-07-tools.md`.

A VS Code fork built as an AI-first IDE. Cloud Agents GA, iOS beta. Supports
OpenRouter for some surfaces. **Limitation:** Cursor's agent mode is still
optimized for vendor frontier models (Claude, GPT, Gemini). OpenRouter support
exists but is less mature than OpenCode/Aider/Cline.

**Where it fits in Architect OS:** Optional. Use Cursor if you prefer an
IDE-first workflow and are willing to trade model flexibility for visual polish.
The Architect OS default remains OpenCode (terminal) + Cline (VS Code) because
both fully support the open-frontier model layer.

---

## Spec Kit (GitHub) — spec-driven commands

**Source:** [github.com/github/spec-kit](https://github.com/github/spec-kit)
(122k stars, v0.13.0 Jul 17 2026), `research-2026-07-specs.md`.

### What it is

GitHub's open-source CLI toolkit for spec-driven development. Installs as
`specify` CLI + slash commands (`/speckit.*`) into your coding agent of choice.
Works with OpenCode, Claude Code, Copilot, Gemini CLI, and 30+ other agents.

### Key commands

- `/speckit.specify` — requirements → spec
- `/speckit.clarify` — clarify underspecified areas before planning
- `/speckit.plan` — spec + constraints → technical plan
- `/speckit.tasks` — plan → dependency-ordered task list
- `/speckit.taskstoissues` — convert to GitHub Issues
- `/speckit.implement` — execute tasks
- `/speckit.converge` — **assess codebase against spec/plan, append remaining
  work as new tasks** ← the missing conformance pass
- `/speckit.analyze` — cross-artifact consistency analysis
- `/speckit.checklist` — "unit tests for English" — spec quality checklists
- `/speckit.constitution` — project governing principles

### Pricing

Free, MIT license. The `specify` CLI is Python, installed via `uv` or `pipx`.

### Where it fits in Architect OS

Spec Kit is **agent-agnostic** — it works inside OpenCode, Cline, or any other
BYO-key agent. Use Spec Kit's commands as the tooling inside Architect OS's
S2–S5 stages. The key new commands to adopt: `/speckit.converge` (spec
conformance at S7) and `/speckit.checklist` (spec quality gate at S2 exit).

---

## BMAD v6 — heavy-ceremony alternative

**Source:**
[github.com/bmad-code-org/BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD)
(50.8k stars, v6.10.0 Jul 3 2026), `research-2026-07-specs.md`.

Persona-based AI development methodology with **scale-adaptive sizing**
(automatically adjusts planning depth to project complexity — the fix for
architect-os's "over-specify a one-line bugfix" failure mode). 12+ specialized
agent personas. "Party Mode" brings multiple personas into one session for
debate. Test Architect module for risk-based test strategy. Web Bundles package
BMad skills as Google Gemini Gems and ChatGPT Custom GPTs — **run planning on a
flat-rate subscription, implement on metered APIs.** This is a pragmatic
cost-control pattern architect-os should note.

**Where it fits in Architect OS:** Heavy-ceremony profile. Adopt BMad's
scale-adaptive sizing in the S0/S1 size-classification step. Adopt the Web
Bundle pattern (flat-rate planning, metered implementation) as a cost-control
lever.

---

## The One-Glance Matrix (rewritten)

| Layer                             | Default                                                  | Alternative                                           | Skip when                                      |
| --------------------------------- | -------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------- |
| S0 Capture                        | None / OpenCode chat                                     | —                                                     | —                                              |
| S1 Frame (BRD)                    | OpenCode + DeepSeek V4 Pro                               | —                                                     | Lightweight: skip BRD                          |
| S2 Specify (PRD/FSD)              | OpenCode + DeepSeek V4 Pro + Spec Kit `/speckit.specify` | Spec Kit alone                                        | Lightweight: acceptance criteria in issue body |
| S3 Design                         | OpenCode + DeepSeek V4 Pro + Figma MCP                   | v0, Cursor Composer                                   | Headless; lightweight profile                  |
| S4 Architect                      | OpenCode + DeepSeek V4 Pro (plan mode)                   | Spec Kit `/speckit.plan`, BMad architect persona      | —                                              |
| S5 Plan (tickets)                 | OpenCode + DeepSeek V4 Pro + `to-tickets`                | Spec Kit `/speckit.tasks`, Aider architect mode       | Lightweight: plan in issue body                |
| S6 Implement                      | OpenCode + DeepSeek V4 Flash (fresh session per ticket)  | Aider (repo map), Cline (IDE), GLM 4.5 Air (fallback) | —                                              |
| S6 (hard ticket)                  | DeepSeek V4 Pro as implementer                           | Qwen 3 Coder, GLM 5.2                                 | When Flash struggles twice                     |
| S6 (XS async)                     | OpenCode backgrounded                                    | Copilot coding agent (Enterprise), Codex Web          | Lightweight: manual is fine for XS             |
| S6 (batch refactor)               | `opencode -p` loop over file list                        | Aider in batch mode                                   | —                                              |
| S7 Review (human)                 | You + 10-min rubric                                      | —                                                     | **Never skip**                                 |
| S7 Review (always-on AI)          | CodeRabbit                                               | —                                                     | —                                              |
| S7 Review (2nd AI, uncorrelated)  | OpenRouter → GLM 5.2                                     | Llama 4 Maverick, Qwen 3 Coder                        | Lightweight: skip if PR ≤50 lines              |
| S7 Review (3rd AI, OpenAI family) | Copilot Code Review (Business)                           | —                                                     | Lightweight: skip                              |
| S7 Review (runtime validation)    | Greptile TREX                                            | Self-rolled sandbox                                   | Lightweight: skip if PR is pure logic          |
| S7 Review (conformance)           | Spec Kit `/speckit.converge`                             | Custom converge skill                                 | Lightweight: skip if PR trivial                |
| S8 Release                        | CI/CD (Vercel/Cloudflare)                                | —                                                     | —                                              |
| S9 Learn                          | OpenCode + Gemini 2.5 Flash                              | DeepSeek V4 Flash                                     | Lightweight: skip graph update                 |

---

## Model routing defaults (rewritten)

| Task                       | Model                                  | Cost / typical op | Why                                                                                |
| -------------------------- | -------------------------------------- | ----------------- | ---------------------------------------------------------------------------------- |
| Spec generation (S2)       | **DeepSeek V4 Pro**                    | ~$8.72            | Intelligent planning, 28× cheaper than Opus 4.8. Thinking mode for deep reasoning. |
| Architecture (S4)          | **DeepSeek V4 Pro**                    | ~$17.44           | Deep trade-off analysis. 1M context fits whole repo.                               |
| Ticket decomposition (S5)  | **DeepSeek V4 Pro**                    | ~$5–10            | Reads real files, structured plans. Human gate catches errors.                     |
| Implementation (S6)        | **DeepSeek V4 Flash**                  | ~$1.40/ticket     | 35× cheaper than Sonnet 5. Fast, tool calls, FIM.                                  |
| Hard ticket fallback (S6)  | GLM 4.5 Air, Qwen 3 Coder              | ~$2–4/ticket      | Different family when Flash struggles.                                             |
| Self-review (S6)           | GLM 5.2 (via OpenRouter)               | ~$0.16            | **Different family from author** — uncorrelated.                                   |
| AI review always-on (S7)   | CodeRabbit (subscription)              | ~$12–15/mo        | Multi-model, always-on, path instructions + learnings.                             |
| AI review 2nd opinion (S7) | Llama 4 Maverick (via OpenRouter)      | ~$0.40/review     | Third family, 1M context.                                                          |
| AI review 3rd opinion (S7) | Copilot Code Review (Business)         | $19/user/mo       | OpenAI family. Third uncorrelated pass.                                            |
| Runtime validation (S7)    | Greptile TREX                          | subscription      | Sandboxed execution, catches runtime bugs.                                         |
| Spec conformance (S7)      | DeepSeek V4 Pro + Spec Kit `/converge` | ~$2/run           | Spec → diff → gap report.                                                          |
| Learning/dumps (S9)        | Gemini 2.5 Flash                       | ~$0.014/dump      | Cheapest viable, 1M context, different family.                                     |

### The three-family rule

For any PR >50 lines, run review across **three different model families**:

1. **CodeRabbit** (multi-model, always-on, subscription) — primary
2. **GLM 5.2 or Llama 4** (OpenRouter, per-token) — uncorrelated 2nd opinion,
   different family from author
3. **Copilot Code Review** (Business tier, OpenAI family) — 3rd opinion,
   different family from #1 and #2

If the author was DeepSeek, family #1 (CodeRabbit) uses Anthropic + OpenAI under
the hood, family #2 is GLM/Llama, family #3 is OpenAI. Three families, no shared
blind spots.

### The cache hit principle

DeepSeek cache hit input is **50× cheaper** than cache miss ($0.0028 vs $0.14
per 1M). To maximize cache hits:

- Keep `AGENTS.md` stable — it's loaded every session, cacheable
- Keep FSD sections stable — same spec section read across S5, S6, S7
- Use the same model within a ticket's lifecycle (S5 plan + S6 implement + S7
  self-review) — the prompt prefix is cacheable
- Don't inject timestamps or per-request context into the prefix — it breaks the
  cache

---

## Setup checklist (rewritten)

- [ ] **DeepSeek account** — top up $20 at
      [platform.deepseek.com](https://platform.deepseek.com/). Get API key. Test
      with
      `curl https://api.deepseek.com/v1/chat/completions -H "Authorization: Bearer $KEY" ...`
- [ ] **OpenRouter account** — top up $10 at
      [openrouter.ai](https://openrouter.ai/). Get API key. Verify GLM 5.2
      routing:
      `curl https://openrouter.ai/api/v1/chat/completions -H "Authorization: Bearer $OR_KEY" -d '{"model":"zhipuai/glm-5.2",...}'`
- [ ] **OpenCode installed** — `npm install -g opencode` or platform installer.
      Configure `opencode.json` with DeepSeek + OpenRouter providers (see sketch
      above).
- [ ] **Aider installed** (optional, for repo map) — `pip install aider-chat`.
      Configure `~/.aider.conf.yml` with DeepSeek + OpenRouter.
- [ ] **Cline installed** (optional, IDE layer) — VS Code marketplace. Configure
      with OpenRouter key.
- [ ] **CodeRabbit GitHub App installed** on all repos. Free for public,
      ~$12–15/mo for private. Drop in `.coderabbit.yaml` from this doc.
- [ ] **GitHub Copilot Business** ($19/user/mo) — for Copilot Code Review as the
      OpenAI-family third reviewer. Optional but recommended.
- [ ] **`gh` CLI authenticated** — for PR creation, issue management,
      `gh pr review`.
- [ ] **Spec Kit CLI installed** (optional) — `uv tool install specify-cli`.
      Provides `/speckit.*` commands.
- [ ] **Greptile** (optional, for runtime validation) — install on repos where
      runtime bugs are expensive.
- [ ] **claude-code-action** (optional, for Bedrock-routed uncorrelated review)
      — install GitHub Action. Configure with Bedrock backend + Llama 4 model.
- [ ] **Claude Code** (escape hatch) — install only if you need
      Anthropic-specific agent loop features. Configure with
      `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic` to use DeepSeek
      backend.
- [ ] **Codex CLI** (escape hatch) — `npm install -g @openai/codex` only if you
      already pay for ChatGPT and want a third implementation opinion.

### Monthly budget estimate

| Item                                                            | Cost            |
| --------------------------------------------------------------- | --------------- |
| DeepSeek API (15–20 tickets, planning + implementation)         | $15–40          |
| OpenRouter (GLM 5.2 + Llama 4 reviews, ~$0.16–0.40/review × 20) | $5–10           |
| CodeRabbit Pro                                                  | $12–15          |
| GitHub Copilot Business (optional, third reviewer)              | $19             |
| Greptile (optional, runtime validation)                         | ~$20            |
| **Total (default profile)**                                     | **~$32–65/mo**  |
| **Total (with all optional layers)**                            | **~$70–105/mo** |

Compare to the mid-2025 baseline: Claude Max $200 + CodeRabbit $15 + Codex
$10–20 + Copilot $10 = ~$232–249/mo. **The open-frontier stack is 65–85% cheaper
at near-equivalent quality for most tasks.**

---

## What changed from the mid-2025 matrix (summary)

| Mid-2025                                        | July 2026                                                                   |
| ----------------------------------------------- | --------------------------------------------------------------------------- |
| Claude Code as primary harness ($200/mo Max)    | OpenCode as primary harness ($0, BYO-key)                                   |
| Claude Sonnet/Opus for all stages               | DeepSeek V4 Pro for planning, V4 Flash for implementation                   |
| Codex CLI as second-opinion implementer         | GLM 5.2 / Qwen 3 / Llama 4 via OpenRouter as uncorrelated reviewers         |
| Copilot Enterprise for Code Review ($39 + GHEC) | Copilot Business for Code Review ($19, no GHEC required)                    |
| CodeRabbit as primary reviewer (unchanged)      | CodeRabbit as primary reviewer (now with path_instructions + Learnings)     |
| Traycer as S5 alternative                       | Spec Kit `/speckit.tasks` as S5 alternative (free, open-source, 122k stars) |
| "Claude Code has no async fire-and-forget"      | OpenCode backgrounded + Codex Web + Copilot coding agent all do async       |
| No runtime validation layer                     | Greptile TREX (Jun 2026) as optional S7 step                                |
| Same-vendor author+review accepted              | Constitution C36 — Reviewer independence (must be different family)         |
| ~$232–249/mo total                              | ~$32–65/mo default; ~$70–105/mo with all optional layers                    |

---

## What to _not_ change

- **CLI agents as primary harness.** The thesis holds — terminal agents read
  real files, run real tests, leave full audit trails. Only the _model behind
  the harness_ changed.
- **Narrow context per ticket.** DeepSeek's 1M context doesn't change the rule —
  broad context still degrades quality. Use the 1M for specs and long files, not
  for cramming the whole repo.
- **400-line PR cap.** Unchanged.
- **WIP limit 2.** Unchanged.
- **Human review first.** Unchanged. The three-family AI review stack is _after_
  the human rubric, not instead of it.
- **The spec is the source of truth.** Unchanged. DeepSeek V4 Pro is the spec
  author, not the spec.

---

## Constitution amendment recommendation

Add to `constitution.md`:

> **C36 — Reviewer independence.** The AI reviewer must not share model family
> with the author when both are AI. If the PR was authored by DeepSeek, the
> second-opinion reviewer must be GLM, Llama, Qwen, Gemini, or Copilot (OpenAI
> family). The third-opinion reviewer must be a third family. Same-vendor
> author+review is the correlated-vendor problem (see
> `insights-and-issues-2026-07.md` issue I2). CodeRabbit is exempt because it's
> multi-model under the hood and always-on (subscription, not per-token).

---

_Sources: DeepSeek pricing verified at api-docs.deepseek.com/quick_start/pricing
(fetched 2026-07-19). OpenRouter docs at openrouter.ai/docs. OpenCode, Aider,
Cline documentation. CodeRabbit docs at docs.coderabbit.ai. GitHub Copilot docs
at docs.github.com/copilot. Spec Kit repo at github.com/github/spec-kit
(v0.13.0, Jul 17 2026). BMad v6 repo at github.com/bmad-code-org/BMAD-METHOD
(v6.10.0, Jul 3 2026). Greptile TREX at greptile.com/blog/trex. Cross-referenced
with the six `research-2026-07-*.md` reports and
`insights-and-issues-2026-07.md` in this directory._
