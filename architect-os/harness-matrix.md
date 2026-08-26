# Harness Matrix — Which Tool, Which Layer, and Why

_Every tool comparison in one place, cited to primary sources. Covers **both
stacks** of the OS: the **default stack** (Claude Code + Anthropic models) and
the **open-frontier stack** (OpenCode + DeepSeek via OpenRouter) — canonical
definitions in [adoption-plan.md](adoption-plan.md). The question isn't "which
tool is best" — it's "which model for which job at which stage, at what cost."_

**Confidence tiers.** Every load-bearing fact in this doc carries one:

- ✅ **verified-primary** — fetched from the official source on 2026-07-19
- 📣 **vendor-claim** — from the vendor's own blog/marketing; directionally
  useful, not independently verified
- ❔ **inferred** — cross-referenced from secondary mentions; treat as a
  hypothesis, re-verify before relying on it

Full evidence trail: `research/research-2026-07-tools.md` (12 fetches, 6 succeeded),
`research/research-2026-07-review.md`, `research/research-2026-07-update.md`, in `research/`.
This doc eats its own dog food: re-verify everything after **2026-10-19** (the
research snapshot expiry).

---

## Part I — The shape of the market (July 2026)

The mid-2025 trichotomy (IDE extensions / CLI agents / cloud agents) is
obsolete. The market reorganized into **seven categories**; the two axes that
matter are _who runs the agent_ (harness) and _which model powers it_ (routing
decision). ✅ `research/research-2026-07-tools.md`

| Category                         | Example tools                                                                        | Defining trait                                                                          |
| -------------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| **Agent-native control centers** | Claude Code Desktop/Web, GitHub Copilot app, Cursor + Cloud Agents, Devin            | Multi-session orchestration UI ("My Work"), canvases, worktrees, cross-surface teleport |
| **CLI agents (BYO-key)**         | OpenCode, Aider, Cline, Continue                                                     | Terminal-native, file-grounded, model-agnostic, fresh session per task                  |
| **Vendor-locked CLI agents**     | Claude Code, Codex CLI                                                               | Terminal agents tied to vendor models (widening: Claude Code accepts DeepSeek)          |
| **IDE agents**                   | Cursor, Windsurf, Cline, Continue, Zed AI, Trae, Copilot in-editor                   | Embedded in editor, synchronous, inline diffs                                           |
| **Cloud async agents**           | Claude Code Routines/Web, Copilot cloud agent, Codex Web, Devin, Cursor Cloud Agents | Event/schedule-triggered, runs while you're offline                                     |
| **Code review platforms**        | CodeRabbit, Copilot code review, Cursor Bugbot, Devin Review, Greptile               | Distinct product category with tiered reasoning, attribution, follow-through            |
| **Vibe-coding app builders**     | Replit Agent, Lovable, Bolt, v0                                                      | Non-technical audience, idea → deployed app, no real codebase                           |

Plus **spec/workflow frameworks** (GitHub Spec Kit, BMAD, Architect OS itself)
— methodology + prompts, not model-backed products.

**How to read this matrix:** the OS runs **two named stacks** (canonical
definitions: [adoption-plan.md](adoption-plan.md)). **Part II** documents the
open-frontier stack's components (OpenCode, DeepSeek, OpenRouter, GLM/Llama
reviewers). **Part III** documents the wider market — including the default
stack's harness and reviewers (Claude Code, Codex, Copilot). One routing
principle governs both stacks: intelligent models for planning, cheap models
for implementation, different model families for review. Pick one stack per
repo; vendor-locked control centers are optional escape hatches and async
lanes on either stack.

**MCP is load-bearing in 2026.** The Model Context Protocol became the de
facto integration standard; tools without MCP support look architecturally
stranded. Scores per tool in Part IV.

---

## Part II — The Architect OS stack (what we use)

### The cost-quality frontier (July 2026)

Open-frontier models closed the quality gap with Anthropic/OpenAI while
staying 10–40× cheaper. The routing problem is no longer "which vendor" — it's
"which tier at which stage."

#### The open-frontier model landscape

| Model                         | Input $/1M | Output $/1M | Context | Best for                                                     | Confidence |
| ----------------------------- | ---------- | ----------- | ------- | ------------------------------------------------------------ | ---------- |
| **DeepSeek V4 Pro**           | $0.435     | $0.87       | 1M      | Planning, spec generation, architecture, reasoning           | ✅ api-docs.deepseek.com |
| **DeepSeek V4 Flash**         | $0.14      | $0.28       | 1M      | Implementation, coding, self-review                          | ✅ cache hit input: $0.0028/1M |
| **GLM 5.2** (Zhipu)           | ~$0.50     | ~$2.00      | 200k    | Code review, uncorrelated second opinion                     | ❔ OpenRouter catalog |
| **GLM 4.5 Air**               | ~$0.10     | ~$0.40      | 128k    | Cheap implementation, mechanical edits                       | ❔ |
| **Qwen 3 Coder** (Alibaba)    | ~$0.20     | ~$0.60      | 256k    | Code generation, refactoring                                 | ❔ |
| **Qwen 3 235B**               | ~$0.15     | ~$0.60      | 256k    | General reasoning, planning fallback                         | ❔ |
| **Llama 4 Maverick** (Meta)   | ~$0.20     | ~$0.60      | 1M      | Long-context review, uncorrelated auditor                    | ❔ |
| **Kimi K2** (Moonshot)        | ~$0.60     | ~$2.50      | 256k    | Reasoning, planning alternative                              | ❔ |
| **Gemini 2.5 Flash** (Google) | $0.075     | $0.30       | 1M      | Mechanical work, dumps, classification                       | ❔ |
| **Gemini 2.5 Pro** (Google)   | $1.25      | $10.00      | 2M      | Long-context planning, massive repo analysis                 | ❔ |
| **Grok 4** (xAI)              | ~$3.00     | ~$15.00     | 256k    | Reasoning alternative                                        | ❔ |

