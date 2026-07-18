# Cost Control — Budgets, Arbitrage & Kill Switches

*AI agents can burn money if unchecked. This is the control system.*

---

## Per-ticket budget

| Ticket size | Model | Budget ceiling | Typical actual |
|---|---|---|---|
| XS (1 file, <50 lines) | Sonnet 4.6 | $1.00 | $0.10–$0.40 |
| S (1–3 files, <150 lines) | Sonnet 4.6 | $3.00 | $0.30–$1.00 |
| M (3–5 files, <400 lines) | Sonnet 4.6 | $5.00 | $1.00–$3.00 |
| S2 Spec | Opus 4.6 | $2.00/session | $0.50–$2.00 |
| S4 Architecture | Opus 4.6 | $3.00/session | $0.75–$3.00 |
| S9 Dump | Haiku/GPT-5 mini | $0.10 | $0.01–$0.05 |

Budget overrun → the agent is in a fix-loop. Kill the session. Re-plan the ticket (S5).

---

## Model arbitrage

### Downgrade when

| Situation | Downgrade to | Savings |
|---|---|---|
| Memory dumps (S9) | Haiku / GPT-5 mini | 90% vs Sonnet |
| Label classification, summaries | Haiku / GPT-5 mini | 90% |
| Second opinion on simple PR | o4-mini instead of o3 | 75% |
| Batch mechanical changes | Gemini 3 Flash | 85% vs Sonnet |

### Upgrade when

| Situation | Upgrade to | Why |
|---|---|---|
| Complex spec (S2) | Opus 4.6 | Bad spec costs 10× the premium |
| Architecture with 3+ trade-offs (S4) | Opus or o3 | Wrong architecture costs weeks |
| Multi-step refactor in unfamiliar code | Opus 4.6 | Sonnet can miss cross-cutting concerns |
| Agent stuck 2+ times on same ticket | Opus 4.6 (fresh session) | Problem harder than it looked |

### Dual-provider strategy

Run same ticket through Claude Code (Sonnet) AND Codex (GPT-5.2). Compare approaches. Double cost but different models catch different blind spots. Reserve for: security-sensitive code, performance-critical paths, agent-flagged uncertainty, weekly spot-check of spec quality.

---

## Kill switches

### Automatic kill conditions
- Exceeds ticket budget → kill, re-plan
- >3 test-fix cycles without green → kill, approach wrong
- Edits files not in plan → kill, C6/C7
- Adds dependency without ADR → revert, kill, C8
- Proposes major refactor not in scope → reject, C7

### Manual kill signals
- Output quality suddenly degrades (context window saturation)
- Agent repeats itself or cycles
- "Helpfully" improving adjacent code (C7 pattern)
- Confusion/hedging in >20% of responses
- 10+ minutes without meaningful progress

### After a kill
Don't restart the same session. Kill → diagnose → re-plan:
1. Read transcript. What went wrong?
2. Bad plan? Update plan, re-launch.
3. Bad spec? File spec delta, update FSD, re-decompose.
4. Model fluke? Re-launch same plan, same model.

---

## The real cost of bad context

Most expensive thing: code that passes review but has subtle bugs discovered later. Happens when agent has too much context (invents "better" solutions breaking assumptions) or too little (doesn't know about shared utils). Narrow-but-correct context beats broad-but-noisy. Every unnecessary page in the agent's window is a liability.

---

## Subscription justification

"$250/month for AI tools?" vs alternatives:
- Junior contractor: $4,000–8,000/month
- Senior engineer: $12,000–20,000/month
- Your time: if the OS saves 10+ hours, $25/hour — cheaper than any human labor

The OS makes you 3–5× more productive. If you'd spend 40h/month coding and it cuts that to 10, the $250 buys 30h of your life back.
