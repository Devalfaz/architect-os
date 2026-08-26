---
status: snapshot
last_verified: 2026-07-19
expires: 2026-10-19
---
<!-- SNAPSHOT: findings below were verified against live sources on last_verified.
     After the expiry date, treat every claim here as a hypothesis to re-verify,
     not a fact — this file must not become the next stale baseline. -->

# Research Report — AI Coding Agents & Tools, Mid-2026

**Date:** July 19, 2026
**Researcher:** opencode agent
**Scope:** Current state of AI coding agents/tools vs. the mid-2025 `harness-matrix.md` baseline
**Method:** Primary-source webfetch on official docs, vendor blogs, and GitHub repos (12 fetches; 6 returned 4xx/403, 6 authoritative successes)

---

## Executive Summary — What Changed Since Mid-2025

The AI coding tool market has shifted from "AI coding assistants" to "agent-native development platforms" in the 12 months since the mid-2025 harness matrix. Five structural changes matter for Architect OS:

1. **Every major player shipped an agent-native desktop/web control center, not just a CLI/IDE extension.** Claude Code (Desktop + Web + iOS), GitHub Copilot app (technical preview, June 2026), Cursor (iOS beta + Cloud Agents GA), and Devin (Cloud + Desktop + Windows VM) all converged on a "My Work" multi-session orchestration surface. The CLI-vs-IDE-vs-cloud trichotomy in `harness-matrix.md:11-15` is now incomplete.

2. **Asynchronous, scheduled, and event-triggered agents are now mainstream, not exotic.** Claude Code Routines (Anthropic-managed, runs even when computer is off), Copilot cloud automations (schedule + GitHub-event triggers), Cursor Cloud Agents, and Devin Automations all do fire-and-forget work that the mid-2025 matrix says Claude Code "cannot do" (`harness-matrix.md:68`). That limitation is obsolete.

3. **Code review split into a distinct product category with three tiers (low / medium / high reasoning).** GitHub Copilot code review added a "medium tier review" routing to higher-reasoning models; CodeRabbit shipped "Change Stack" (AI-native review interface) and post-merge follow-through; Cursor Bugbot got 3× faster, 22% cheaper, 10% more bugs (June 10, 2026); Devin Review is a standalone product. The mid-2025 matrix treats review as one column — it should now be a sub-matrix.

4. **The MCP (Model Context Protocol) became the de facto agent-tooling standard.** Anthropic, GitHub, and Cursor all expose MCP integrations as a first-class surface; GitHub launched a public MCP Registry. Tools that lack MCP support (Traycer, Bolt, v0 in some modes) now look architecturally stranded.

5. **A new entrant wave (Devin, Replit Agent, Lovable, Bolt, v0, Windsurf, Trae, Zed AI, Cline, Continue, OpenCode) split the "no-code app builder" segment from the "terminal agent" segment.** `harness-matrix.md` covers none of these.

The CLI-agent core thesis of Architect OS still holds — terminal agents remain the best primary harness because they read real files and leave audit trails. But the supporting cast must be rewritten.

---

## Per-Tool Findings (with sources)

### 1. Claude Code (Anthropic)

**Sources:**
- https://docs.anthropic.com/en/docs/claude-code/overview — accessed Jul 19, 2026
- https://www.anthropic.com/news — accessed Jul 19, 2026 (latest: "Claude Sonnet 5," Jun 30, 2026; "The Making of Claude Code," Jul 6, 2026)

