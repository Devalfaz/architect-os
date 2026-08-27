# Cost Control — Budgets, Arbitrage & Kill Switches

*AI agents can burn money if unchecked. This is the control system. Pricing verified against official provider docs July 2026.*

**Scope:** budgets below price the [default stack](adoption-plan.md) (Claude Code + Anthropic models). The open-frontier stack runs the same budget structure at ~26–30× lower per-op cost (recomputed 2026-08-27 at Sonnet 5 standard pricing) — routing and per-op figures live in [harness-matrix.md](harness-matrix.md).

---

## Per-ticket budget

*API-billed figures. **Max-plan users can ignore the dollar amounts** — the flat
$200 doesn't meter tokens — but the ceilings still work as a **fix-loop
tripwire**, which is their real job. Sonnet rows re-run at standard $3/$15 on
2026-08-27 (1.5× the retired intro rate).*

| Ticket size | Model | Budget ceiling | Typical actual |
|---|---|---|---|
| XS (1 file, <50 lines) | Sonnet 5 | $1.50 | $0.15–$0.60 |
| S (1–3 files, <150 lines) | Sonnet 5 | $4.50 | $0.45–$1.50 |
| M (3–5 files, <400 lines) | Sonnet 5 | $7.50 | $0.75–$3.75 |
| S2 Spec | Opus 4.8 or GPT-5.6-sol | $3.00/session | $0.50–$2.00 |
| S4 Architecture | Opus 4.8 or o3 | $3.00/session | $0.75–$3.00 |
| S9 Dump | Haiku 4.5 or GPT-5.4-nano | $0.10 | $0.01–$0.05 |

Budget overrun → the agent is in a fix-loop. Kill the session. Re-plan the ticket (S5).
C24 (fix loops ≤2 rounds) is both a quality control AND a cost control.

---

## Per-orchestrator budget (multi-agent runs)

Multi-agent systems burn **~15× the tokens of chat** — a fanned-out run turns a $3 task into a $50 run when misapplied ([multi-agent.md](multi-agent.md)). Orchestrator runs get their own ceilings, separate from per-ticket:

| Run type                                  | Budget ceiling | Kill trigger                        |
| ----------------------------------------- | -------------- | ----------------------------------- |
| Orchestrator + 3–5 subagents (research)   | $10.00         | Token spend, not fix-loop count     |
| Batch migration (`claude -p` loop, N files) | $0.50 × N files | First 2–3 files wrong → stop, re-prompt |
| Agent team (multi-day coordinated)        | $25.00/day     | Daily spend review                  |

If an orchestrator run hits the ceiling, the decomposition was wrong or the task didn't warrant fan-out. Single-agent is the default; fan-out is a budgeted exception.

---

## Model arbitrage (updated July 2026)

### Downgrade when

| Situation | Downgrade to | Savings |
|---|---|---|
| Memory dumps (S9) | Haiku 4.5 or GPT-5.4-nano | ~95% vs Opus |
| Label classification, summaries | Haiku 4.5 or GPT-5.4-nano | ~95% |
| Second opinion on simple PR | GPT-5.4-mini instead of o3 | ~85% |
| Batch mechanical changes | GPT-5.4-nano (batch) | ~95% vs Sonnet |
| Simple CRUD boilerplate | GPT-5.4-mini | ~65% vs Sonnet 5 |

### Upgrade when

| Situation | Upgrade to | Why |
|---|---|---|
| Complex spec (S2) | Opus 4.8 or GPT-5.6-sol | Bad spec costs 10× the premium |
| Architecture with 3+ trade-offs (S4) | Opus 4.8 or o3 | Wrong architecture costs weeks |
| Multi-step refactor in unfamiliar code | Opus 4.8 | Sonnet 5 can miss cross-cutting concerns |
| Complex multi-file agent task | Fable 5 ($10/$50) | Purpose-built for long-running agentic work |
| Agent stuck 2+ times on same ticket | Opus 4.8 (fresh session) | Problem harder than it looked |

### Dual-provider strategy

Run same ticket through Claude Code (Sonnet 5) AND Codex (gpt-5.3-codex). Compare approaches. Different model families catch different blind spots — the same principle C36 makes mandatory for review. Reserve for: security-sensitive code, performance-critical paths, agent-flagged uncertainty, weekly spot-check.

---

## Kill switches

### Automatic kill conditions
- Exceeds ticket budget → kill, re-plan
- >3 test-fix cycles without green → kill, approach wrong
- Edits files not in plan → kill, C6/C7 violation
- Adds dependency without ADR → revert, kill, C8 violation
- Proposes major refactor not in scope → reject, C7

### Manual kill signals
- Output quality suddenly degrades (context window saturation)
- Agent repeats itself or cycles without converging
- "Helpfully" improving adjacent code (C7 pattern)
- Confusion/hedging in >20% of responses
- 10+ minutes without meaningful progress

