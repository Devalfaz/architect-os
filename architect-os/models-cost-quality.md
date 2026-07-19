# Models — Cost, Quality & Routing

*Which models for which tasks. Pricing verified against official provider documentation as of July 2026. Route by task importance, not cost minimization.*

**Sources:** [Anthropic Pricing](https://www.anthropic.com/pricing) (verified July 2026), [OpenAI API Pricing](https://developers.openai.com/api/docs/pricing) (verified July 2026), [PE Collective cross-check](https://pecollective.com/tools/anthropic-api-pricing/), [PE Collective OpenAI cross-check](https://pecollective.com/tools/openai-api-pricing/).

---

## Current model landscape (July 2026)

### Anthropic Claude (current generation)

| Model | Input ($/1M) | Output ($/1M) | Cached Input | Best for |
|---|---|---|---|---|
| **Fable 5** | $10.00 | $50.00 | $1.00 | Long-running agents, highest intelligence |
| **Opus 4.8** | $5.00 | $25.00 | $0.50 | Complex agentic coding, enterprise work |
| **Sonnet 5** | $2.00¹ | $10.00¹ | $0.20 | Coding and agents — introductory pricing |
| **Haiku 4.5** | $1.00 | $5.00 | $0.10 | Fastest, most cost-efficient |
| Sonnet 4.6 (legacy) | $3.00 | $15.00 | $0.30 | Still available, migrate to Sonnet 5 |
| Opus 4.7/4.6 (legacy) | $5.00 | $25.00 | $0.50 | Still available, migrate to Opus 4.8 |

¹ Sonnet 5 introductory pricing $2/$10 through August 31, 2026; $3/$15 standard thereafter. **⏰ Verify-by 2026-08-31:** re-run the budget tables in [cost-control.md](cost-control.md) at standard pricing — the monthly estimate rises ~30% on S6 work if usage is API-billed (Max-plan users unaffected).

### OpenAI (current generation)

| Model | Input ($/1M) | Output ($/1M) | Cached Input | Best for |
|---|---|---|---|---|
| **GPT-5.6-sol** | $5.00 | $30.00 | $0.50 | Flagship — frontier problem-solving |
| **GPT-5.6-terra** | $2.50 | $15.00 | $0.25 | Strong all-rounder |
| **GPT-5.6-luna** | $1.00 | $6.00 | $0.10 | Budget general-purpose |
| **GPT-5.4** | $2.50 | $15.00 | $0.50 | Production workhorse (replaced GPT-4o) |
| **GPT-5.4-mini** | $0.75 | $4.50 | $0.075 | Mid-tier production at 3–6× lower cost |
| **GPT-5.4-nano** | $0.20 | $1.25 | $0.02 | Classification, extraction, routing |
| **gpt-5.3-codex** (Codex) | $1.75 | $14.00 | $0.175 | Codex-specific model for coding tasks |
| **o4-mini** | $1.10 | $4.40 | $0.275 | Best-value reasoning model |
| **o3** | $2.00 | $8.00 | $0.50 | Advanced reasoning (post-launch price cut) |

**Key changes from mid-2025 (stale) to July 2026 (verified):**
- Anthropic added Fable 5 ($10/$50) for long-running agents and Opus 4.8 ($5/$25). Sonnet 5 at $2/$10 intro pricing. Previous "Opus 4.6" and "Sonnet 4.6" are legacy.
- OpenAI added GPT-5.6 family (sol/terra/luna). GPT-4.1 replaced GPT-4o. GPT-4.1 Nano at $0.10/$0.40. Codex uses gpt-5.3-codex at $1.75/$14. o3 cut from $10/$40 to $2/$8.
- Prompt caching: Anthropic 90% off cache reads, OpenAI 50–90% off depending on model family.
- Batch API: both providers offer 50% off all token costs for async processing.

---

## Claude Code subscription vs API

| Plan | Price | What you get |
|---|---|---|
| Claude Pro | $17–20/mo | Limited Claude Code usage, rate-limited. Annual $17/mo, monthly $20. |
| Claude Max (5×) | $100/mo | 5× the usage of Pro. Daily coding at moderate volume. |
| Claude Max (20×) | $200/mo | 20× the usage of Pro. Recommended for heavy daily usage. |
| Max + API credits | $200/mo + API usage | Switch to pay-as-you-go at standard API rates when limit reached. |
| Team (Standard) | $20/seat/mo (annual) | More than Pro usage per seat. |
| Team (Premium) | $100/seat/mo (annual) | 5× more than Standard. Includes Claude Code + Claude Cowork. |
| Enterprise | $20/seat + API rates | Per-seat pricing + usage billed at API rates. |

**Source:** [anthropic.com/pricing](https://www.anthropic.com/pricing), verified July 2026.

---

## Cost per typical operation (updated July 2026)

| Operation | Model | Approx cost |
|---|---|---|
| S2 Spec generation | Opus 4.8 or GPT-5.6-sol | $0.50–$2.00 |
| S4 Architecture | Opus 4.8 or o3 | $0.75–$3.00 |
| S5 Ticket decomposition | Sonnet 5 or GPT-5.6-terra | $0.30–$1.00 |
| S6 XS ticket (1 file) | Sonnet 5 | $0.10–$0.40 |
| S6 M ticket (3–5 files) | Sonnet 5 | $0.50–$2.50 |
| S6 Complex ticket | Opus 4.8 | $1.00–$5.00 |
| S7 Self-review | Sonnet 5 | $0.05–$0.15 |
| S9 Memory dump | Haiku 4.5 or GPT-5.4-nano | $0.01–$0.05 |

**Monthly estimate (15–20 tickets):** Claude Max $200/mo covers all usage. Codex second-opinion: ~$10–20/mo. Total: ~$210–220/mo.

---

## Model routing (updated July 2026)

| Task | Model | Why |
|---|---|---|
| **Spec generation (S2)** | Opus 4.8 or GPT-5.6-sol | Highest-leverage work. Bad spec costs 10× the premium. |
| **Architecture (S4)** | Opus 4.8 or o3 | Deep trade-off analysis. |
| **Ticket decomposition (S5)** | Sonnet 5 | Human gate catches errors. Cost-effective for planning. |
| **Implementation (S6)** | Sonnet 5 | Daily driver. 90% of tickets. Intro price $2/$10 through Aug 2026. |
| **Self-review (S7)** | Sonnet 5 | Same model reviews own work. |
| **Second opinion** | gpt-5.3-codex or GPT-5.6-terra | **C36 (hard rule):** cross-family from the author, or it counts as no review. |
| **AI review (S7)** | CodeRabbit (multi-model) | Subscription, not per-token. |
| **Memory dumps (S9)** | Haiku 4.5 or GPT-5.4-nano | Cheapest models that can do mechanical distillation. |

### Price-quality frontier (coding tasks, July 2026)

```
Fable 5       ██████████████  frontier, expensive, for long-running agents
Opus 4.8      ████████████░░  best quality, reasonable cost
Sonnet 5      ██████████░░░░  strong value — intro pricing through Aug 2026
GPT-5.6-terra ██████████░░░░  comparable to Sonnet 5
GPT-5.3-codex ███████████░░  Codex-specific, good for coding
Haiku 4.5     ████████░░░░░░  fast, cheap, good for simple tasks
GPT-5.4-nano  ██████░░░░░░░░  cheapest, mechanical only
```

---

## Prompt caching (both providers, July 2026)

**Anthropic:** Cache writes 1.25× (5-min TTL) or 2.0× (1-hour TTL) base input. Cache reads = 0.1× base input (90% savings). Stacks with Batch API. Two operational bonuses: cache reads **do not count against rate limits**, and the cache-diagnostics beta reports exactly where consecutive prompts diverged (use it before hand-debugging cache misses).

**OpenAI:** Discount varies by model family. GPT-5.6 family: 90% off cached reads. GPT-5.4 family: 75% off. o-series: 50% off. **⚠️ The `prompt_cache_key` ceiling:** on GPT-5.6+ you must set `prompt_cache_key`, and traffic per key must stay under **~15 requests/min** — above that, routing degrades and cache hit rates silently collapse. Partition busy workloads across multiple keys. Cached tokens still count against OpenAI rate limits (unlike Anthropic).

**Batch APIs (both providers):** 50% off everything, async (typically <1h Anthropic, ≤24h OpenAI), and it **stacks with caching**. Use the 1-hour cache TTL inside batches (5-min expires mid-batch). Natural fits: S9 memory dumps and weekly distills, bulk second-opinion review scans, eval runs, large mechanical migrations.

**Architect OS caching:** AGENTS.md, constitution, FSD sections = cache candidates. Effective savings: 40–60% on implementation sessions. Automatic — keep AGENTS.md stable (every edit invalidates the prefix behind it).

---

## Subscription comparison (July 2026)

| Tool | Plan | Monthly |
|---|---|---|
| Claude Code | Max (20×) | $200 |
| CodeRabbit | Pro | ~$12–15 |
| Copilot | Pro (optional) | $10 |
| Codex CLI | API usage | ~$10–20 |
| GitHub | Free/Pro | $0–4 |
| **Total** | | **~$232–249** |

**Lightweight profile:** Claude Pro $17–20 + CodeRabbit $0–15 = ~$20–35/mo.

**Enterprise notes:** Copilot Enterprise $39/user/mo (adds Workspace + Code Review). Anthropic Enterprise $20/seat + API rates. Team Premium at $100/seat includes Claude Code + Claude Cowork.

---

## Key cost changes (mid-2025 → July 2026)

| What changed | Impact |
|---|---|
| Sonnet 5 at $2/$10 intro (was $3/$15 for Sonnet 4.6) | Primary S6 model got cheaper |
| Anthropic 90% prompt caching (was 90% already — unchanged) | Still the biggest cost lever |
| GPT-5.4-nano at $0.20/$1.25 (was GPT-4.1 Nano $0.10/$0.40) | Budget option slightly more expensive |
| Opus 4.8 at $5/$25 (same price as Opus 4.6, more capable) | Flagship improved at same cost |
| o3 price cut: $10/$40 → $2/$8 | Advanced reasoning dramatically cheaper |
| Fable 5 added at $10/$50 | New tier for long-running agents |

---

*Sources: Anthropic pricing page ([anthropic.com/pricing](https://www.anthropic.com/pricing)), OpenAI API pricing ([developers.openai.com/api/docs/pricing](https://developers.openai.com/api/docs/pricing)), PE Collective cross-verification ([pecollective.com/tools/anthropic-api-pricing/](https://pecollective.com/tools/anthropic-api-pricing/), [pecollective.com/tools/openai-api-pricing/](https://pecollective.com/tools/openai-api-pricing/)). All numbers verified July 2026.*
