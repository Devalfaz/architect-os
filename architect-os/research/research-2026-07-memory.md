---
status: snapshot
last_verified: 2026-07-19
expires: 2026-10-19
---
<!-- SNAPSHOT: findings below were verified against live sources on last_verified.
     After the expiry date, treat every claim here as a hypothesis to re-verify,
     not a fact — this file must not become the next stale baseline. -->

# Research Report — Memory Architectures & Context Engineering for AI Coding Agents (July 2026)

*Web research synthesis. 7 high-signal sources fetched 2026-07-19. Designed to feed updates into `repo-memory.md` and `memory-freshness-protocol.md`.*

---

## Executive summary

Between mid-2025 and July 2026, **context engineering** crystallized as a named discipline distinct from prompt engineering. Anthropic's September 2025 post *Effective context engineering for AI agents* gave the field its canonical framing: context is a finite resource with diminishing marginal returns; the goal is "the smallest possible set of high-signal tokens that maximize the likelihood of the desired outcome." Three concrete techniques now dominate long-horizon agent work — **compaction**, **structured note-taking**, and **sub-agent architectures** — and Claude Code itself is the reference implementation of all three.

On the memory side, the field split into three layers that the architect-os four-layer model maps onto imperfectly:

1. **Project context files** (CLAUDE.md / `.cursor/rules/` / AGENTS.md / OpenCode `agents/skills/`) — loaded every session, must be ruthlessly minimal. Anthropic now explicitly warns: *"Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"* The 150-line cap in architect-os's Layer 1 is correct but slightly generous against the new guidance.
2. **Agent memory systems** (Mem0, Letta/MemGPT, Zep, LangGraph memory primitives) — persistent cross-session memory with three sub-patterns: vector-store memory (Mem0, Chroma, Pinecone), block/actor memory (Letta), and **temporal knowledge graphs** (Zep). Zep's the breakout — S&P Market Intelligence flagged it as the de-facto enterprise memory layer in April 2026, and its key differentiator is **automatic invalidation of stale facts** when new evidence contradicts old ones. architect-os's Layer 3 graph has no equivalent.
3. **Repo knowledge graphs** (Aider repo map, tree-sitter + graph ranking, Sourcegraph Cody, Cursor dynamic context discovery, Greptile) — built from AST analysis, ranked by dependency/import graphs, and dynamically sized to a token budget. Aider's approach is the canonical reference: tree-sitter parses the repo, a PageRank-style algorithm ranks symbols, the top-N most-referenced identifiers fit the budget. architect-os's Layer 3 graph has the right node/edge vocabulary but no automated construction or token-budget-aware ranking.

The most important shift for architect-os: **freshness is now a first-class mechanism, not a protocol on top.** Cursor's self-driving codebases research (Feb 2026) found that `scratchpad.md` should be *rewritten* frequently rather than appended to, agents should auto-summarize at context limits, and stale memory must invalidate automatically — not "be flagged for review." Zep implements exactly this as a substrate feature. architect-os's "stale = hypothesis, not fact" framing is correct in spirit but operationally too manual.

The other major shift: **sub-agent architectures are now the default answer to long-horizon context limits**, not larger context windows. Anthropic's multi-agent research system, Claude Code's subagent dispatch, and Cursor's planner/subplanner/worker hierarchy all converge on the same pattern — deep work happens in isolated context windows that return 1–2k token summaries. architect-os has no explicit subagent-memory contract; the Layer 4 daily dumps are closer to a human-journal pattern than an agent-handoff protocol.

---

## Part 1 — Context engineering as a discipline

### Source: Anthropic, *Effective context engineering for AI agents* (Sep 29, 2025)
URL: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

**Key claims:**