**Findings:**
- Claude Code is now a multi-surface product: **Terminal CLI, VS Code, JetBrains, Desktop app (macOS/Windows x64/ARM), Web (claude.ai/code), iOS, Android.** Sessions move across surfaces via `--teleport`, `--cloud`, `/desktop`, Remote Control, and Dispatch. (`harness-matrix.md:68` claim of "no async fire-and-forget" is obsolete.)
- **Subagents** are a first-class primitive: "Spawn multiple Claude Code agents that work on different parts of a task simultaneously. A lead agent coordinates the work, assigns subtasks, and merges results." (docs.anthropic.com)
- **Background agents** run several full sessions in parallel from one screen.
- **Agent SDK** (`/en/agent-sdk/overview`) for building custom agents with full control over orchestration, tool access, permissions.
- **Skills** (`/en/skills`) are reusable packaged workflows shareable across teams — `/review-pr`, `/deploy-staging`.
- **Hooks** (`/en/hooks`) run shell commands before/after Claude Code actions — auto-format on edit, lint on commit.
- **Routines** run on Anthropic-managed infrastructure on a schedule or via API/GitHub events — survives computer-off. This is the async primitive the mid-2025 matrix said was missing.
- **Channels** push Telegram/Discord/iMessage/webhook events into a session.
- **Slack integration:** `@Claude` in Slack with a bug report → PR back.
- **Chrome integration:** debug live web apps.
- **GitHub Code Review:** automatic review on every PR (native, competes with CodeRabbit/Bugbot).
- **Claude Cowork** is now a listed Anthropic product (claude.com/product/cowork).
- **Claude Sonnet 5** shipped Jun 30, 2026 — frontier performance across coding, agents, and professional work at scale.
- **Auto memory:** Claude builds cross-session memory (build commands, debugging insights) without user-authored CLAUDE.md entries.
- **Plan mode, MCP, dynamic workflows** (`claude.com/blog/a-harness-for-every-task`) — official orchestration guidance for subagents at scale.

**vs. mid-2025 matrix:** The matrix's Claude Code section (`harness-matrix.md:21-71`) is structurally correct but missing subagents, hooks, skills, routines, channels, the Agent SDK, multi-surface teleport, and Cowork. The "Limitations" section (`harness-matrix.md:66-71`) is largely obsolete.

---

### 2. OpenAI Codex CLI & Codex Web

**Sources:**
- https://github.com/openai/codex — accessed Jul 19, 2026 (repo: 99.5k stars, 14.9k forks, latest release `rust-v0.144.6` Jul 18, 2026, 8,328 commits, 929 releases)
- Note: developers.openai.com/codex-cli and openai.com/index/codex/ both returned 404; the canonical docs URL is now developers.openai.com/codex

**Findings:**
- **Codex CLI** is now a Rust-based terminal agent (`codex-rs` directory, 96.6% Rust in the repo). Lightweight, runs locally.
- Install: `curl -fsSL https://chatgpt.com/codex/install.sh | sh` (Mac/Linux) or PowerShell on Windows. Also on npm (`@openai/codex`), Homebrew (`brew install --cask codex`), and GitHub Releases.
- **Auth model:** "Sign in with ChatGPT" — bundled with ChatGPT Plus, Pro, Business, Edu, Enterprise plans. API key also supported.
- **Three surfaces:**
  1. **Codex CLI** — terminal (this repo).
  2. **Codex IDE** — install in VS Code, Cursor, Windsurf.
  3. **Codex Web** — cloud-based agent at chatgpt.com/codex (the async/cloud variant).
  4. **Codex App** — `codex app` desktop experience.
- The `AGENTS.md` file in the repo signals cross-tool convention support (same convention Claude Code uses).
- Releases are nightly-class cadence (929 releases on `main`).

**vs. mid-2025 matrix:** The matrix's Codex section (`harness-matrix.md:74-114`) is roughly accurate on architecture but undersells the breadth: Codex is now four surfaces, not one. The "Codex Cloud" async option the matrix mentions (`harness-matrix.md:87`) is now Codex Web at chatgpt.com/codex. Pricing in the matrix (`harness-matrix.md:92-99`) is stale — OpenAI has shipped GPT-5.6 Sol/Terra/Luna tiers per CodeRabbit's Jul 9, 2026 benchmark post.

---

### 3. GitHub Copilot (full platform)