#### The vendor frontier (escape hatches)

| Model           | Input $/1M    | Output $/1M | When to use                                                                                     | Confidence |
| --------------- | ------------- | ----------- | ----------------------------------------------------------------------------------------------- | ---------- |
| Claude Opus 4.8 | $5.00         | $25.00      | Only when DeepSeek V4 Pro fails a planning task — highest reasoning ceiling, ~16× more expensive per call | ✅ |
| Claude Sonnet 5 | $2.00 (intro) | $10.00      | Only when DeepSeek V4 Flash produces low-quality code on a hard ticket — ~20× more expensive per call. **⏰ Intro price ends 2026-08-31** ($3/$15 after) | ✅ |
| GPT-5.6-sol     | $5.00         | $30.00      | Escape hatch for OpenAI-ecosystem tasks                                                         | 📣 tier naming per CodeRabbit benchmark |

#### The routing principle

> **Intelligent models for planning (where mistakes are expensive), cheap flash
> models for coding (where iterations are cheap), different model families for
> review (where correlation is the enemy).**

Planning mistakes cost 10× the diff they produce. Coding mistakes cost 1× the
diff. Review mistakes from correlated models cost the bug rate of the author's
blind spots. Route accordingly.

---

### OpenRouter — the routing layer

**Confidence:** ✅ openrouter.ai/docs fetched 2026-07-19.

A unified API across 400+ models. One API key, one billing surface, automatic
fallbacks, and an MCP server at `https://mcp.openrouter.ai/mcp` that lets any
coding agent pull live model and pricing data.

- **Unified API:** `OPENAI_BASE_URL=https://openrouter.ai/api/v1` works with
  OpenCode, Aider, Cline, Continue, and any OpenAI-compatible client.
- **Latest aliases:** `~openai/gpt-latest`, `~anthropic/claude-sonnet-latest`,
  `~deepseek/deepseek-latest` — no model-name churn.
- **Automatic fallbacks:** DeepSeek down → GLM 5.2 → Qwen 3 → Llama 4, per-route.
- **Provider routing:** pin cheapest/fastest provider per model.
- **Pricing:** pay-as-you-go credits; small markup (typically 5–20%) over
  provider pricing ❔ — compare against direct API for frontier models.

#### Where it fits

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

### OpenCode — the open-frontier stack's primary harness

**Confidence:** ❔ — the tools research *ran on* OpenCode but never fetched its
docs as a primary source; capability list is self-referenced. Verify at
opencode.ai before relying on a specific feature.

Terminal-based, model-agnostic coding agent. Reads files, writes code, runs
commands, tests its own work, self-corrects in a loop. Accepts any
OpenAI-compatible API, OpenRouter, or direct provider endpoints. The harness
the open-frontier stack is built on.

- **Model-agnostic:** `opencode.json` with any provider; switch models per-stage.
- **Subagents:** `task` tool dispatches parallel subagents in isolated context.
- **Skills:** `.opencode/skills/`, same convention as Claude Code.
- **AGENTS.md / opencode.json:** project instructions + per-stage model routing.
- **MCP support:** connect any MCP server (Figma, GitHub, OpenRouter pricing).
- **Plan mode, hooks, context isolation** per ticket.

**Pricing:** open source — **$0 for the harness.** You pay only for model API
usage. The core economic advantage over Claude Code ($200/mo Max) and Codex
($20/mo ChatGPT).

| Stage                | Role                                      | Model routed               |
| -------------------- | ----------------------------------------- | -------------------------- |
| S1 Frame (BRD)       | Primary — interactive grilling + drafting | DeepSeek V4 Pro            |
| S2 Specify (PRD/FSD) | Primary — spec creation + grill-with-docs | DeepSeek V4 Pro            |
| S3 Design            | Primary + Figma MCP                       | DeepSeek V4 Pro            |
| S4 Architect         | Primary — plan mode, ADR drafting         | DeepSeek V4 Pro            |
| S5 Plan (tickets)    | Primary — to-tickets + wayfinder          | DeepSeek V4 Pro            |
| S6 Implement         | Primary — one fresh session per ticket    | DeepSeek V4 Flash          |
| S6 Self-review (C22) | Author's own pass — same model as author  | DeepSeek V4 Flash          |
| S7 Second opinion    | **C36 cross-family reviewer**             | GLM 5.2 (different family) |
| S9 Learn             | Primary — mechanical                      | Gemini 2.5 Flash           |

#### Configuration sketch

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

### Aider — alternative CLI harness

**Confidence:** ✅ github.com/Aider-AI/aider fetched 2026-07-19 (47.5k stars,
Apache-2.0; latest release v0.86.0 Aug 2025, repo active through 2026).

Terminal pair-programming agent. Stronger than OpenCode on one axis:
**automatic repo-map construction** via tree-sitter + PageRank-style symbol
ranking — the `repo-graph.json` Layer 3 from `repo-memory.md`, built
automatically where architect-os hand-curates. Weaker on subagent/multi-agent.

- **OpenRouter native:** `--model openrouter/zhipuai/glm-5.2` just works.
- **Architect mode:** separate architect model (planning) and editor model
  (implementation) in one run — the Architect OS S5→S6 split in one command:
  `aider --architect-model deepseek/deepseek-v4-pro --model deepseek/deepseek-v4-flash`
- **Git-native:** every edit is a commit. 88% of new code in the last release
  written by Aider itself 📣.

**Pricing:** open source, $0 + API usage. **Where it fits:** S6 alternative on
large repos (repo map gives better symbol context); S4 alternative (the map
surfaces architecture).

---

### Cline — IDE-native harness

**Confidence:** ✅ via `research/research-2026-07-tools.md`.

VS Code extension turning the editor into an agent surface. First-class
OpenRouter support, MCP support, `@`-mentions, plan/act mode split.