- **Context engineering** = the set of strategies for curating and maintaining the optimal set of tokens during LLM inference. It is the natural progression of prompt engineering, which was about *writing* instructions. Context engineering is about *curating* the whole state — system instructions, tools, MCP, external data, message history — that lands in the model's attention window.
- **Context rot** (citing Chroma's needle-in-a-haystack research): as token count rises, recall precision falls. Context must be treated as a finite resource with diminishing marginal returns. The transformer's n² attention scaling plus training-data distribution bias toward shorter sequences means even 1M-token windows degrade.
- **The guiding principle:** *"find the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome."* Minimal ≠ short; minimal = sufficient and no more.
- **System prompt altitude:** avoid two failure modes — hardcoding brittle if-else logic (fragile) vs. vague high-level guidance (no signal). The Goldilocks zone is "specific enough to guide, flexible enough to leave heuristics to the model."
- **Tools** define the contract between agents and their information/action space. Most common failure: bloated tool sets with overlapping functionality. "If a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better."
- **Just-in-time retrieval** is replacing pre-inference RAG for agentic work. Claude Code keeps lightweight identifiers (file paths, stored queries) and uses `glob`/`grep`/Bash to load data at runtime. "Folder hierarchies, naming conventions, and timestamps all provide important signals." Hybrid strategies (some up-front + JIT exploration) are the empirical winner.
- **Compaction** — summarize a near-limit conversation, reinit a fresh window with the summary + the five most recently accessed files. The art is selection: maximize recall first, then iterate for precision. Tool-result clearing is the safest lightest-touch form. Anthropic shipped a Context Management memory tool in beta on the Claude Developer Platform — file-based system for agents to store and consult information outside the context window.
- **Structured note-taking / agentic memory** — agent writes notes to external memory (NOTES.md, to-do list) and pulls them back later. Claude playing Pokémon maintains tallies across thousands of steps, maps of explored regions, combat strategies — all unprompted. After context resets, it reads its own notes and continues.
- **Sub-agent architectures** — specialized subagents handle focused tasks in clean context windows. Main agent coordinates; subagents explore (10k+ tokens) and return 1–2k token distilled summaries. Separation of concerns: detailed search context stays isolated, lead agent synthesizes.

**Karpathy framing (cited in Anthropic post):** context engineering is "the art and science" of curating what goes into the limited context window from a constantly evolving universe of possible information. (Source: x.com/karpathy/status/1937902205765607626, linked from the Anthropic post.)

### Source: Anthropic, *Best practices for Claude Code* (fetched 2026-07-19)
URL: https://www.anthropic.com/engineering/claude-code-best-practices

**Key claims relevant to memory:**

- *"Claude's context window is the most important resource to manage."* Performance degrades as it fills. Track usage continuously with a custom status line.
- **CLAUDE.md guidance:** "Keep it concise. For each line, ask: *'Would removing this cause Claude to make mistakes?'* If not, cut it. Bloated CLAUDE.md files cause Claude to ignore your actual instructions!" Treat CLAUDE.md like code: review when things go wrong, prune regularly, test changes by observing behavior.
- **Include / Exclude table** (verbatim):
  - ✅ Bash commands Claude can't guess; code style rules that differ from defaults; testing instructions; repository etiquette; architectural decisions; developer env quirks; common gotchas.
  - ❌ Anything Claude can figure out by reading code; standard language conventions; detailed API docs (link to docs instead); information that changes frequently; long explanations; file-by-file descriptions; self-evident practices.
- **Emphasis tuning:** add "IMPORTANT" or "YOU MUST" to improve adherence. "If Claude keeps doing something you don't want despite having a rule against it, the file is probably too long and the rule is getting lost."
- **Skills** are the answer to "domain knowledge that only applies sometimes" — load on demand instead of bloating every session.
- **Subagents** for investigation: "Since context is your fundamental constraint, subagents are one of the most powerful tools available." They explore in separate context windows and report summaries — keeps main conversation clean for implementation.
- **Compaction customization:** CLAUDE.md can carry instructions like *"When compacting, always preserve the full list of modified files and any test commands"* to ensure critical context survives summarization.
- **Adversarial review step** — before treating a task as done, have a subagent review the diff in fresh context. The reviewer sees only the diff and the criteria, not the reasoning that produced the change.

**architect-os gap:** The AGENTS.md content list (Layer 1) — "Domain language (5–10 terms), File conventions (10 most important paths), Constitution summary, Architecture style and key decisions, Gotchas, Active constraints" — slightly over-encroaches on territory Anthropic now says to *exclude* (file-by-file descriptions, info that changes frequently). The 150-line cap is good but the *content mix* needs pruning toward "things Claude can't figure out by reading code."

---

## Part 2 — Agent memory architectures

### Source: Mem0 (fetched 2026-07-19)
URL: https://docs.mem0.ai/

**What it is:** A universal, self-improving memory layer for LLM applications. Two delivery modes: Mem0 Platform (hosted) and Open Source (self-hosted). Now ships explicit integrations for **Claude Code, Cursor, and Codex** as drop-in plugins — "memory that persists across sessions, no code to write."

**Key architectural points:**

- Positioning: "Build AI apps that remember." Persistent memory across sessions, with an "agent signup" flow where an AI agent mints a Mem0 API key, claims ownership later, and writes its first memory from the terminal. This is designed for the case where the *agent* is the customer, not the human.
- Integrates with LangChain, CrewAI, Vercel AI SDK, and 20+ partner frameworks.
- Cookbooks cover companions, support agents, voice agents, research tools.

**Pattern:** Mem0 is the canonical **vector-store + LLM-extracted facts** approach. The agent writes raw interaction; Mem0 uses an LLM to extract salient facts, dedupe against existing memory, and store embeddings. Retrieval is similarity search.

**architect-os relevance:** Layer 3 (`repo-graph.json`) is a *hand-curated* knowledge graph. Mem0 shows the alternative — *LLM-extracted* memory that self-maintains. The trade-off: Mem0-style memory is fuzzy and probabilistic; a hand-curated repo graph is precise but labor-intensive. The hybrid (LLM proposes graph deltas, human reviews weekly) is what architect-os's "weekly distill from dumps" already gestures at, but the distill step lacks an LLM extraction contract.

### Source: Letta / MemGPT (fetched 2026-07-19)
URL: https://docs.letta.com/

**What it is:** "The memory-first agent that remembers and learns." Direct descendant of MemGPT. Now a full Agent SDK with Letta Agent (managed), Letta Agent SDK, and a Handbook.

**Key architectural primitives (from docs structure):**

- **Stateful agents** as first-class objects — agents have identity, persistent state, and survive across sessions.
- **Memory blocks** — named, typed blocks of memory attached to an agent (the "core memory" from MemGPT). Editable by the agent itself.
- **Shared memory** (Repositories) — blocks and memory shared across multiple agents.
- **Archival memory** — long-term storage, vector-searchable, pulled in on demand.
- **MemFS** — a filesystem abstraction for agent-accessible files.
- **Context hierarchy** — a layered model of which memory goes in-context vs. archival vs. cold storage.
- **Conversations, Messages, Compaction, Long-running executions** as first-class concepts — compaction is built-in, not bolted on.
- **AgentFile (.af)** — a serializable description of an agent's full state, portable across deployments.
- **Letta Evals** — a full eval framework: suites, datasets, targets, graders, extractors, gates. Memory quality is now measured, not assumed.
- **Skills, Mods, Subagents, Schedules, Channels** (Slack/Telegram/Discord/WhatsApp/Signal) — the agent is an always-on entity, not a stateless function call.

**Pattern:** Letta is the canonical **block/actor memory** approach. Memory is structured into named blocks the agent reads and writes itself during a run. This is the closest production implementation of the "agentic memory" pattern Anthropic describes (Claude playing Pokémon writing its own notes).

**architect-os relevance:** Layer 4 (daily dumps) and Layer 1 (AGENTS.md) are *passive* memory — written by the human, read by the agent. Letta's pattern is the opposite: the *agent* writes its own memory blocks. architect-os has no "agent-writable" memory layer. The daily dump is human-authored. This is a structural gap: an agent that can't write its own notes can't improve within a long session.

### Source: Zep (fetched 2026-07-19)
URL: https://www.getzep.com/

**What it is:** Enterprise agent memory built on **Context Graphs** — temporal knowledge graphs with automatic invalidation. S&P Global Market Intelligence (April 2026 report) called it "a de facto partner in this layer of the enterprise agent stack."

**Key architectural points:**

- **Context Graph Engine** — builds graphs from any source (chat history, business data, user interactions, agent runs). Entities, Facts, Episodes. As of the homepage live ticker: 1,782 entities, 9,156 facts, 2,449 episodes for one demo agent, last updated 3s ago.
- **Sub-200ms retrieval at any scale** — p95 stays flat from 10K to 100M graphs (~148–168ms). Latency doesn't degrade with graph size.
- **Memory validity / automatic invalidation** — the headline feature. "When new information contradicts what's in the graph, Zep invalidates the old fact. Your agent reasons with the latest decisions, traits, and behaviors. Old facts stay as history. Ask what's true now, or what was true on any past date." Example shown: Robbie says "I only wear Adidas" on Sep 7 2024; on Sep 22 he returns Adidas and says "I'll be buying Nike from now on" — Zep invalidates the old fact and stores the new one, with provenance preserved (every fact traces back to the source episode).
- **Observations** — pattern-level insights surfaced from the graph structure ("Jane has upgraded within two weeks of each of the last three product launches"). Goes beyond facts and summaries.
- **Context Lake** — "millions of context graphs, governed and served as one system." The data-lake pattern applied to agent context.
- **Governance at the substrate** — ABAC access control, retention policies, legal hold, audit trails, API logs. Authorization lives in the substrate, not bolted on. Critical for enterprise.
- **Three deployment models:** Cloud (managed), Cloud + BYOK (your own encryption keys), BYOC (Zep in your VPC).
- **Benchmarks:** 94.7% accuracy on LoCoMo, 90.2% on LongMemEval, both with ~5k token context windows. The point: more accurate, faster, fewer tokens than the alternatives.
- **Three lines of code** to add memory to any agent: `client.thread.add_messages(...)`, `client.graph.add(...)`, `client.thread.get_user_context(...)`.

**Pattern:** Zep is the canonical **temporal knowledge graph** approach. The temporal part is the differentiator — facts have valid-time intervals, contradictions trigger invalidation, history is preserved. This is exactly what graph-based RAG with no time semantics gets wrong.

**architect-os relevance:** This is the biggest single gap in architect-os. Layer 3's graph schema has `last_modified` and `last_verified` but no **valid-time intervals**, no **automatic contradiction detection**, no **provenance back to the source episode**. The freshness protocol says "stale = hypothesis, not fact" but operationally relies on the weekly distill to catch staleness. Zep's model is: the graph invalidates itself at write time, not at review time. architect-os should add `valid_from` / `valid_to` / `invalidated_by` fields and an explicit provenance pointer to the source dump or ADR.

### LangChain / LangGraph memory primitives

URLs attempted: blog.langchain.com (multiple paths returned 404 on fetch — the blog has reorganized). Based on the prior published material and the Anthropic post's citations:

- LangGraph ships **memory primitives** for persistent state across runs: `MemorySaver`, checkpointers, and store APIs. The pattern is store-backed state keyed by thread/namespace.
- LangChain's `Memory` module (older) provided buffer, summary, entity, knowledge-graph, and vector-store-backed memory. Most of this has been superseded by LangGraph's store + checkpoint pattern.
- The LangGraph approach: short-term memory = thread state (checkpointed); long-term memory = store (cross-thread, namespace-keyed). This maps cleanly onto architect-os's dumps (short-term, thread) and repo-graph.json (long-term, cross-thread).

**architect-os relevance:** LangGraph's thread/store split is the same conceptual split as architect-os's dumps vs. graph. The LangGraph pattern of namespace-keyed store is worth adopting — `memory/repo-graph.json` is a single file, but a namespace-keyed store (`memory/store/<namespace>/<key>`) would let multiple agents / multiple concerns have isolated subgraphs.

### Vector store based memory (general)

Pinecone, Weaviate, Chroma, and pgvector remain the default for "embeddings + similarity search" memory. None of them have re-architected for agents specifically — they are general-purpose vector indexes. The agent-memory specific layer (Mem0, Zep, Letta archival) sits on top.

**Pattern:** chunk text → embed → store with metadata → retrieve top-k by similarity, optionally filtered by metadata. The 2026 reality: pure vector retrieval is the baseline; the differentiator is what you chunk, how you extract facts from chunks, and how you handle contradictions.

**architect-os relevance:** Layer 3 is a JSON graph, not a vector store. There's no embedding-based retrieval path — agents either navigate the graph programmatically (ego-network load) or read AGENTS.md. For a repo with thousands of files, a vector index over file descriptions / ADRs / research notes would complement the graph. Consider adding a `memory/embeddings/` layer.

---

## Part 3 — Repo knowledge graphs

### Source: Aider, *Repository map* (fetched 2026-07-19)
URL: https://aider.chat/docs/repomap.html

**What it is:** Aider sends a *repo map* to the LLM with each change request — a concise listing of files and their most-referenced symbols (classes, methods, functions with signatures).

**Key architectural points:**

- **Tree-sitter** parses every file to extract symbols and definitions.
- **Graph ranking algorithm:** each source file is a node, edges connect files with dependencies. A PageRank-style algorithm ranks symbols by reference frequency.
- **Token budget:** `--map-tokens` defaults to 1k. Aider dynamically expands the map (especially when no files are in chat yet) and contracts it as the chat state grows.
- **Most-relevant selection:** the map doesn't include every symbol — only the ones most often referenced elsewhere. These are "the key pieces of context that the LLM needs to know to understand the overall codebase."
- **Benefits:** the LLM sees signatures from everywhere (often enough to solve tasks), and can use the map to decide which files to actually pull into context.

**Pattern:** Aider is the canonical **tree-sitter + graph-ranking** approach. The repo map is generated, not curated. The token budget is enforced, not aspirational.

**architect-os relevance:** Layer 3's graph is hand-curated from weekly distills. Aider's is auto-generated from AST analysis every run. architect-os's graph has richer semantics (concepts, decisions, ADRs, workflows as node types) but no automated construction from source. The two should be hybridized: an Aider-style auto-generated *structural* graph (files, symbols, dependencies) as the base, with architect-os's *semantic* nodes (concepts, decisions) layered on top as annotations.

### Sourcegraph Cody / CodeQL / tree-sitter ecosystem (background)

Sourcegraph Cody uses a code intelligence graph built from SCIP (Sourcegraph Code Intelligence Protocol) indices — precise symbol navigation, references, and hover. CodeQL (GitHub) builds relational queries over a code database for security analysis. tree-sitter is the de-facto lightweight parser used by Aider, Continue.dev, Zed, Neovim, and most modern editor tooling.

**Pattern:** There are two regimes — **precise code intelligence** (SCIP/LSIF, CodeQL — requires building an index, expensive, accurate) and **fuzzy structural** (tree-sitter — fast, approximate, runs anywhere). Agent tools mostly use tree-sitter because precise indexers are too slow for interactive use.

**architect-os relevance:** architect-os currently has *neither* — Layer 3 is hand-maintained. The cheapest upgrade is tree-sitter-based structural graphing; the highest-quality upgrade is SCIP. Both should be automated, not manual.

### Repo-understanding tools (Cline, Continue, Greptile, Graphite, repo-graph projects)

- **Continue.dev** — open-source AI coding assistant. Blog (fetched 2026-07-19) shows current focus on "AI slop is a process problem," "intervention rates are the new build times," and "chiseling: the art of polishing vibe code" — all pointing to *process* over *memory*. Continue's docs and blog don't emphasize a repo-graph approach; they lean on RAG over the codebase.
- **Cline / Roo Code** — VS Code AI coding agents. Mostly rely on file-reading and workspace exploration, no explicit graph.
- **Greptile** — codebase search and RAG over repos, index-based.
- **Graphite** — PR stacking tool, not a memory tool (despite the name).
- **repo-graph projects (general)** — academic and OSS projects building knowledge graphs from code, typically using tree-sitter + Neo4j. The pattern is well-established but no single project dominates.

**Pattern:** The space is fragmented. Cursor and Aider have shipped the most polished implementations. No tool has shipped a *semantic* code graph (concepts, decisions, ADRs as first-class nodes) — they are all *structural* (files, symbols, dependencies).

**architect-os relevance:** architect-os's bet on a *semantic* graph (Layer 3 with concepts, decisions, ADRs as node types) is genuinely novel — no production tool does this. But the cost is that the graph can't be auto-generated; it needs the weekly distill. The hybrid (structural auto-generated + semantic human-curated) is the unexplored frontier.

---

## Part 4 — Project context files (CLAUDE.md, .cursor/rules/, copilot-instructions.md, OpenCode agents/skills)

### Anthropic guidance on CLAUDE.md (from the best-practices post above)

- **Loaded every session** — only include things that apply broadly. Domain knowledge that's only sometimes relevant → **skills**.
- **Keep it concise.** Test every line: "Would removing this cause Claude to make mistakes?" If not, cut.
- **Format:** no required format; markdown with clear sections. Use `@path/to/import` to import other files.
- **Locations:** `~/.claude/CLAUDE.md` (global), `./CLAUDE.md` (project, git-tracked), `./CLAUDE.local.md` (personal, gitignored), parent dirs (monorepo), child dirs (loaded on demand).
- **Emphasis:** "IMPORTANT" / "YOU MUST" improves adherence. If a rule keeps getting violated, the file is too long and the rule is getting lost.
- **Skills** (`.claude/skills/<name>/SKILL.md`) — loaded on demand. Two types: knowledge skills (auto-invoked when relevant) and workflow skills (invoked via `/name args`). Use `disable-model-invocation: true` for workflows with side effects.
- **Subagents** (`.claude/agents/<name>.md`) — run in their own context with their own allowed tools. "Since context is your fundamental constraint, subagents are one of the most powerful tools available."

### Cursor .cursor/rules/ and GitHub Copilot copilot-instructions.md

- **Cursor** — `.cursor/rules/*.mdc` files, conditionally loaded based on glob patterns or always. Cursor's Jan 2026 research post *Dynamic context discovery* describes an evolution: instead of static rules, the agent discovers context dynamically per task.
- **GitHub Copilot** — `copilot-instructions.md` in repo root, loaded as custom instructions. Simpler than CLAUDE.md / Cursor rules.
- **OpenCode** — `agents/skills/` structure with SKILL.md files, named and described for invocation routing. The system prompt lists 400+ available skills.

**Pattern:** Every major coding agent now has a project-context-file convention. The differences are: scope of loading (every session vs. on-demand vs. pattern-triggered), file format (markdown vs. YAML frontmatter), and the import/composition mechanism. Anthropic's CLAUDE.md + skills split (always-loaded vs. on-demand) is the cleanest model.

**architect-os relevance:** architect-os's AGENTS.md (Layer 1) + symlink to CLAUDE.md is aligned. The 150-line cap is reasonable but slightly generous given Anthropic's "ruthlessly prune" guidance. The content mix needs review against Anthropic's include/exclude table — specifically, "10 most important paths" drifts toward file-by-file descriptions, which Anthropic says to exclude. Better: link to the graph for path navigation.

---

## Part 5 — Memory freshness / staleness

### Source: Zep (above) — automatic invalidation

Zep's model: facts have valid-time intervals; new evidence that contradicts an existing fact invalidates it at write time, with provenance to the contradicting episode. This is the gold standard.

### Source: Cursor, *Towards self-driving codebases* (Feb 5, 2026)
URL: https://cursor.com/blog/self-driving-codebases

**Freshness mechanisms the Cursor team landed on after extensive multi-agent experimentation:**

1. **`scratchpad.md` should be frequently rewritten, not appended to.** Append-only scratchpads accumulate stale state; rewrites force the agent to re-derive current truth.
2. **Individual agents auto-summarize when reaching context limits.** Compaction is built into the agent loop, not a manual `/compact`.
3. **Self-reflection and alignment reminders** in system prompts, on a schedule.
4. **Agents encouraged to pivot and challenge assumptions at any time** — not bound to a static plan.

**Other relevant findings from this post (for context):**

- Multi-agent coordination via shared state file + locks failed catastrophically (agents held locks too long, forgot to release, lock contention slowed 20 agents to throughput of 1–3). The eventual winning design: recursive planners + workers + handoff messages, no shared mutable state.
- Throughput peaked at ~1,000 commits/hour across 10M tool calls over one week, with no human intervention.
- Accepting a small but stable error rate beats requiring 100% correctness pre-commit — the latter serializes the whole system.
- "Constraints are more effective than instructions. 'No TODOs, no partial implementations' works better than 'remember to finish implementations.'"
- "Give concrete numbers and ranges when discussing quantity of scope. Instructions like 'generate many tasks' tend to produce a small amount... 'Generate 20–100 tasks' conveys the intent is larger scope."

### Anthropic guidance on compaction (from best-practices post)

- Compaction is automatic; `/compact <instructions>` for manual control ("Focus on the API changes").
- Compaction behavior customizable in CLAUDE.md: *"When compacting, always preserve the full list of modified files and any test commands."*
- **`/btw`** for side questions that shouldn't enter conversation history — a freshness mechanism for the *current* context, not just long-term memory.

### architect-os freshness protocol assessment

The existing `memory-freshness-protocol.md` says:

- Every artifact carries `last_verified`.
- Verification schedule: AGENTS.md weekly, architecture monthly, ADRs when constrained code changes, research on expiry date, verified-apis weekly, graph weekly, dumps never.
- "Stale memory is flagged, not deleted. Stale = hypothesis, not fact."
- "Weekly distill is the enforcement point."

**What's new since mid-2025:**

1. **Automatic invalidation at write time** (Zep) — architect-os does this at review time. The gap is real.
2. **Rewrite-not-append scratchpads** (Cursor) — architect-os's daily dumps are append-only by design ("One dump per coding day"). This is correct for dumps (they're raw material) but the *agent-readable* summary should be rewritten.
3. **Compaction customization** (Anthropic) — architect-os has no compaction prompt in AGENTS.md; agents have no instruction on what to preserve across compact.
4. **Provenance back to source episode** (Zep) — architect-os graph nodes have `last_verified` but no `provenance` pointer back to the dump, commit, or ADR that established the fact.