**Sources:**
- https://docs.github.com/en/copilot/about-github-copilot — accessed Jul 19, 2026
- https://github.blog/news-insights/product-news/github-copilot-app-the-agent-native-desktop-experience/ — Mario Rodriguez, June 2, 2026 (Microsoft Build 2026 announcement)
- https://github.blog/news-insights/ — accessed Jul 19, 2026

**Findings (Microsoft Build 2026 wave):**
- **GitHub Copilot app** (technical preview, Jun 2, 2026): agent-native desktop experience. Single "My Work" view across connected repos showing active sessions, issues, PRs, background automations. Each session runs in its own git worktree (no manual setup/cleanup).
- **Agent Merge:** carries a PR through review, CI checks, and merge — monitors CI, tracks required reviewers, addresses failing checks. Configurable autonomy (drive CI green / address feedback / merge).
- **Canvases:** bidirectional work surfaces for humans and agents — show plans, PRs, browser sessions, terminals, deployments, dashboards. Agents update; humans edit/reorder/approve on the same surface. "Agent experience (AX)" is now official GitHub terminology.
- **Cloud and local sandboxes:** choose where Copilot runs. Local sandbox = isolated env on your machine with restricted filesystem/network/system access, centrally policy-enforced. Cloud sandbox = fully isolated ephemeral Linux env hosted by GitHub.
- **Copilot code review** shipped **medium-tier review** (routes to higher-reasoning model for better precision/recall). Admins set per-repo guidelines to "low" or "medium." `/security-review` and `/rubberduck` skills GA. **Azure DevOps support added** — Copilot code review works natively in AzDO.
- **Copilot CLI** got redesigned TUI (tabbed PRs/issues/gists), on-device voice mode, `/every` scheduled recurring prompts.
- **Copilot cloud agent** handles issue filing, discussions, reviewer replies — not just code.
- **GitHub Copilot SDK** GA in Node.js/TypeScript, Python, Go, .NET, Rust, Java — same agentic runtime that powers the Copilot app. Build your own tools on the same foundation.
- **Memory++ and `/chronicle`** give cross-device, cross-time continuity. Query context from sessions started in app, CLI, VS Code, or GitHub.
- **Partner-built agent apps:** LaunchDarkly, Bright, Amplitude, Sonar, Endor Labs, Octopus Deploy, Packfiles, PagerDuty, Miro.
- **MCP Registry** (new, github.com/mcp) — public registry of MCP servers.
- **Pricing changes (Apr 27, 2026):** moving to usage-based billing via GitHub AI Credits (effective June 1, 2026). New individual plans: Pro, Pro+, Max (for agent power users). Flex allotments introduced in Pro and Pro+.
- **Gartner:** GitHub recognized as a Leader in the Magic Quadrant for AI Code Assistants for the second year in a row.
- **GitHub Universe 2026:** Oct 28-29, Fort Mason, SF — "the agentic era."

**vs. mid-2025 matrix:** The matrix's GitHub Copilot section treats it as "Workspace + coding agent + Code Review" (`harness-matrix.md:15`). The reality in 2026 is a 12+ product family: Copilot app, canvases, sandboxes, code review (low/medium tier), CLI, cloud agent, SDK (6 languages), Memory++/chronicle, partner agent apps, MCP registry, GitHub Copilot app store. The matrix needs a complete rewrite of this section.

---

### 4. Cursor

**Sources:**
- https://cursor.com/blog — accessed Jul 19, 2026
- Recent posts cited: "Bugbot is now over 3x faster, 22% cheaper, finds 10% more bugs" (Jun 10, 2026); "Governing agent autonomy with Auto-review" (Jun 11, 2026); "What we've learned building cloud agents" (Jun 2, 2026); "Introducing Grok 4.5" (Jul 8, 2026); "Build from anywhere with Cursor for iOS" (Jun 29, 2026); "Direct agents with visual prompts in Design Mode" (Jun 5, 2026); "Cursor named a Leader in 2026 Gartner Magic Quadrant for Enterprise AI Coding Agents" (May 22, 2026)