**Pricing:** open source, $0 + API usage. **Where it fits:** the IDE layer —
exploration, light edits, visual diff review. Use OpenCode for S5–S6 heavy
work (narrow context, auditable sessions); Cline for reading code and small
edits.

---

### DeepSeek — the open-frontier stack's primary model provider

**Confidence:** ✅ api-docs.deepseek.com/quick_start/pricing fetched 2026-07-19 —
but note this is a *single-fetch* ✅ carrying the most load-bearing numbers in
the doc; double-source it at the 2026-10-19 re-verification.

| Model                 | Input (cache miss) | Input (cache hit) | Output   | Context | Concurrency |
| --------------------- | ------------------ | ----------------- | -------- | ------- | ----------- |
| **deepseek-v4-flash** | $0.14/1M           | $0.0028/1M        | $0.28/1M | 1M      | 2500        |
| **deepseek-v4-pro**   | $0.435/1M          | $0.003625/1M      | $0.87/1M | 1M      | 500         |

- **Thinking mode** (default) and non-thinking; **tool calls; JSON output; FIM.**
- **Context caching:** cache hit input is **50× cheaper** ($0.0028 vs $0.14).
  Keep AGENTS.md and spec sections stable to maximize hits.
- **Anthropic-compatible API:** `https://api.deepseek.com/anthropic` — Claude
  Code, claude-code-action, and any Anthropic-ecosystem tool can run on
  DeepSeek. **The correlated-vendor escape hatch:** Anthropic's agent loop +
  an uncorrelated model.
- **1M context:** fits entire repos or long specs — but narrow context per
  ticket remains the rule (quality, not cost).

#### Pricing comparison vs vendor frontier

Per single API call at the stated token volumes (in + out):

| Task                                          | DeepSeek V4 Pro                | Claude Opus 4.8            | Multiple        |
| --------------------------------------------- | ------------------------------ | -------------------------- | --------------- |
| Spec generation (S2, ~50k input, ~10k output) | $0.022 + $0.009 = **$0.031**   | $0.25 + $0.25 = **$0.50**  | **~16× cheaper** |
| Architecture (S4, ~100k input, ~20k output)   | $0.044 + $0.017 = **$0.061**   | $0.50 + $0.50 = **$1.00**  | **~16× cheaper** |

| Task                                        | DeepSeek V4 Flash              | Claude Sonnet 5             | Multiple        |
| ------------------------------------------- | ------------------------------ | --------------------------- | --------------- |
| Implementation (S6, ~30k input, ~5k output) | $0.004 + $0.001 = **$0.006**   | $0.06 + $0.05 = **$0.11**   | **~20× cheaper** |
| Self-review (S7, ~20k input, ~3k output)    | $0.003 + $0.001 = **$0.004**   | $0.04 + $0.03 = **$0.07**   | **~20× cheaper** |

**The agentic multiplier:** a real ticket session is not one call — tool loops,
retries, and re-reads run **10–50×** a single call's tokens (caching absorbs
most of the input side). That's how ~$0.006 calls become the **~$15–40/mo**
DeepSeek estimate at 15–20 tickets, and $0.11 calls become a $200/mo Max plan
being fair value on the default stack.

**Honesty note on totals:** the per-token gap is 16–20×, but the *bill* gap is
smaller — most of the residual open-frontier spend is subscriptions both stacks
share (CodeRabbit, ChatGPT-for-Codex). Compare totals in Part VI, not model
prices alone.

---

### Open-frontier models for review (GLM, Qwen, Llama, Gemini)

**Confidence:** ❔ pricing from OpenRouter catalog + provider pages, not all
directly fetched. The reviewer-independence principle is ✅ (constitution C36).

The AI reviewer must not share model family with the author (C36). Route review
to a _different family_:

- **GLM 5.2 (Zhipu) — primary review model.** Different training corpus from
  DeepSeek. ~$0.50/$2.00 per 1M; a typical review ~$0.02. Via OpenRouter
  (`zhipuai/glm-5.2`). S7 second opinion, S4 council debates.
- **Llama 4 Maverick (Meta) — long-context review.** 1M context, third family.
  ~$0.20/$0.60. S7 third opinion, large-PR review.
- **Qwen 3 Coder (Alibaba) — code-specialized review.** Strong on
  code-specific bugs. ~$0.20/$0.60. S7 bug hunting, S6 fallback.
- **Gemini 2.5 Flash (Google) — mechanical and dump.** $0.075/$0.30; a memory
  dump ~$0.014. S9 dumps, classification, S7 quick skim.

---

### CodeRabbit — always-on AI reviewer

**Confidence:** ✅ coderabbit.ai/blog fetched 2026-07-19. **Pricing ✅ re-verified
2026-08-27** — the previous ❔ estimate (~$12–15 Pro) was wrong by ~2×.

**Tiers:** Free · Pro **$24**/user/mo annual (**$30** monthly) · Pro Plus $48 ·
Enterprise (custom, from ~$15k/mo at 500+ users). Billing counts **only
developers who open PRs** — reviewers and managers are not seats.

**Start on Free.** Pro is **free forever on public repos** (full feature set:
auto-fix, 40+ linters, custom instructions). On private repos the free tier
gives PR summaries, AI review comments, unlimited repos and members, and
IDE/CLI reviews — rate-limited to **200 files/hour and 4 PR reviews/hour**,
which is far above a WIP≤2 solo pipeline. The canonical
[.coderabbit.yaml](github/.coderabbit.yaml) works on Free; upgrade only on
hitting the limit.

Now a **review-and-follow-through platform**, not just a PR bot: PR / IDE /
CLI / Discord surfaces, **Change Stack** (guided layer-by-layer PR walkthrough —
your reading order for the 10-minute rubric), **Overview page** (PR home:
"what is this and can it merge"), **Post-Merge Actions** (Jul 16 2026:
changelogs, docs, tickets after merge), **source attribution** on every
comment (Jul 2 2026).