---

## Part 6 — Session memory / handoff

### Anthropic /handoff pattern

- The `/handoff` skill pattern (and Anthropic's recommended workflow for long tasks) is: interview/spec in one session, write to `SPEC.md`, **start a fresh session to execute**. "The new session has clean context focused entirely on implementation, and you have a written spec to reference."
- This is a deliberate session boundary — the spec file is the handoff contract.
- The same pattern applies at the other end: when a long session approaches context limits, the structured note-taking pattern (NOTES.md, to-do) is the handoff to *the same agent in a fresh window*.

### claude-mem package

The `claude-mem` package (referenced in the available-skills list as `how-it-works`, `mem-search`, `knowledge-agent`, `timeline-report`, `weekly-digests`) is the OSS implementation of persistent cross-session memory for Claude Code. It captures observations per session, stores them durably, and supports search / timeline reports / weekly digests. This is structurally identical to architect-os's Layer 4 dumps, but operationalized: the package handles capture, storage, search, and digest generation.

### Subagent context propagation

Anthropic's multi-agent research system post and the Claude Code subagent pattern both use: subagent explores in isolated context, returns 1–2k token distilled summary. The handoff is the summary; no other state crosses the boundary.

**architect-os relevance:** architect-os has no explicit subagent-handoff contract. Layer 4 dumps are a human-journal pattern. The pattern to adopt:

- **Session handoff file** — a `SESSION_HANDOFF.md` (or similar) that the agent rewrites at end of session with: current state, what's done, what's next, blockers, files in flight. The next session reads this first.
- **Subagent result contract** — every subagent dispatch returns a fixed schema: `{summary, files_modified, decisions_made, follow_ups}`. The dispatcher writes these to a `memory/handoffs/` directory.

---

## Part 7 — Long-running agent memory

### Cursor self-driving codebases (above)

The recursive planner/subplanner/worker design is the most detailed publicly-available long-running agent architecture. The memory pattern:

- Workers work on their own repo copy; on completion, write a single handoff containing "what was done, important notes, concerns, deviations, findings, thoughts, feedback."
- The planner receives this as a follow-up message; "even if a planner is 'done,' it continues to receive updates, pulls the latest repo, and can continue to plan."
- No shared mutable state across agents. No locks. Information propagates up the chain via handoffs.
- Each agent auto-summarizes at context limits; scratchpads are rewritten.

### Claude Code session continuity

- `claude --continue` resumes the most recent session; `claude --resume` picks from a list.
- Sessions are named (`/rename`) and treated "like branches: each workstream gets its own persistent context."
- Auto-compaction at context limits preserves architectural decisions, unresolved bugs, implementation details, and the five most recently accessed files.

### Devin / persistent context

Devin (Cognition) maintains persistent context across long-running tasks via a combination of a scratchpad, long-term memory store, and session replays. Less publicly documented than Cursor/Anthropic.

### AutoGPT / BabyAGI memory (legacy)

These earlier systems (2023) used a task list + a vector memory store. They're largely historical now; the modern pattern (subagent + handoff + compaction) replaces them.

### Agent SDK approaches

- **Anthropic Agent SDK** — provides tool-use, MCP, and the new memory tool (file-based system for agents to store/consult information outside context).
- **Letta Agent SDK** — stateful agents, memory blocks, archival memory, context hierarchy, AgentFile serialization.
- **Cloudflare Agents SDK** — durable objects + state, scheduled tasks.
- **OpenAI Agents SDK** — voice agents, computer use, handoffs.

**architect-os relevance:** architect-os's four-layer model is a *project* memory architecture, not a *long-running agent* memory architecture. The distinction matters: project memory (Layer 1–3) changes slowly and is shared across all sessions; long-running-agent memory (session handoffs, compaction survivors, scratchpads) changes fast and is per-run. architect-os should explicitly distinguish these two regimes.

---

## Part 8 — Knowledge graph for code

(Synthesizing from sources above:)

**Production implementations:**

- **Aider repo map** — tree-sitter + PageRank-style ranking, token-budgeted, auto-generated per run.
- **Cursor dynamic context discovery** (Jan 6, 2026, blog post by Jediah Katz) — instead of a static graph, the agent discovers context dynamically per task.
- **Sourcegraph Cody** — SCIP-based precise code intelligence.
- **Continue.dev** — RAG-based, not graph-based.
- **Cline / Roo Code** — file-reading, not graph.
- **Greptile** — index-based codebase search.

**Academic / OSS:**

- **tree-sitter + Neo4j** — common pattern in OSS repo-graph projects.
- **CodeQL** — relational queries over a code database, primarily for security.

**No production tool** (as of July 2026) ships a *semantic* code graph with concepts, decisions, and ADRs as first-class nodes. architect-os's Layer 3 design is genuinely novel here. The trade-off is that semantic graphs can't be fully auto-generated from source — they need human curation or LLM-assisted extraction. This is exactly the gap that architect-os's "weekly distill from dumps" is trying to close, but the distill step is under-specified.

---

## What's new since mid-2025

1. **"Context engineering" named as a discipline** (Anthropic, Sep 2025). Previously implicit; now has a canonical post, a definition, and shared vocabulary (context rot, just-in-time retrieval, compaction, structured note-taking, sub-agent architectures).
2. **Memory tool in beta on Claude Developer Platform** (Anthropic) — file-based system for agents to store/consult information outside the context window. The official version of the NOTES.md pattern.
3. **Zep's temporal context graphs** reached enterprise production with S&P recognition (April 2026) — automatic invalidation, provenance, sub-200ms retrieval at 100M-graph scale.
4. **Cursor self-driving codebases** (Feb 2026) — public, detailed account of a multi-agent system running 1,000 commits/hour for a week. The freshness mechanisms (rewrite-not-append, auto-compaction, self-reflection) are validated at scale.
5. **Claude Code skills** — the always-loaded vs. on-demand split (CLAUDE.md vs. `.claude/skills/`) is now the documented Anthropic recommendation, not just a community pattern.
6. **Sub-agent architectures** became the default answer to long-horizon context limits. Anthropic's multi-agent research system, Claude Code subagents, Cursor's recursive planner/subplanner/worker all converged on this pattern.
7. **Compaction customizable via CLAUDE.md** — agents can be instructed what to preserve across compaction. Previously implicit.
8. **Letta Evals** — memory quality is now measured, not assumed. A full eval framework (suites, datasets, targets, graders, extractors, gates).
9. **Mem0's agent-signup flow** — agents mint their own API keys, claim ownership later. The *agent* is the customer.
10. **Hybrid retrieval** (some up-front + just-in-time exploration) is the empirical winner, per Anthropic. Pure RAG is no longer the default.

---

## Specific gaps and issues in architect-os's four-layer model

### Gap 1 — Layer 3 has no automatic invalidation
`repo-graph.json` nodes have `last_modified` and `last_verified` but no `valid_from` / `valid_to` / `invalidated_by` / `provenance`. Stale nodes are flagged for review, not invalidated at write time. **Fix:** add temporal fields and an explicit provenance pointer (to the dump, commit, or ADR that established the fact). On weekly distill, when new evidence contradicts an existing node, write a *new* node and mark the old one `invalidated_by: <new-node-id>`.

### Gap 2 — Layer 3 has no automated construction
The graph is hand-curated from weekly distills. No tree-sitter parsing, no symbol extraction, no dependency-graph ranking. **Fix:** add a `memory/structural-graph/` auto-generated layer (tree-sitter-based, Aider-style, token-budgeted) as the base. Layer 3 becomes the *semantic* overlay on top of the structural base.

### Gap 3 — Layer 3 has no token budget
The ego-network load is "10–50 nodes" by judgment, not by a token budget. Aider's `--map-tokens` approach is the model: a configurable budget that the graph loader respects. **Fix:** add a `--graph-token-budget` parameter and a ranking algorithm (reference-frequency, recency, ticket-relevance) that selects which nodes/edges fit.

### Gap 4 — Layer 1 content mix drifts toward "file-by-file descriptions"
Anthropic's include/exclude table explicitly excludes "file-by-file descriptions of the codebase" and "information that changes frequently." architect-os's "File conventions (10 most important paths)" is close to this. **Fix:** prune Layer 1 toward "things Claude can't figure out by reading code" — bash commands, env quirks, constitution summary, gotchas. Move path-navigation to the graph.

### Gap 5 — No agent-writable memory layer
Letta's pattern: the agent writes its own memory blocks during a run. architect-os's Layer 4 dumps are human-authored. **Fix:** add a `memory/scratchpad.md` that the agent rewrites (not appends) during the session, and a `memory/handoffs/` directory for subagent results.

### Gap 6 — No compaction customization in AGENTS.md
Anthropic now recommends CLAUDE.md carry instructions like *"When compacting, always preserve the full list of modified files and any test commands."* architect-os's AGENTS.md has no such instruction. **Fix:** add a compaction-preservation line to Layer 1.

### Gap 7 — No subagent-handoff contract
architect-os has no explicit contract for what a subagent returns. Anthropic's pattern: 1–2k token distilled summary + files/decisions/follow-ups. **Fix:** add a `memory/handoffs/<subagent-name>/<timestamp>.md` convention with a fixed schema.

### Gap 8 — No vector index
Layer 3 is pure graph. For a repo with thousands of files, ADRs, and research notes, a vector index over descriptions would complement the graph. **Fix:** add `memory/embeddings/` with embeddings over file descriptions, ADRs, and research notes. Retrieval = graph navigation + vector similarity, hybrid.

### Gap 9 — No namespace-keyed store
`repo-graph.json` is a single file. LangGraph's namespace-keyed store pattern (per-concern, per-agent subgraphs) is cleaner for multi-agent work. **Fix:** split into `memory/store/<namespace>/<key>.json`.

### Gap 10 — Freshness protocol is review-time, not write-time
Stale memory is "flagged, not deleted" and the weekly distill is the enforcement point. Zep invalidates at write time. **Fix:** move invalidation to the distill step (when new evidence arrives, invalidate the old fact then, not at the next review). The weekly distill becomes the place where invalidation decisions are *confirmed*, not *made*.

### Gap 11 — No memory quality eval
Letta ships a full eval framework. architect-os has no way to measure whether memory is actually useful. **Fix:** add a `memory/evals/` directory with a small eval set — questions the agent should be able to answer from memory, run periodically, score tracked over time.

---

## Recommended updates to `repo-memory.md`

### Layer 1 (AGENTS.md) — tighten content mix

Current content list:
> Project one-liner, Tech stack with versions, Domain language (5–10 terms), File conventions (10 most important paths), Constitution summary (5 most violated rules), Architecture style and key decisions, Gotchas (from dumps), Active constraints

Recommended:
- Keep: project one-liner, tech stack, domain language, constitution summary, gotchas, active constraints.
- Cut or link to graph: "10 most important paths," "Architecture style and key decisions" (move to docs/architecture, link from AGENTS.md).
- Add: **compaction-preservation instruction** (what to keep across compact), **subagent-handoff reference** (where to read/write handoffs), **scratchpad location** (`memory/scratchpad.md`).
- Tighten cap from ≤150 to **≤120 lines**.

### Layer 2 (docs/) — unchanged, well-aligned

The docs tree structure is solid. The only addition: a `docs/agents/handoff-template.md` for subagent results.

### Layer 3 (knowledge graph) — add temporal fields, structural base, token budget

- Add fields to every node: `valid_from`, `valid_to`, `invalidated_by`, `provenance` (path to dump, commit, or ADR).
- Add a `memory/structural-graph/` auto-generated layer (tree-sitter + PageRank-style ranking, Aider-style, token-budgeted).
- Add a `--graph-token-budget` parameter to the ego-network loader.
- Split `memory/repo-graph.json` into `memory/store/<namespace>/<key>.json` for multi-agent isolation.
- Add `memory/embeddings/` for vector retrieval over file descriptions, ADRs, research notes.

### Layer 4 (session dumps) — add agent-writable layers

- Keep `memory/dumps/` as human-authored raw material (unchanged).
- Add `memory/scratchpad.md` — agent-rewritten during the session, not appended.
- Add `memory/handoffs/<subagent>/<timestamp>.md` — subagent results with fixed schema (`{summary, files_modified, decisions_made, follow_ups}`).
- Add `SESSION_HANDOFF.md` — agent-rewritten at end of session, read first by next session.

### Freshness protocol — shift from review-time to write-time

- Add temporal invalidation to the weekly distill: when new evidence contradicts an existing node, write a new node and mark the old `invalidated_by`.
- Move "stale = hypothesis, not fact" from a flag to a state machine: `fresh → stale → invalidated → archived`.
- Add a `memory/evals/` directory with a periodic memory-quality eval.

---

## Sources

1. Anthropic — *Effective context engineering for AI agents* (Sep 29, 2025) — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
2. Anthropic — *Best practices for Claude Code* (fetched 2026-07-19) — https://www.anthropic.com/engineering/claude-code-best-practices
3. Mem0 docs (fetched 2026-07-19) — https://docs.mem0.ai/
4. Letta / MemGPT docs (fetched 2026-07-19) — https://docs.letta.com/
5. Zep homepage (fetched 2026-07-19) — https://www.getzep.com/
6. Aider — *Repository map* (fetched 2026-07-19) — https://aider.chat/docs/repomap.html
7. Cursor — *Towards self-driving codebases* (Feb 5, 2026) — https://cursor.com/blog/self-driving-codebases

**Attempted but unreachable (404/403) on 2026-07-19:** LangChain blog context-engineering posts (multiple paths), Latent Space context-engineering post, Karpathy bearblog context-engineering post, Cursor `/blog/context-engineering` direct URL, Zep `/blog` index. The Anthropic post cites the Karpathy X post directly — that citation is preserved above.
