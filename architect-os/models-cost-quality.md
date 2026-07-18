# Models — Cost, Quality & Routing

*Which models for which tasks. Pricing verified against official provider docs mid-2025. The rule: route by task importance, not cost minimization.*

---

## The model landscape

| Model | Provider | Input ($/1M) | Output ($/1M) | Best for |
|---|---|---|---|---|
| Claude Opus 4.6 | Anthropic | $5.00 | $25.00 | Highest-quality reasoning, specs, architecture |
| Claude Sonnet 4.6 | Anthropic | $3.00 | $15.00 | Daily driver: implementation, planning, review |
| Claude Haiku 4.5 | Anthropic | $1.00 | $5.00 | Mechanical: dumps, simple refactors, boilerplate |
| GPT-5.2 | OpenAI | $1.75 | $14.00 | Strong Sonnet alternative; second-opinion implementation |
| GPT-5.2 Pro | OpenAI | $21.00 | $168.00 | Premium reasoning (rarely needed for coding) |
| o3 | OpenAI | $10.00 | $40.00 | Complex multi-step reasoning, planning |
| o4-mini | OpenAI | $1.10 | $4.40 | Budget reasoning |
| GPT-5 mini | OpenAI | ~$0.15 | ~$0.60 | Cheapest: dumps, label classification |
| Gemini 3.1 Pro | Google | $2.00 | $12.00 | Strong all-rounder |
| Gemini 3 Flash | Google | $0.50 | $3.00 | Budget, batch processing |

**Sources:** [Anthropic pricing](https://docs.anthropic.com/en/docs/about-pricing), [OpenAI pricing](https://platform.openai.com/docs/pricing), [Google AI pricing](https://ai.google.dev/pricing).

---

## Claude Code subscription vs API

| Plan | Price | What |
|---|---|---|
| Claude Pro | $20/mo | Limited Claude Code, rate-limited |
| Claude Max (5×) | $100/mo | Daily coding at moderate volume |
| Claude Max (20×) | $200/mo | Recommended for heavy daily coding |
| Max + API key | $200/mo + API | Max plan + BYO API key |

**Recommendation:** Max (20×, $200/mo) for heavy usage. 15–20 tickets/month covered by the plan.

---

## Cost per typical operation

| Operation | Model | Approx cost |
|---|---|---|
| S2 Spec generation | Opus 4.6 | $0.50–$2.00 |
| S4 Architecture | Opus 4.6 or o3 | $0.75–$3.00 |
| S5 Ticket decomposition | Sonnet 4.6 | $0.30–$1.00 |
| S6 XS ticket (1 file) | Sonnet 4.6 | $0.10–$0.50 |
| S6 M ticket (3–5 files) | Sonnet 4.6 | $0.50–$3.00 |
| S6 Complex S ticket | Sonnet 4.6 | $1.00–$5.00 |
| S7 Self-review | Sonnet 4.6 | $0.05–$0.20 |
| S9 Memory dump | Haiku / GPT-5 mini | $0.01–$0.05 |

**Monthly estimate (15–20 tickets):** Claude Max $200/mo + Codex $10–20/mo = ~$210–220/mo.

---

## Model routing

| Task | Model | Why |
|---|---|---|
| Spec generation (S2) | Opus 4.6 | Highest-leverage work. Bad spec costs 10× the premium. |
| Architecture (S4) | Opus 4.6 or o3 | Deep trade-off analysis. |
| Ticket decomposition (S5) | Sonnet 4.6 | Human gate catches errors. |
| Implementation (S6) | Sonnet 4.6 | Daily driver. 90% of tickets. |
| Self-review (S7) | Sonnet 4.6 | Same model catches own patterns. |
| Second opinion | GPT-5.2 (Codex) | Different model family → different blind spots. |
| AI review (S7) | CodeRabbit (multi-model) | Subscription, not per-token. |
| Memory dumps (S9) | Haiku / GPT-5 mini | Mechanical distillation. |

### Price-quality frontier (coding tasks)

```
Opus 4.6     ████████████  best quality, highest cost
Sonnet 4.6   ██████████░░  90% of Opus quality at 60% cost
GPT-5.2      ██████████░░  comparable to Sonnet
o3           █████████░░░  best structured reasoning
Haiku 4.5    ████████░░░░  good for simple tasks
GPT-5 mini   ██████░░░░░░  mechanical only
```

The practical rule: **route by task importance, not cost.** Specs and architecture get the best model. Implementation gets the good model. Mechanical work gets the cheap model. The cost of fixing a bad spec in implementation is ~1 hour. The model premium for that spec is ~$1.

---

## Prompt caching

Both Anthropic and OpenAI support prompt caching — repeated prefix content billed at ~10% regular rate. AGENTS.md, constitution, and FSD sections are cache candidates. Effective savings: 40–60% of input tokens for implementation sessions. Automatic — no config needed.

---

## Subscription comparison

| Tool | Plan | Monthly |
|---|---|---|
| Claude Code | Max (20×) | $200 |
| CodeRabbit | Pro | ~$12 |
| Copilot | Pro (optional) | $10 |
| Codex CLI | API usage | ~$10–20 |
| GitHub | Free/Pro | $0–4 |
| **Total** | | **~$232–246** |

**Lightweight profile:** Claude Pro $20 + CodeRabbit $0–12 = ~$20–32/mo.

---

## Cost cliff notes — what NOT to do

1. Don't use Opus for everything. Reserve for S2/S4.
2. Don't let fix-loops spiral. C24 is cost control, not just quality.
3. Don't give agents the whole repo. Narrow context saves 60–80% of tokens.
4. Don't run without prompt caching. It's automatic — just keep AGENTS.md stable.
5. Don't use frontier models for mechanical work. Haiku/GPT-5 mini for dumps.
6. Don't skip self-review. $0.10 self-review catches 30% of errors before human review.

**Sources:** Provider pricing pages (Anthropic, OpenAI, Google); Reuters on subscription tiers.