- **Multi-model pipeline:** Claude for deep analysis, GPT for cross-validation —
  the one place you pay for premium models, but as a subscription, not per-token.
- **Path Instructions + Learnings + `@coderabbitai emit path instructions`:**
  the constitutional hook and the learning loop (see rituals-and-metrics.md).

**Where it fits:** S7 always-on first AI pass on every PR. It is **not** the
C36 second opinion — that must be cross-family from the author.

**Config:** the canonical `.coderabbit.yaml` lives at
[github/.coderabbit.yaml](github/.coderabbit.yaml) (assertive profile, `[Cn]`
citations with severity, 🟡/🔵 batching, explicit re-includes for
`docs/`/`templates/`/`memory/*.json`). This doc deliberately does not duplicate
it — one source of truth.

---

### claude-code-action — uncorrelated review via Bedrock/Vertex/DeepSeek

**Confidence:** ✅ github.com/anthropics/claude-code-action via
`research/research-2026-07-review.md`.

Anthropic's GitHub Action for running Claude Code on PRs/issues. The key trick:
it supports **Bedrock and Vertex backends hosting non-Claude models** (Llama 4,
Nova) — the Claude Code agent loop with a non-Claude brain. Alternative: point
it at DeepSeek's Anthropic-compatible API (`api.deepseek.com/anthropic`).

**Where it fits:** S7 cross-family second opinion when the author was
Claude-family. Free action; you pay model API (~$0.20–0.87/1M input).

---

## Part III — The wider market (what we track, don't default to)

### Claude Code — multi-surface agent platform

**Confidence:** ✅ docs.anthropic.com + anthropic.com/news fetched 2026-07-19.

Claude Code is no longer "a CLI." It's a seven-surface platform — **Terminal
CLI, VS Code, JetBrains, Desktop (macOS/Windows x64+ARM), Web
(claude.ai/code), iOS, Android** — with sessions moving across surfaces via
`--teleport`, `--cloud`, `/desktop`, Remote Control, and Dispatch.

**Primitives (all shipped, all relevant to this OS):**

| Primitive              | What it does                                                              | Architect OS relevance                                        |
| ---------------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **Subagents**          | Lead agent coordinates parallel workers, merges results                   | The `task` pattern; orchestrator-worker reference             |
| **Background agents**  | Parallel full sessions from one screen                                    | WIP-managed parallel tickets                                  |
| **Agent teams**        | Coordinated multi-session with shared tasks, messaging, team lead         | The 20% multi-agent case — see [multi-agent.md](multi-agent.md) |
| **Agent SDK**          | Custom agents with full orchestration/permissions control                 | Building OS-native tooling                                    |
| **Skills / Hooks**     | Packaged workflows; shell commands before/after actions                   | Same conventions this OS uses                                 |
| **Routines**           | Scheduled or API/GitHub-event-triggered, Anthropic-managed, computer-off  | **The async primitive the mid-2025 matrix said was missing**  |
| **Channels**           | Telegram/Discord/iMessage/webhook events pushed into a session            | Event-triggered work — governed by **C37**                    |
| **Claude Code on the web** | Isolated cloud VMs, async fire-and-forget                             | S6 XS-async lane                                              |
| **Slack integration**  | `@Claude` bug report → PR back                                            | Triage lane — C37 applies                                     |
| **GitHub Code Review** | Native automatic review on every PR                                       | Competes with CodeRabbit; same-family if Claude authored (C36)|
| **Auto memory**        | Cross-session memory without authored CLAUDE.md                           | Compare repo-memory.md (ours is human-filtered, theirs auto)  |

The mid-2025 claim "no async fire-and-forget" is **obsolete** — Routines,
Channels, web sessions, and agent teams are the async primitives.

**The DeepSeek escape hatch:** `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic`
+ DeepSeek key → Anthropic's agent loop at DeepSeek's price (~$15–40/mo vs
$200/mo Max). Use when you need a Claude-Code-specific feature (a skill, an
MCP integration) OpenCode doesn't yet support.

**Where it fits:** the **default stack's** harness (planning on Opus 4.8,
implementation on Sonnet 5); on the open-frontier stack, an optional escape
hatch and async-lane candidate. Either way, its native patterns (worktrees,
`claude -p` batch, agent teams, `/code-review` skill) are the reference
implementation for [multi-agent.md](multi-agent.md).

---

### OpenAI Codex — four surfaces

**Confidence:** ✅ github.com/openai/codex fetched 2026-07-19 (99.5k stars,
Rust-based, v0.144.6 Jul 18 2026). Model tier naming 📣 (CodeRabbit benchmark,
Jul 9 2026 — OpenAI's own pages 404'd at fetch time).

1. **Codex CLI** — Rust terminal agent, runs locally.
2. **Codex IDE** — VS Code / Cursor / Windsurf extension.
3. **Codex Web** — cloud async agent at chatgpt.com/codex (fire-and-forget).
4. **Codex App** — `codex app` desktop experience.

Auth: "Sign in with ChatGPT" (bundled with Plus/Pro/Business/Edu/Enterprise)
or API key. Supports the `AGENTS.md` cross-tool convention. Model tiers:
GPT-5.6 Sol (flagship) / Terra (lower-cost) / Luna (fastest) 📣.

**Where it fits:** **the standing C36 cross-family reviewer** (`codex review`
against Claude/DeepSeek-authored PRs — see [review-workflow.md](review-workflow.md));
optional third implementation opinion on hard tickets; Codex Web as an async
XS lane. Only worth it if you already pay for ChatGPT.

---

### GitHub Copilot — a 12-product platform family

**Confidence:** ✅ docs.github.com/copilot + github.blog fetched 2026-07-19.