### After a kill
Don't restart the same session. Kill → diagnose → re-plan:
1. Read transcript. What went wrong?
2. Bad plan? Update plan, re-launch.
3. Bad spec? File spec delta, update FSD, re-decompose.
4. Model fluke? Re-launch same plan, same model. Sometimes it's just a bad sample.

---

## Prompt caching — the free money

**Anthropic:** Cache writes 1.25× (5-min TTL) or 2.0× (1-hour TTL). Cache reads = 0.1× base input (90% savings). Stacks with Batch API. Cache reads don't count against rate limits; the cache-diagnostics beta pinpoints where a prefix diverged.

**OpenAI:** 50–90% off cached reads depending on model family. GPT-5.6 family: 90%. GPT-5.4 family: 75%. o-series: 50%. On GPT-5.6+, set `prompt_cache_key` and keep each key under **~15 requests/min** — exceeding it silently collapses hit rates; partition across keys.

**Batch APIs — the second 50%:** both providers give 50% off async batches, stacking with caching (use the 1-hour TTL inside batches). Route through batch: S9 dumps, weekly distills, bulk review scans, eval runs, mechanical migrations. Anything that doesn't need an answer this minute shouldn't pay interactive prices.

**Architect OS caching:** AGENTS.md, constitution, FSD sections = auto-cached between sessions. Effective savings: 40–60% on implementation sessions. Keep AGENTS.md stable — cache invalidates on every edit.

**Health metric:** track `cache_read_input_tokens` ÷ total input tokens in the weekly cost review. A falling cache-hit ratio usually means AGENTS.md churn or unstable session prefixes — fix the prefix, don't buy a cheaper model.

> **This section is not only about money.** Prefix stability is what makes
> keeping a session's history affordable, which is what makes *not* compacting
> viable, which is what preserves the agent's recall. Compaction rewrites the
> cached prefix — so you pay full price to recompute exactly what caching saved,
> and lose recall doing it. Treat the rules above as **context strategy** with a
> budget side-effect: [repo-memory.md § Layer 0](repo-memory.md),
> evidence in [research-2026-08-context-engineering.md](research/research-2026-08-context-engineering.md).

---

## Subscription comparison (July 2026)

| Tool | Plan | Monthly |
|---|---|---|
| Claude Code | Max (20×) | $200 |
| CodeRabbit | **Free tier**, or Pro if you outgrow it | **$0**, or $24 (annual) / $30 (monthly) |
| Copilot | Pro (optional) | $10 |
| Codex CLI | API usage (gpt-5.3-codex) | ~$10–20 |
| GitHub | Free/Pro | $0–4 |
| **Total** | | **~$220–264** |

**⚠ Start CodeRabbit on Free, not Pro.** ✅ Verified 2026-08-27: Pro is **free
forever on public repos**, and the free tier on private repos allows 200
files/hour and **4 PR reviews/hour** — comfortably above a WIP≤2 solo pipeline.
The canonical [.coderabbit.yaml](github/.coderabbit.yaml) works on Free. Upgrade
only when you actually hit the rate limit; billing counts **only developers who
open PRs**, so solo is one seat. (Prior editions of this doc listed ~$12–15 ❔ —
that was wrong by roughly 2×.)

**Open-frontier stack:** OpenCode $0 + DeepSeek API $15–40 + OpenRouter reviews $5–10 + CodeRabbit $0–30 = **~$20–80/mo** (routing detail: [harness-matrix.md](harness-matrix.md)).

**Lightweight profile:** Claude Pro $17–20 + CodeRabbit $0–30 = ~$17–50/mo.

**Cost justification:** If the OS saves you 10+ hours/month of coding time, $250 is $25/hour — cheaper than any human labor. The real metric: what's your time worth?

---

## Key cost differences (mid-2025 stale → July 2026 verified)

| What changed | Impact |
|---|---|
| Sonnet 5 at $3/$15 standard (intro $2/$10 expired 2026-08-31) | ✅ **Actioned 2026-08-27** — per-ticket budgets re-run at standard pricing, Sonnet rows ×1.5. Same rate as legacy Sonnet 4.6, so the intro was a discount window, not a price cut. Max-plan users unaffected |
| Opus 4.8 same price as Opus 4.6 ($5/$25), more capable | Flagship improved at zero cost increase |
| GPT-5.3-codex ($1.75/$14) for coding tasks | Competitive second-opinion pricing |
| GPT-5.4-nano ($0.20/$1.25) for mechanical work | Budget option for dumps and classification |
| Fable 5 ($10/$50) for long-running agents | New tier — only use when Sonnet/Opus isn't enough |
| o3 price cut: $10/$40 → $2/$8 | Advanced reasoning 5× cheaper for architecture decisions |

---

*Sources: Anthropic pricing ([anthropic.com/pricing](https://www.anthropic.com/pricing)), OpenAI API pricing ([developers.openai.com/api/docs/pricing](https://developers.openai.com/api/docs/pricing)), PE Collective cross-verification. All numbers verified July 2026.*
