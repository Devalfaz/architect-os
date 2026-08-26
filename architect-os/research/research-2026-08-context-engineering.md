# Research Update — Context Engineering (August 2026)

*A targeted follow-up to the July 2026 passes, covering one discipline:
**context engineering** — what enters the model's window, and why that is a
quality question before it is a cost question. Fetched 2026-08-26.
**Re-verify after 2026-11-26.***

**Confidence tiers** (same scheme as [harness-matrix.md](../harness-matrix.md)):
✅ verified-primary · 📣 vendor-claim · ❔ inferred/secondary

Scope note: this file is *evidence*, not law. The rules it supports live in
[repo-memory.md](../repo-memory.md); the numbers live here.

---

## 1. Context engineering is now a named discipline

**Confidence:** ✅ anthropic.com/engineering/effective-context-engineering-for-ai-agents
(fetched 2026-08-26).

The framing this OS has used informally ("narrow context beats big context") is
the field's own. Context engineering is defined as *curating and maintaining the
optimal set of tokens during LLM inference* — the successor question to prompt
engineering, covering system instructions, tools, external data, and message
history across many turns.

The governing sentence, worth quoting into any agent-facing doc:

> find the smallest set of high-signal tokens that maximize the likelihood of
> your desired outcome.

Techniques named in the primary source, mapped to where this OS already does them:

| Technique | Anthropic's framing | Where the OS does it |
|---|---|---|
| **Right altitude** | prompts specific enough to guide, flexible enough to leave heuristics | [templates/AGENTS.md](../templates/AGENTS.md) pruning test |
| **Just-in-time retrieval** | hold lightweight identifiers (paths, queries); load at runtime | ego-network graph loading, Layer 3 |
| **Structured note-taking** | persistent external memory outside the window | Layer 4 dumps; `docs/` tree |
| **Sub-agent architectures** | focused agents, clean contexts, return 1–2k-token summaries | [multi-agent.md](../multi-agent.md) Pattern 3 |
| **Compaction** | summarize history near the limit | **now contested — see §3** |

Tool design guidance also lands on this OS's side: minimal overlap between tools,
token-efficient outputs, and the heuristic that *if a human can't decide which of
two tools to use, neither can the agent*.

---

## 2. Context rot: the quality claim is now measured

**Confidence:** ✅ arxiv.org/abs/2602.07962 (LOCA-bench),
arxiv.org/html/2605.12366v1 (Classifier Context Rot); ❔ Chroma 18-model figure
via secondary reporting.