The mid-2025 matrix's "Workspace + coding agent + Code Review" is now a
platform family:

- **Copilot app** (technical preview, Jun 2 2026): agent-native desktop "My
  Work" across repos — sessions, issues, PRs, automations; **each session in
  its own git worktree** (no manual setup).
- **Agent Merge:** carries a PR through review, CI, and merge with configurable
  autonomy (drive CI green / address feedback / merge).
- **Canvases:** bidirectional human+agent work surfaces (plans, PRs, terminals,
  deployments). "Agent experience (AX)" is now official GitHub terminology.
- **Sandboxes:** local (restricted fs/network, policy-enforced) or cloud
  (ephemeral Linux, GitHub-hosted).
- **Code review:** **low/medium reasoning tiers** (per-repo admin setting);
  `/security-review` and `/rubberduck` skills GA; **Azure DevOps support**;
  available at **Business tier ($19/user/mo)** — no Enterprise needed.
- **Copilot CLI:** redesigned TUI, voice mode, `/every` recurring prompts.
- **Cloud agent:** issue filing, discussions, reviewer replies — not just code.
- **SDK (GA):** Node/TS, Python, Go, .NET, Rust, Java — build on the same
  agentic runtime as the Copilot app.
- **Memory++ / `/chronicle`:** cross-device, cross-time continuity.
- **Partner agent apps:** LaunchDarkly, Bright, Amplitude, Sonar, Endor Labs,
  Octopus Deploy, Packfiles, PagerDuty, Miro.
- **MCP Registry:** public registry at github.com/mcp.
- **Billing:** usage-based via GitHub AI Credits (effective Jun 1 2026);
  individual Pro / Pro+ / Max plans.

| Tier       | Price       | Key features                                          |
| ---------- | ----------- | ----------------------------------------------------- |
| Free       | $0          | 2K completions/mo, 50 chat messages/mo                |
| Pro        | $10/mo      | Unlimited completions + chat, multi-model, @workspace |
| Business   | $19/user/mo | Pro + Code Review, org policies, IP indemnity         |
| Enterprise | $39/user/mo | Business + coding agent, knowledge bases              |

**Where it fits:** S6 IDE exploration (Pro); S7 **OpenAI-family third
reviewer** via code review (Business); coding agent / cloud automations as an
async XS lane. The strongest managed-platform alternative if you ever trade
the BYO-key stack for a subscription one.

---

### Cursor — agent-native platform, not just an IDE

**Confidence:** ✅ cursor.com/blog fetched 2026-07-19. Performance and revenue
figures 📣 vendor-reported.

Products: Agents, **Cloud Agents (GA)**, Composer, **iOS beta** (Jun 29 2026),
Automations, CLI, Marketplace, **Bugbot** review, **SDK** (Notion embeds it),
Organizations (Enterprise), Team Marketplace MCPs.

- **Bugbot:** 3× faster, 22% cheaper, 10% more bugs found (Jun 10 2026) 📣.
- **Auto-review** (Jun 11 2026): automated gate before agent actions land.
- **Design Mode** (Jun 5 2026): point/draw/narrate UI in browser, agents edit
  the code underneath.
- **Gartner MQ Leader**, Enterprise AI Coding Agents (May 22 2026, via vendor) 📣.
- Bloomberg: $2B ARR, doubled in three months (Mar 2 2026) 📣.

**Where it fits:** optional IDE-first alternative for those who trade model
flexibility for visual polish — its agent mode is still optimized for vendor
frontier models and OpenRouter support is less mature than OpenCode/Aider/Cline.
**C36 note:** Bugbot never reviews Cursor-authored PRs (same vendor).

---

### Devin (Cognition) — delegated long-horizon agent

**Confidence:** ✅ devin.ai fetched 2026-07-19. Customer metrics 📣.

Four surfaces — **Cloud, Desktop, CLI, Windows VM** — plus **Devin Review**,
**Security Swarm**, and **DeepWiki** (auto-generated docs + diagrams for legacy
codebases). **Devin Automations:** schedule, API, Linear/Slack/Teams triggers.
Enterprise tier; government and security verticals.

Use cases: PR review + visual QA, documentation, code migration (COBOL, .NET,
Talend, legacy ETL), scheduled chores, issue triage. 📣 Nubank case study:
8–12× engineering efficiency, 20× cost savings on an 8-year ETL monolith
migration; fine-tuning on past examples doubled completion scores.

**Where it fits:** not a daily harness — the benchmark for **delegated
multi-week multi-repo projects**. If a migration is too big for the batch
pattern in [multi-agent.md](multi-agent.md), Devin is the managed alternative.
DeepWiki is the reference for auto-built repo docs (compare repo-memory.md
Layer 3 — ours stays human-filtered).

---

### Vibe-coding app builders — S1 prototype lane only

**Confidence:** ✅ replit.com/ai + lovable.dev fetched 2026-07-19. Bolt/v0 ❔
(indirect evidence only).

- **Replit Agent 4:** idea → built app; screenshot-to-app; Agent/Design/
  Database/Publish family. Audience: non-technical creators.
- **Lovable:** chat-based app generation, templates, MCP server, desktop app,
  enterprise tier.
- **Bolt / v0:** same segment (StackBlitz / Vercel respectively).

**Where it fits:** **never as implementation harnesses.** Legitimate use:
S1/S2 idea-to-prototype — a disposable visual to react to before the BRD
hardens. Output never enters the repo as code; it informs the spec, then it's
thrown away (the prototype contract, lifecycle S2).

---

## Part IV — Sub-matrices

### Code review sub-matrix

The 2026 review category, scored on the dimensions that matter. (C36 routing:
[review-workflow.md](review-workflow.md).)