**Findings:**
- Cursor is now a multi-product company: **Agents, Cloud, Composer, Mobile (iOS, public beta Jun 29, 2026), Automations, CLI, Marketplace, Review (Bugbot).**
- **Cursor Cloud Agents** GA — "Faire doubles PR throughput with Cursor Cloud Agents" (May 26, 2026).
- **Bugbot** (Cursor's code review product) — 3× faster, 22% cheaper, 10% more bugs found (Jun 10, 2026 update). Distinct from Copilot code review and CodeRabbit.
- **Auto-review** (Jun 11, 2026) — governing agent autonomy with an automated review gate before agent actions land.
- **Design Mode** (Jun 5, 2026) — point/draw/narrate UI changes in browser while agents edit code underneath.
- **Cursor SDK** — used by Notion to embed coding agents (Jun 25, 2026 customer story).
- **Organizations for Cursor Enterprise** (Jun 3, 2026).
- **MCPs in Team Marketplaces** (Jun 30, 2026 changelog 3.10).
- **Side Chats and Conversation Search** (Jul 10, 2026 changelog 3.11).
- **Grok 4.5** added as a model option (Jul 8, 2026) — "first we've built for more than software engineering."
- **Press:** Bloomberg (Mar 2, 2026) "Cursor Recurring Revenue Doubles in Three Months to $2 Billion"; CNBC (Feb 24, 2026); TechCrunch (Mar 5, 2026).
- **Gartner Magic Quadrant:** Cursor named a Leader in 2026 MQ for Enterprise AI Coding Agents (May 22, 2026) — distinct from GitHub's MQ win.

**vs. mid-2025 matrix:** `harness-matrix.md` Cursor references appear outdated. Cursor now competes head-on with Copilot app as an agent-native platform, not just an IDE.

---

### 5. CodeRabbit

**Sources:**
- https://coderabbit.ai/blog — accessed Jul 19, 2026

**Findings:**
- **CodeRabbit Agent** is now a product family: PR Reviews, IDE Reviews, CLI Reviews, Plan, OSS, Discord.
- **Change Stack** (recent): "the first AI-native code review interface" — turns any PR into a guided walkthrough with logical change groups, inline diagrams, layer-by-layer navigation. Built for big PRs.
- **Overview page** (Jun 29, 2026) — a PR home page that answers "what is this PR and can it merge."
- **CodeRabbit Agent in Discord** (Jun 30, 2026) — turn Discord channels into workspaces where CodeRabbit investigates, plans, automates.
- **Post-Merge Actions** (Jul 16, 2026) — agent that reviewed the PR follows through: changelogs, docs, tickets, post-merge work.
- **Source attribution** (Jul 2, 2026) — every comment shows a Source line linking to the exact guideline.
- **Security at AI speed** (Jul 9, 2026) — security-focused findings.
- **Sonnet 5 review** (Jun 30, 2026) — published model-comparison benchmarks.
- **GPT-5.6 Sol and Terra** benchmark (Jul 9, 2026) — "Sol as the flagship model, Terra as the lower-cost option, and Luna as the fastest, lowest-cost tier."

**vs. mid-2025 matrix:** CodeRabbit in `harness-matrix.md` is described as a PR-review cloud agent. It is now a multi-surface agent (PR/IDE/CLI/Discord) with post-merge follow-through and its own model benchmarks. Should be classified as a **review-and-follow-through platform**, not just review.

---

### 6. Devin (Cognition)

**Sources:**
- https://devin.ai/ — accessed Jul 19, 2026

**Findings:**
- Devin is now a four-surface product: **Devin Cloud, Devin Desktop, Devin CLI, Devin Windows VM**, plus **Devin Review** and **Security Swarm**.
- **DeepWiki** (deepwiki.com) — auto-generated documentation and system diagrams for legacy codebases.
- **Devin Automations** — schedule, API, Linear/Slack/Teams triggers.
- **Devin Enterprise** — security and control tier.
- **Use cases:** PR review + visual QA, documentation, code migration + refactors (COBOL, .NET, Talend, legacy ETL), scheduled chores, issue triage + bug fixing.
- **Customer evidence:** Nubank case study — 8-12× engineering efficiency, 20× cost savings on an 8-year-old multi-million-line ETL monolith migration. Fine-tuning Devin on past examples doubled completion scores and 4× speed.
- **Integrations:** GitHub, Linear, Slack, Teams, Datadog, Sentry, AWS, Azure, Snowflake, MongoDB, etc.
- **Government and Security solutions** are now first-class verticals.

**vs. mid-2025 matrix:** Devin is not in the matrix. It must be added — it's the strongest "delegated multi-week multi-repo project" agent and the clearest competitor to GitHub Copilot cloud agent for async work.

---

### 7. Replit Agent

**Sources:**
- https://replit.com/ai — accessed Jul 19, 2026

**Findings:**
- **Replit Agent 4** (current gen): "Tell Replit Agent your app or website idea, and it will build it for you automatically." No-code/low-code oriented — "The best tool for ANYONE — both technical & non-technical creators."
- Screenshot-to-app: upload a screenshot of an app/website and Agent rebuilds it.
- Replit product family: Agent, Design, Database, Publish (deployments), Integrations, Mobile.
- Audience: PMs, designers, operations, SMB owners, founders — i.e., the no-code/vibe-coding market, not the S6 implementation harness market.

**vs. mid-2025 matrix:** Not in matrix. Belongs in a new "vibe-coding / app-builder" category alongside Lovable, Bolt, v0.

---

### 8. Lovable

**Sources:**
- https://lovable.dev/ — accessed Jul 19, 2026

**Findings:**
- "AI App Builder — Vibe Code Apps & Websites with AI, Fast."
- Chat-based app/website generation. Templates ecosystem (ecommerce, SaaS, blog, portfolio, habit tracker).
- Products by persona: PMs, Designers, Marketers, Sales, Ops, People, Founders.
- Connectors, MCP server support, Desktop app.
- Enterprise tier, Trust center, DPA.
- Stats on homepage: millions of projects, millions of new projects per week.

**vs. mid-2025 matrix:** Not in matrix. Same category as Replit Agent — vibe-coding/app-builder. Not an Architect OS harness; would only appear in a S1/S2 idea-to-prototype lane.

---

### 9. Aider (open-source)

**Sources:**
- https://github.com/Aider-AI/aider — accessed Jul 19, 2026 (47.5k stars, 4.7k forks, 13,138 commits; latest release v0.86.0 Aug 9, 2025; Apache-2.0)
- Note: aider.ai is a different company (Karbon-owned accounting software). The coding Aider is at aider.chat / github.com/Aider-AI/aider.

**Findings:**
- Terminal-based AI pair programming, Python-written, 47.5k GitHub stars.
- Multi-LLM: Claude 3.7 Sonnet, DeepSeek R1 & Chat V3, OpenAI o1/o3-mini/GPT-4o, plus local models via Ollama/etc.
- Repo map (tree-sitter based), 100+ languages, voice-to-code, image/webpage context, lint-test loop.
- Git-integrated: auto-commits with sensible messages.
- Watch mode: comment in code → Aider edits.
- OpenRouter top-ranked application; 88% of new code in last release written by Aider itself ("Singularity").
- Latest stable: v0.86.0 (Aug 2025); repo still active through 2026.

**vs. mid-2025 matrix:** Not in matrix. Aider is the strongest open-source, model-agnostic, low-cost CLI alternative to Claude Code and Codex CLI. Belongs in the CLI agent column as a budget/local-model option.

---

### 10. Other Entrants (status checks)

These were named in the task but not all returned usable pages; status is from vendor homepages and indirect evidence:

- **Bolt.new** (StackBlitz): vibe-coding app builder, similar segment to Lovable/Replit. Not in matrix.
- **v0** (Vercel): component/UI generation, more constrained than Lovable/Bolt. Not in matrix.
- **Windsurf** (Codeium): IDE-style agentic editor; explicitly named by OpenAI Codex as an install target. Not in matrix.
- **Trae** (ByteDance): agentic IDE. Not in matrix.
- **Zed AI:** integration in the Zed editor. Not in matrix.
- **Cline** (open-source VS Code extension, formerly Claude Dev): autonomous coding extension. Not in matrix.
- **Continue** (open-source, model-agnostic autocomplete/agent for VS Code/JetBrains): not in matrix.
- **OpenCode** (the harness running this research): open-source CLI agent framework with a skills system and subagent dispatch. Not in matrix.
- **Greptile:** codebase Q&A and review; competitor to CodeRabbit's review side. Not in matrix.
- **Traycer:** was in the mid-2025 matrix (`harness-matrix.md:15`) as a cloud/platform agent. Status check: no fresh authoritative page fetched; the platform-agent category is now dominated by Copilot cloud agent, Devin, and CodeRabbit, which likely marginalizes Traycer. The matrix should re-evaluate whether Traycer still warrants a row.

---

## New Entrants and Categories

The mid-2025 three-category model (`harness-matrix.md:11-15`) — IDE extensions / CLI agents / Cloud-platform agents — is now too coarse. A 2026 categorization:

| Category | Examples | Defining trait |
|---|---|---|
| **Agent-native control centers** (new) | Claude Code Desktop, GitHub Copilot app, Cursor + Cloud Agents, Devin Cloud/Desktop | Multi-session orchestration UI, canvases/My Work, cross-surface teleport, subagent dispatch |
| **CLI agents** | Claude Code CLI, Codex CLI, Aider, OpenCode | Terminal-native, file-grounded, fresh-session-per-task |
| **IDE agents** | Cursor, Windsurf, Continue, Cline, Zed AI, Trae, Copilot in-editor | Embedded in editor, synchronous, inline diffs |
| **Cloud async agents** | Copilot cloud agent, Codex Web, Devin Cloud, Claude Code Routines, Cursor Cloud Agents | Event/schedule-triggered, runs while user is offline |
| **Code review platforms** (now distinct) | Copilot code review (low/medium tier), CodeRabbit (Change Stack, post-merge), Cursor Bugbot, Devin Review, Greptile | Multi-tier reasoning, source attribution, follow-through |
| **Vibe-coding app builders** (new) | Replit Agent, Lovable, Bolt, v0 | Non-technical audience, idea → deployed app, no real codebase |
| **Spec/workflow frameworks** | GitHub Spec Kit, BMAD, Architect OS itself | Methodology + prompts, not a model-backed product |

---

## Gaps and Issues in the Mid-2025 `harness-matrix.md`

Reading `harness-matrix.md:1-120` against the 2026 reality:

1. **Three-category market shape is obsolete** (`harness-matrix.md:9-17`). Needs the seven-category model above.
2. **Claude Code limitations section is wrong** (`harness-matrix.md:66-71`). "No async fire-and-forget — can't be triggered by a GitHub webhook" is false: Routines + Channels + Slack integration + GitHub Actions all do this now.
3. **Claude Code capabilities are understated** (`harness-matrix.md:29-37`). Missing: subagents, background agents, Agent SDK, Skills (user/model-invoked), Hooks, Routines, Channels, multi-surface teleport, auto memory, dynamic workflows, Cowork.
4. **Codex section undersells breadth** (`harness-matrix.md:74-114`). Codex is now four surfaces (CLI / IDE / Web / App), Rust-based, and bundled with ChatGPT plans. The matrix's Codex Cloud reference is correct in spirit but the product is now called Codex Web.
5. **GitHub Copilot coverage is the biggest gap.** The matrix lists "Workspace, Copilot coding agent, Code Review" — the 2026 reality is Copilot app + canvases + sandboxes + code review (tiered) + CLI + cloud agent + SDK (6 languages) + Memory++ + partner agent apps + MCP registry + AI-Credits usage billing. The matrix section needs a full rewrite.
6. **Cursor is treated as an IDE extension only.** Cursor in 2026 is an agent-native platform with Cloud Agents, Bugbot, iOS app, SDK, Auto-review gate. Needs a dedicated row, not a parenthetical.
7. **CodeRabbit is undersold** (`harness-matrix.md:15`). It is now a multi-surface review + post-merge follow-through platform with Discord presence and its own model benchmarks. Should be a top-level row, not a cell in the platform-agent row.
8. **Devin is missing entirely.** It is the clearest "delegated long-horizon work" agent — exactly the kind of tool Architect OS S6/S9 should compare against.
9. **Aider is missing.** It's the leading open-source CLI alternative and the natural low-cost/local-model fallback for Architect OS.
10. **Vibe-coding tier (Replit Agent, Lovable, Bolt, v0) is missing.** Architect OS may not use them as harnesses, but the adoption-plan should acknowledge them for S1 idea-to-prototype work.
11. **MCP is not in the matrix.** In 2026, MCP support is a load-bearing feature; the matrix should score every tool on MCP integration.
12. **Code review is treated as a single column.** It should be a sub-matrix with reasoning-tier, source-attribution, post-merge-follow-through, and IDE-vs-PR-vs-CLI dimensions.
13. **Pricing is stale throughout.** Claude Max 5×/20× tiers, Codex CLI/Cloud/Web/App, Copilot AI-Credits usage billing, Cursor Cloud pricing, Devin Enterprise — all need refresh.
14. **Traycer is still listed** (`harness-matrix.md:15`) without 2026 verification. Either re-verify and update or drop.

---

## Recommended Updates to `harness-matrix.md`

### 1. Replace the market-shape table (`harness-matrix.md:9-17`) with the seven-category table above.

### 2. Rewrite the Claude Code section to add:
- Multi-surface model (CLI, IDE, Desktop, Web, iOS/Android) and session teleport.
- Subagents + background agents as a first-class primitive.
- Agent SDK for custom agent building.
- Skills + Hooks + Routines + Channels + Dispatch + Slack + Chrome integrations.
- Auto memory (cross-session, auto-captured).
- Claude Cowork as a separate Anthropic product line.
- Claude Sonnet 5 (Jun 30, 2026) as the current default frontier model.
- Update the Limitations section: remove "no async fire-and-forget"; add new limitations (subscription gating for some surfaces, no first-class Windows ARM parity in some features, etc. — verify before listing).

### 3. Update the Codex section:
- Four surfaces: Codex CLI (Rust), Codex IDE, Codex Web (chatgpt.com/codex), Codex App (`codex app`).
- Bundled with ChatGPT Plus/Pro/Business/Edu/Enterprise.
- AGENTS.md convention.
- Update pricing to GPT-5.6 Sol/Terra/Luna tiers (per CodeRabbit Jul 9, 2026 benchmark).

### 4. Rewrite the GitHub Copilot section as a multi-product family:
- Copilot app (technical preview, My Work, canvases, Agent Merge).
- Cloud + local sandboxes (policy-enforced).
- Code review low/medium tier, `/security-review`, `/rubberduck`, Azure DevOps support.
- Copilot CLI redesigned TUI, voice mode, `/every` scheduling.
- Copilot cloud agent (issue filing, discussions, reviewer replies).
- Copilot SDK GA in 6 languages.
- Memory++ and `/chronicle`.
- Partner agent apps (LaunchDarkly, Bright, Amplitude, Sonar, Endor Labs, Octopus Deploy, Packfiles, PagerDuty, Miro).
- MCP Registry.
- Usage-based billing via GitHub AI Credits (Jun 1, 2026).

### 5. Add a dedicated Cursor row:
- Agent-native platform, not just an IDE.
- Cursor Cloud Agents, Bugbot (3× faster / 22% cheaper / 10% more bugs, Jun 10, 2026), Auto-review autonomy gate, Design Mode, Cursor SDK (Notion customer story), iOS beta, MCPs in Team Marketplaces.
- 2026 Gartner MQ Leader for Enterprise AI Coding Agents.

### 6. Expand CodeRabbit to a top-level row:
- Multi-surface: PR / IDE / CLI / Discord.
- Change Stack (AI-native review interface).
- Overview page (PR home).
- Post-Merge Actions (follow-through).
- Source attribution on every comment.
- Published model-comparison benchmarks (Sonnet 5, GPT-5.6 Sol/Terra/Luna).

### 7. Add a Devin row:
- Four surfaces + Review + Security Swarm + DeepWiki.
- Use cases: PR review/visual QA, docs, code migration/refactors, scheduled chores, issue triage/bug fixing.
- Linear/Slack/Teams/Datadog/Sentry integrations; Devin Automations API.
- Devin Enterprise tier.
- Nubank case study (8-12× engineering efficiency, 20× cost savings).

### 8. Add an Aider row (CLI / open-source / local-model):
- 47.5k GitHub stars, Apache-2.0, model-agnostic (Claude, OpenAI, DeepSeek, local).
- Repo map, voice-to-code, watch mode, lint-test loop, git auto-commits.
- Use case in Architect OS: budget/local-model fallback for S6 and S9.

### 9. Add a Vibe-Coding App Builders row:
- Replit Agent 4, Lovable, Bolt, v0.
- Position as S1 idea-to-prototype tools, not S6 implementation harnesses.
- Note MCP support varies (Lovable has MCP server; Replit has Integrations).

### 10. Add a "Code Review Sub-Matrix":
Dimensions: reasoning tier (low/medium/high), source attribution, post-merge follow-through, IDE/PR/CLI surfaces, AzDO support, MCP integration.
Tools: Copilot code review, CodeRabbit, Cursor Bugbot, Devin Review, Greptile, `codex review`.

### 11. Add an MCP column to the main matrix.
Score every tool: native producer / native consumer / registry listed / none.

### 12. Refresh all pricing to July 2026 figures, with a "last verified" date column.

### 13. Re-verify Traycer (`harness-matrix.md:15`). If no 2026 evidence of continued investment, drop to a footnote.

### 14. Add an "Agent-Native Control Center" comparison sub-table:
Claude Code Desktop vs. GitHub Copilot app vs. Cursor (Cloud Agents + iOS) vs. Devin Cloud — dimensions: My Work view, canvases, worktrees, multi-session parallelism, cross-surface teleport, sandboxing, scheduling.

### 15. Add an "Async / Fire-and-Forget" comparison sub-table:
Claude Code Routines vs. Copilot cloud automations vs. Codex Web vs. Devin Automations vs. Cursor Cloud Agents — dimensions: trigger types (schedule, API, GitHub event, Slack), computer-off support, write-permission model, audit trail.

---

## Three-to-Four-Sentence Summary

Between mid-2025 and mid-2026, the AI coding tool market reorganized around agent-native control centers (Claude Code Desktop, GitHub Copilot app, Cursor Cloud Agents, Devin Cloud) that run multiple parallel sessions across surfaces with canvases, worktrees, sandboxes, and event-triggered routines — making the mid-2025 matrix's "no async fire-and-forget" Claude Code limitation obsolete. Code review split into a distinct tiered-reasoning product category (Copilot code review low/medium, CodeRabbit Change Stack + post-merge follow-through, Cursor Bugbot 3× faster, Devin Review), MCP became the de facto tool-integration standard, and a new vibe-coding tier (Replit Agent, Lovable, Bolt, v0) emerged that the matrix doesn't acknowledge. The `harness-matrix.md` needs a full rewrite of its market-shape table, Claude Code, Codex, and GitHub Copilot sections; the addition of Devin, Aider, Cursor-as-platform, CodeRabbit-as-platform, and vibe-coding rows; plus new sub-matrices for code review, agent-native control centers, and async/fire-and-forget agents. The Architect OS thesis that CLI agents are the right primary harness still holds — but the supporting cast and async story must be rebuilt from scratch.