Principle 2 of this OS ("more context makes agents worse *and* more expensive —
a quality rule that happens to save money") was previously an assertion. It now
has an evidence base:

- **Universal degradation.** Chroma's evaluation across **18 frontier models**
  found every one degrades in output quality as input tokens grow ❔.
- **The long-horizon miss rate.** On a task requiring identification of a subtly
  dangerous coding-agent action, frontier models (Opus 4.6, GPT 5.4, Gemini 3.1)
  missed it **2× to 30× more often** when it occurred after ~800K tokens of
  benign activity than when presented alone ✅.
- **Primary failure mode.** For coding agents specifically, context rot is
  described as *the* primary failure mode: agents accumulate noise through
  search, exploration, and backtracking, and that noise degrades every
  subsequent output ❔.
- **LOCA-bench** ✅ provides the controlled benchmark for agents under
  dynamically growing context.

### The three degradation mechanisms

Useful vocabulary — these are distinct and have distinct fixes:

| Mechanism | What happens | The OS's existing defense |
|---|---|---|
| **Context poisoning** | a hallucination enters history and is reproduced at every subsequent step | fresh session per ticket (S6); write-time invalidation ([protocol](../memory-freshness-protocol.md)) |
| **Context distraction** | the agent leans on accumulated history instead of synthesizing a new plan | frozen plan at S5; `converge` grades against it, not the transcript |
| **Context confusion** | irrelevant material in the window degrades response quality | C6/C7 scope rules; ego-network loading, not repo dumps |

**Consequence for this OS:** none of the rules change. The citation changes —
C6 and C7 should be justified as *measured quality mechanisms*, not preferences.

---

## 3. Compaction is contested — "shrink, don't rewrite"

**Confidence:** ❔ single production report (louisbouchard.ai/context-engineering-2026,
fetched 2026-08-26); directionally consistent with published caching economics ✅.
**This is the one item in this file that argues against a widely-standard practice —
treat it as a strong hypothesis, not settled law, and re-verify at the next pass.**

The standard harness move when a window fills is **compaction**: summarize the
session and reinitialize from the summary. The counter-argument is economic and
mechanical:

> Summarizing rewrites the cached prefix, so you pay full price to recompute
> everything you just tried to save.

Reported production numbers, one deployment, 11–13-turn sessions ❔:

| Metric | Full history | Compaction preset |
|---|---|---|
| Cost per turn | **$0.11** | $0.24 |
| Memory recall | **92%** | 38% |
| Time to first token | **17 s** | 21 s |

All three favored keeping history. The prescribed alternative is **"shrink the
context, don't rewrite it"**:

1. **Cap tool outputs** at fixed sizes (reported 38% cost reduction with no
   measured memory loss ❔).
2. **Retrieve, don't stuff** — pull documents on demand rather than pre-loading.
3. **Offload to files, keep a pointer** — fully reversible; nothing is lost and
   retrieval brings it back just in time.
4. **Keep sessions short; start fresh when the task changes.**

**Item 4 is this OS's S6 rule arrived at from the opposite direction.** The
fresh-session-per-ticket discipline is not merely a preference — it is the
recommended mitigation, and it sidesteps the compaction question entirely by
never letting a session reach the window limit.

---

## 4. Caching stability is a context rule, not a cost rule

**Confidence:** ✅ provider pricing already recorded in
[cost-control.md](../cost-control.md) and [harness-matrix.md](../harness-matrix.md).

The OS already records the numbers — Anthropic cache reads at 0.1× base input;
DeepSeek cache-hit input **50× cheaper** ($0.0028 vs $0.14/1M) — and files them
under cost. §3 shows why that filing is wrong: **caching economics are the
mechanism that makes "keep the prefix stable" a context-strategy rule.** A stable
prefix is what makes keeping history affordable, which is what makes not-compacting
viable, which is what preserves recall.

Operationally unchanged, but now justified twice over:
- Keep `AGENTS.md` and FSD sections **stable**; batch edits (weekly distill),
  don't trickle them.
- Never inject timestamps or volatile data into the prompt prefix.
- Track `cache_read_input_tokens ÷ total input tokens` weekly. A falling ratio
  means prefix churn — fix the prefix, don't buy a cheaper model.

---

## 5. Harness engineering has a literature now

**Confidence:** ✅ anthropic.com/engineering/effective-harnesses-for-long-running-agents
and /harness-design-long-running-apps (fetched 2026-08-26).

Architect OS *is* a harness — "the engineering layer that wraps a model so it can
complete work no single context window can hold." The closest external analogue
converges on the same primitives from independent direction:

| Their pattern | This OS's equivalent |
|---|---|
| `claude-progress.txt` — what was accomplished | `memory/dumps/` (Layer 4) |
| `feature_list.json` — requirements marked pass/fail | frozen ACs + the `converge` conformance table |
| git history as the recovery mechanism | one ticket = one branch = one squash commit (C10) |
| a distinct initializer prompt for the first context window | repo bootstrap (S4) vs. per-ticket sessions (S6) |
| browser automation revealing bugs unit tests miss | `converge` tests-as-evidence; Greptile TREX as the paid upgrade |

Their two named failure modes are ones this OS already gates against:

1. **Agents attempting the whole implementation at once**, exhausting context
   mid-feature → the OS's ticket sizing (≤1 day, PR ≤400 lines, C9).
2. **Agents prematurely declaring the project complete** → the `converge` gate,
   which grades the diff against frozen criteria rather than trusting the
   agent's own account.

One guidance item the OS does *not* yet encode: **remove harness scaffolding as
model capability improves.** The quarterly subtraction ritual in
[rituals-and-metrics.md](../rituals-and-metrics.md) is the right home for it — the
question to ask each quarter is "which of our scaffolds is now redundant?"

---

## What changes in the OS

Small. Four edits, no architectural change:

| Change | File | Status |
|---|---|---|
| Add the context-strategy section the four layers serve | [repo-memory.md](../repo-memory.md) | ✅ done |
| Add context-discipline lines to the agent entrypoint | [templates/AGENTS.md](../templates/AGENTS.md) | ✅ done |
| Cross-reference cache stability as a context rule | [cost-control.md](../cost-control.md) | ✅ done |
| Register this file in the research table | [README.md](../README.md) | ✅ done |

**What deliberately does not change:** the ten stages, the constitution, the
review pipeline, the four memory layers, WIP≤2, PR≤400, fresh session per ticket.
The evidence arrived on the side of the existing design — which is the point of
recording confidence tiers in the first place.

---

*Sources fetched 2026-08-26: anthropic.com/engineering/effective-context-engineering-for-ai-agents ·
anthropic.com/engineering/effective-harnesses-for-long-running-agents ·
anthropic.com/engineering/harness-design-long-running-apps ·
platform.claude.com/cookbook (context engineering: memory, compaction, tool clearing) ·
arxiv.org/abs/2602.07962 (LOCA-bench) · arxiv.org/html/2605.12366v1 (Classifier Context Rot) ·
arxiv.org/pdf/2512.13564 (Memory in the Age of AI Agents) ·
arxiv.org/pdf/2606.04967 (From Prompt to Process) ·
louisbouchard.ai/context-engineering-2026 · morphllm.com/context-rot ·
sourcegraph.com/blog/context-engineering. Re-verify after 2026-11-26.*