| Reviewer              | Reasoning tiers                    | Source attribution          | Post-merge follow-through        | Surfaces              | Confidence |
| --------------------- | ---------------------------------- | --------------------------- | -------------------------------- | --------------------- | ---------- |
| **CodeRabbit**        | Multi-model (Claude + GPT cross-check) | ✅ every comment (Jul 2026) | ✅ Post-Merge Actions (Jul 2026) | PR, IDE, CLI, Discord | ✅ |
| **Copilot code review** | Low / medium (higher-reasoning routing) | Per-repo guidelines      | —                                | PR, IDE, Azure DevOps | ✅ |
| **Cursor Bugbot**     | Single tuned tier                  | —                           | Auto-review gate                 | PR                    | 📣 perf |
| **Devin Review**      | —                                  | —                           | Visual QA                        | PR                    | ✅ exists / ❔ detail |
| **Greptile (+TREX)**  | Full-repo context                  | Guidelines                  | **TREX runtime sandbox artifacts** | PR                  | ✅ exists / 📣 benchmarks |
| **`codex review`**    | Model-dependent                    | —                           | —                                | CLI-local             | ❔ |

**Runtime validation** (Greptile TREX, Jun 2026) is the one capability static
review structurally lacks — it runs the changed code and produces artifacts.
At solo scale the `converge` skill covers the cheap version (tests run as
evidence); TREX is the subscription upgrade for repos where runtime bugs are
expensive.

### Agent-native control center sub-table

| Surface                 | Multi-session view              | Worktrees/session | Cross-surface teleport | Sandboxing                  | Scheduling            | Confidence |
| ----------------------- | ------------------------------- | ----------------- | ---------------------- | --------------------------- | --------------------- | ---------- |
| **Claude Code Desktop/Web** | Background agents + agent teams | ✅ | ✅ (`--teleport`, `--cloud`, Dispatch) | Standard permission model | Routines              | ✅ |
| **Copilot app** (preview) | "My Work" + canvases            | ✅ automatic      | —                      | Local + cloud, policy-enforced | `/every`, automations | ✅ |
| **Cursor**              | Cloud Agents + iOS              | —                 | —                      | Cloud VMs                   | Automations           | ✅/📣 |
| **Devin Cloud/Desktop** | Parallel sessions               | —                 | —                      | Cloud env, Windows VM       | Devin Automations     | ✅ |

### Async / fire-and-forget sub-table

Everything here is governed by **C37** (event-triggered agents ingest data,
never instructions) — see [constitution.md](constitution.md).

| Primitive                    | Triggers                        | Computer-off | Write-permission model          | Audit trail    | Confidence |
| ---------------------------- | ------------------------------- | ------------ | ------------------------------- | -------------- | ---------- |
| **Claude Code Routines**     | Schedule, API, GitHub events    | ✅           | Permission model + C37 limits   | Session logs, PRs | ✅ |
| **Copilot cloud automations** | Schedule, GitHub events        | ✅           | Policy-enforced sandbox         | PR trail       | ✅ |
| **Codex Web**                | Manual fire-and-forget          | ✅           | Sandboxed env                   | PR output      | ✅ |
| **Devin Automations**        | Schedule, API, Linear/Slack/Teams | ✅         | Enterprise controls             | Session replay | ✅ |
| **Cursor Cloud Agents**      | Event/schedule                  | ✅           | Auto-review gate                | PR output      | ✅ |

### MCP support scores

| Tool         | MCP status                                          | Confidence |
| ------------ | --------------------------------------------------- | ---------- |
| Claude Code  | Consumer + producer (MCP originates at Anthropic)   | ✅ |
| GitHub Copilot | Consumer + public MCP Registry (github.com/mcp)   | ✅ |
| Cursor       | Consumer + Team Marketplace MCPs                    | ✅ |
| Cline        | Consumer (first-class)                              | ✅ |
| OpenCode     | Consumer                                            | ❔ self-referenced |
| CodeRabbit   | Consumer (knowledge-base MCP connections)           | ✅ |
| Lovable      | Producer (MCP server)                               | ✅ |
| Codex CLI    | Consumer                                            | ❔ |
| Aider        | None verified                                       | ❔ |
| Devin        | Rich integrations; MCP-native unclear               | ❔ |

---

## Part V — Spec/workflow frameworks

### Spec Kit (GitHub) — spec-driven commands

**Confidence:** ✅ github.com/github/spec-kit fetched 2026-07-19 (122k stars,
v0.13.0 Jul 17 2026). Free, MIT.

GitHub's open-source CLI toolkit: `/speckit.specify`, `/speckit.clarify`,
`/speckit.plan`, `/speckit.tasks`, `/speckit.taskstoissues`,
`/speckit.implement`, **`/speckit.converge`** (assess codebase against
spec/plan, append remaining work as tasks), `/speckit.analyze`,
`/speckit.checklist`, `/speckit.constitution`. Agent-agnostic — runs inside
OpenCode, Cline, Claude Code, Copilot, 30+ agents.

**Where it fits:** optional tooling inside S2–S5. Note: the OS-native
[`converge` skill](skills/converge/SKILL.md) (Pass 2) already covers the
conformance pass — adopt Spec Kit only if you want its full command surface,
not just converge.

### BMAD v6 — heavy-ceremony alternative

**Confidence:** ✅ github.com/bmad-code-org/BMAD-METHOD (50.8k stars, v6.10.0
Jul 3 2026).

Persona-based methodology with scale-adaptive sizing (the fix for
"over-specify a one-line bugfix" — absorbed into lifecycle S1 in Pass 2), 12+
personas, Party Mode debate, Test Architect module, and **Web Bundles**
(planning on flat-rate Gemini Gems / Custom GPTs, implementation on metered
APIs — a pragmatic cost pattern).

**Where it fits:** heavy-ceremony profile reference. Good ideas already
absorbed (sizing); don't migrate wholesale.

---

## Part VI — The decisions

### The One-Glance Matrix

Columns show the **open-frontier stack's** picks — "default" is deliberately
avoided here because [adoption-plan.md](adoption-plan.md) names Claude Code +
Anthropic the *default stack*; its stage mapping lives there and in Part III.

| Layer                             | Open-frontier pick                                       | Alternatives                                          | Skip when                                      |
| --------------------------------- | -------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------- |
| S0 Capture                        | None / OpenCode chat                                     | —                                                     | —                                              |
| S1 Frame (BRD)                    | OpenCode + DeepSeek V4 Pro                               | —                                                     | Lightweight: skip BRD                          |
| S1 (idea→prototype visual)        | Replit Agent / Lovable (throwaway)                       | v0                                                    | Not an implementation path — spec input only   |
| S2 Specify (PRD/FSD)              | OpenCode + DeepSeek V4 Pro + Spec Kit `/speckit.specify` | Spec Kit alone                                        | Lightweight: acceptance criteria in issue body |
| S3 Design                         | OpenCode + DeepSeek V4 Pro + Figma MCP                   | v0, Cursor Composer                                   | Headless; lightweight profile                  |
| S4 Architect                      | OpenCode + DeepSeek V4 Pro (plan mode)                   | Spec Kit `/speckit.plan`, BMad architect persona      | —                                              |
| S5 Plan (tickets)                 | OpenCode + DeepSeek V4 Pro + `to-tickets`                | Spec Kit `/speckit.tasks`, Aider architect mode       | Lightweight: plan in issue body                |
| S6 Implement                      | OpenCode + DeepSeek V4 Flash (fresh session per ticket)  | Aider (repo map), Cline (IDE), GLM 4.5 Air (fallback) | —                                              |
| S6 (hard ticket)                  | DeepSeek V4 Pro as implementer                           | Qwen 3 Coder, GLM 5.2                                 | When Flash struggles twice                     |
| S6 (XS async)                     | Claude Code on the web / Routines                        | Copilot coding agent, Codex Web, OpenCode backgrounded | Lightweight: manual is fine for XS            |
| S6 (batch refactor)               | `claude -p` / `opencode -p` loop over file list          | Aider batch mode, Devin (if multi-week)               | See [multi-agent.md](multi-agent.md)           |
| S7 Review (human)                 | You + 10-min rubric                                      | —                                                     | **Never skip**                                 |
| S7 Review (conformance)           | `converge` skill (OS-native)                             | Spec Kit `/speckit.converge`                          | XS tickets                                     |
| S7 Review (always-on AI)          | CodeRabbit                                               | —                                                     | —                                              |
| S7 Review (2nd AI, uncorrelated)  | Codex review (C36 standing config)                       | OpenRouter → GLM 5.2, claude-code-action via Bedrock  | Lightweight: skip if PR ≤50 lines              |
| S7 Review (3rd AI, OpenAI family) | Copilot code review (Business)                           | —                                                     | Lightweight: skip                              |
| S7 Review (runtime validation)    | `converge` tests-as-evidence                             | Greptile TREX (subscription)                          | Skip if PR is pure logic with green CI         |
| S8 Release                        | CI/CD (Vercel/Cloudflare)                                | —                                                     | —                                              |
| S9 Learn                          | OpenCode + Gemini 2.5 Flash                              | DeepSeek V4 Flash                                     | Lightweight: skip graph update                 |

### Model routing defaults

Costs are **per session** (single-call cost × the 10–50× agentic multiplier, cache-adjusted):

| Task                       | Model                                  | Cost / session    | Why                                                                                |
| -------------------------- | -------------------------------------- | ----------------- | ---------------------------------------------------------------------------------- |
| Spec generation (S2)       | **DeepSeek V4 Pro**                    | ~$0.30–1.00       | Intelligent planning, ~16× cheaper per token than Opus 4.8                         |
| Architecture (S4)          | **DeepSeek V4 Pro**                    | ~$0.50–1.50       | Deep trade-off analysis; 1M context fits whole repo                                |
| Ticket decomposition (S5)  | **DeepSeek V4 Pro**                    | ~$0.20–0.60       | Reads real files, structured plans                                                 |
| Implementation (S6)        | **DeepSeek V4 Flash**                  | ~$0.10–0.50/ticket | ~20× cheaper per token than Sonnet 5; fast, tool calls, FIM                       |
| Hard ticket fallback (S6)  | GLM 4.5 Air, Qwen 3 Coder              | ~$0.20–0.80/ticket | Different family when Flash struggles                                             |
| Self-review (C22, S6)      | DeepSeek V4 Flash (author's own model) | ~$0.01            | Self-review is the author's pass by definition                                     |
| AI review always-on (S7)   | CodeRabbit (Free tier; Pro if outgrown) | **$0** ✅ (Pro $24–30/mo) | Multi-model, always-on, path instructions + learnings. Free covers solo volume |
| AI review 2nd opinion (S7) | **C36 cross-family**: Codex review (default stack) / GLM 5.2 via OpenRouter (open-frontier) | ChatGPT sub / ~$0.02/review | Different family from author — uncorrelated |
| AI review 3rd opinion (S7) | Copilot code review (Business)         | $19/user/mo       | Third family                                                                       |
| Spec conformance (S7)      | `converge` skill on DeepSeek V4 Pro    | ~$0.10–0.30/run   | Frozen ACs + plan vs diff, tests as evidence                                       |
| Runtime validation (S7)    | Greptile TREX                          | subscription      | Sandboxed execution; optional upgrade                                              |
| Learning/dumps (S9)        | Gemini 2.5 Flash                       | ~$0.002/dump      | Cheapest viable, 1M context                                                        |

#### The three-family rule

For any PR >50 lines, review spans **three different model families**: the
author's family is covered by CodeRabbit's multi-model pipeline, the second
opinion is cross-family per C36 (Codex review), the optional third is a third
family (GLM/Llama via OpenRouter, or Copilot code review). Three families, no
shared blind spots.

#### The cache hit principle

DeepSeek cache hit input is **50× cheaper** than miss ($0.0028 vs $0.14 per
1M). Keep `AGENTS.md` and FSD sections stable; use the same model within a
ticket's lifecycle; never inject timestamps into the prompt prefix. Track
cache-hit ratio weekly (cost-control.md).

### Setup checklist

- [ ] **DeepSeek account** — top up $20 at platform.deepseek.com; test API key ✅
- [ ] **OpenRouter account** — top up $10; verify GLM 5.2 routing
- [ ] **OpenCode installed** — configure `opencode.json` (sketch above)
- [ ] **Aider installed** (optional) — `pip install aider-chat`
- [ ] **Cline installed** (optional, IDE layer) — VS Code marketplace
- [ ] **CodeRabbit GitHub App** on all repos; drop in
      [github/.coderabbit.yaml](github/.coderabbit.yaml)
- [ ] **Codex CLI** — the standing C36 cross-family reviewer; worth it if you
      already pay for ChatGPT
- [ ] **GitHub Copilot Business** (optional, third reviewer) — $19/user/mo
- [ ] **`gh` CLI authenticated**
- [ ] **Spec Kit CLI** (optional) — `uv tool install specify-cli`
- [ ] **Greptile TREX** (optional, runtime validation on repos where runtime
      bugs are expensive)
- [ ] **claude-code-action** (optional, Bedrock/DeepSeek-routed uncorrelated review)
- [ ] **Claude Code** (optional) — for async lanes (Routines, web) or
      Anthropic-specific features; `ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic`
      for the DeepSeek backend

#### Monthly budget estimate

| Item                                                            | Cost            |
| --------------------------------------------------------------- | --------------- |
| DeepSeek API (15–20 tickets, planning + implementation)         | $15–40          |
| OpenRouter (GLM 5.2 + Llama 4 reviews)                          | $5–10           |
| CodeRabbit (Free tier covers solo; Pro $24–30 if outgrown)      | $0–30 ✅        |
| GitHub Copilot Business (optional, third reviewer)              | $19             |
| ChatGPT Plus (for Codex review, if not already subscribed)      | $20             |
| Greptile (optional)                                             | ~$20 ❔         |
| **Total (default profile)**                                     | **~$20–80/mo**  |
| **Total (with all optional layers)**                            | **~$79–139/mo** |

Compared against the default stack at **~$220–264/mo**
([cost-control.md](cost-control.md)), **the open-frontier stack is 70–90%
cheaper at near-equivalent quality for most tasks** — and the floor is now
lower than previously published, because CodeRabbit's free tier covers solo
volume. (Mid-2025 baseline for reference: ~$232–249/mo.)

### What changed from the mid-2025 matrix

| Mid-2025                                        | July 2026                                                                                     |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Three-category market (IDE/CLI/cloud)           | **Seven-category market** (control centers, CLI, vendor CLI, IDE, cloud async, review, vibe)  |
| "Claude Code has no async fire-and-forget"      | Routines, Channels, Claude Code on the web, agent teams — async is native everywhere          |
| Claude Code as the only primary harness ($200/mo Max)    | **Two named stacks**: default (Claude Code + Anthropic) or open-frontier (OpenCode + DeepSeek, $0 harness) — pick per repo |
| Claude Sonnet/Opus for all stages               | DeepSeek V4 Pro planning, V4 Flash implementation                                             |
| Codex CLI as second-opinion implementer         | Codex review as standing C36 cross-family reviewer; GLM/Llama via OpenRouter as alternates    |
| Copilot Enterprise for Code Review ($39 + GHEC) | Copilot Business ($19, no GHEC) — and now a 12-product platform family                        |
| Code review as one column                       | **Review sub-matrix** (tiers, attribution, follow-through, runtime validation)                |
| CodeRabbit as PR bot                            | CodeRabbit as review + post-merge follow-through platform                                     |
| No Devin / vibe-coding rows                     | Devin (long-horizon delegation) and vibe-coding (S1 prototype lane) documented                |
| Traycer listed                                  | Dropped — no 2026 evidence of continued investment ❔                                         |
| Same-vendor author+review accepted              | Constitution C36 — reviewer independence                                                      |
| No confidence tiers                             | Every load-bearing fact tagged ✅/📣/❔ with fetch dates                                      |
| ~$232–249/mo total                              | ~$20–80/mo default; ~$79–139/mo with all optional layers (CodeRabbit free tier lowered the floor) |

### What to _not_ change

- **CLI agents as primary harness.** Terminal agents read real files, run real
  tests, leave audit trails. Only the _model behind the harness_ changed.
- **Narrow context per ticket.** 1M-context models don't change the rule —
  broad context still degrades quality. Use the big window for specs, not for
  cramming the whole repo into S6.
- **400-line PR cap. WIP limit 2. Human review first. The spec is the source
  of truth.** All unchanged; the three-family AI stack is _after_ the human
  rubric, not instead of it.

---

_Sources: DeepSeek pricing verified at api-docs.deepseek.com/quick_start/pricing
(fetched 2026-07-19). OpenRouter docs at openrouter.ai/docs (fetched
2026-07-19). docs.anthropic.com + anthropic.com/news (fetched 2026-07-19).
github.com/openai/codex (fetched 2026-07-19). docs.github.com/copilot +
github.blog (fetched 2026-07-19). cursor.com/blog (fetched 2026-07-19).
coderabbit.ai/blog (fetched 2026-07-19). devin.ai (fetched 2026-07-19).
replit.com/ai + lovable.dev (fetched 2026-07-19). github.com/Aider-AI/aider
(fetched 2026-07-19). github.com/github/spec-kit v0.13.0.
github.com/bmad-code-org/BMAD-METHOD v6.10.0. Full evidence trail:
research/research-2026-07-*.md. Re-verify after 2026-10-19._
