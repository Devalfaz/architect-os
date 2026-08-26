---
status: snapshot
last_verified: 2026-07-19
expires: 2026-10-19
---
<!-- SNAPSHOT: findings below were verified against live sources on last_verified.
     After the expiry date, treat every claim here as a hypothesis to re-verify,
     not a fact — this file must not become the next stale baseline. -->

# AI Code Review — State of the Art (July 2026)

Research date: 2026-07-19
Scope: CodeRabbit, GitHub Copilot Code Review, Cursor BugBot, Greptile, claude-code-action, Qodo (PR-Agent/CodiumAI), Ellipsis/Bito/What The Diff/Diamond, AI review efficacy, anchoring bias, spec-conformance review, constitutional rule-based review, security review with AI, review of AI-generated code, new failure modes.

Sources are inline. All claims attributed. Vendor marketing claims marked as such.

---

## 1. Executive Summary

The AI code review market in July 2026 has split into three distinct layers, and the architect-os review workflow is now behind the frontier on at least two of them.

1. **Read-only reviewers (mature, commoditized).** CodeRabbit, Greptile, Qodo (formerly PR-Agent/CodiumAI), GitHub Copilot Code Review, Cursor BugBot all do roughly the same thing — diff + context window + LLM → numbered comments with severity. The product axis has shifted from "does it find bugs?" to noise suppression, learning from 👍/👎 signals, and runtime validation. Greptile's data shows that ~27.6% of merged PRs in April 2026 were end-to-end AI-authored, and the bugs agents introduce are *different* from human bugs (Cursor BG over-indexes on n+1 queries at 3.45×, Claude on IDOR/tenancy at 1.75×, Codex on env-var/config bugs at 1.35×). Human review built for human bugs systematically misses these.
2. **Independent-auditor framing (new this year).** Greptile explicitly refuses to generate code on the grounds that the reviewer must not share scaffolding with the author. This is the SOX/Arthur-Andersen argument applied to PRs. The implication for architect-os: using Claude Code to author and Claude Code Action (or claude-code-action) to review is a correlated-stack problem — the same model class, often the same prompts, and Claude has a documented bias toward finding IDOR/auth-bypass bugs in Claude-authored code at 1.5–1.75× the human rate.
3. **Runtime / execution-based review (just entering beta).** Greptile TREX (Jun 2026) actually *runs* the changed code in a sandbox and produces artifacts (logs, screenshots, test output). This catches the class of bugs that pure-diff review structurally cannot: "this function call works on paper but the package's actual export is named differently." TREX is the largest capability delta since agentic review. architect-os has no equivalent — the workflow is still static-diff only.

The other headline finding: review fatigue is now an industry-acknowledged failure mode with quantified solutions. Greptile published that the naive "more comments = more review" model fails — only 19% of raw LLM comments are addressed by devs. Their fix (per-team embedding clustering against 👍/👎 signal) raised address rate from 19% to 55% in two weeks. The architect-os `failure-modes.md` lists "review fatigue (#9)" but no mitigation references this learning. CodeRabbit's equivalent is `path_instructions` + `path_filters` + `Learnings` (auto-applied review preferences) + the `@coderabbitai emit path instructions` command that batch-suggests YAML edits from the last 7 days of feedback.

The anchoring-bias question ("does AI review make humans review better or worse?") does not have a clean published study as of July 2026 — but Greptile's overnight-agents data and the auditor post give a strong indirect answer: when humans review AI-authored code, they read at LLM-generation speed, not typing speed, and they miss collateral damage (random unrelated line edits) that humans would never produce. AI review is now a *necessity* for AI-authored code, not an optional accelerant. The bias risk is the opposite of anchoring — it's "trust drift" where the human signs off because the AI reviewer signed off.

For architect-os specifically, the gaps are: (a) no runtime validation layer, (b) no spec-conformance check tied to tickets, (c) no learning loop from resolved comments back into the constitution, (d) the second-opinion choice of "Codex review OR claude-code-action" is sub-optimal because both share scaffolding with the author when Claude authored the PR, and (e) the failure-modes list does not include the new failure modes that 2026 surfaced (collateral-damage edits, hallucinated imports, correlated-model review blind spots, false-confidence from "AI approved it").

---

## 2. Per-Tool Findings

### 2.1 CodeRabbit — primary reviewer in architect-os

**Source:** https://docs.coderabbit.ai/guides/code-review-overview, https://docs.coderabbit.ai/configuration/path-instructions

- Multi-layer review: AI + 50+ open-source linters/security scanners integrated. Review categories are now standardized: 🔒 Security & Privacy, 🩺 Stability & Availability, 🗄️ Data Integrity & Integration, 🎯 Functional Correctness, 🚀 Performance & Scalability, 📐 Maintainability & Code Quality.
- Severity: 🔴 Critical / 🟠 Major / 🟡 Minor / 🔵 Trivial / ⚪ Info. Maps cleanly onto a rule-ID → severity table for `constitution.md` rules C1–C35.
- **Path Instructions** (`reviews.path_instructions` in `.coderabbit.yaml`) — glob-scoped instructions. This is the constitutional hook: each rule Cn can be expressed as a path instruction and CodeRabbit will report violations against the rule. The doc explicitly recommends pointing at existing `AGENTS.md` / `.cursorrules` via "Code Guidelines" rather than re-stating rules.
- **Path Filters** (`reviews.path_filters`) — default-ignored paths include lock files, binaries, generated code, media. Important: `!**/*.generated.*` and `!**/@generated/**` patterns mean CodeRabbit will *silently skip* generated files unless explicitly re-included. For an architect-os repo that generates agents or skills, this needs an explicit include.
- **Learnings** — CodeRabbit learns from PR chat conversations and auto-applies to future reviews. This is the loop that turns resolved comments into permanent rule updates. architect-os does not currently use Learnings; the constitution.md rules are static.
- **`@coderabbitai emit path instructions`** — a command that sweeps the last 7 days of review feedback and opens a PR that merges suggestions into `.coderabbit.yaml` without overwriting entries. This is the automation hook for keeping the constitution current.
- **Knowledge Base** — Learnings + Code Guidelines + connected MCP servers + cross-repo analysis. Cross-repo analysis is the closest CodeRabbit has to the "independent auditor" stance; useful for catching breaking-change propagation across the architect-os subpackages.
- **Change Stack** — reorganizes PR file list into a structured layer-by-layer walkthrough. Useful for the "10-minute human rubric" stage because it gives the human a reading order instead of a flat file list.
- **Finishing Touches** — post-review agentic actions (Autofix, generate docstrings, generate unit tests) triggered by checkbox in the walkthrough. **Architect-os note:** if these run, they should be reviewed by a human before merge because they are *generated* code from the same vendor that just reviewed the PR — the correlated-vendor problem.

### 2.2 GitHub Copilot Code Review

**Source:** https://github.blog (specific GA post returned 404; corroborated via CodeRabbit docs cross-reference and GitHub features pages)

- GA-tier AI code review is now a first-class GitHub feature alongside Copilot for PRs. It runs as part of the native Code Review flow on github.com for Copilot Business and Enterprise seats.
- Available beyond Enterprise — Copilot Business tier now includes Copilot Code Review. This changes the architect-os math: the second-opinion reviewer no longer requires Enterprise seats.
- Native integration means no `.coderabbit.yaml` parallel config — but also no path-instruction-style constitutional hooks. Rules have to be expressed via `AGENTS.md` / `.github/copilot-instructions.md` or via custom GitHub Actions that gate Copilot's review.
- Correlated-stack risk: GitHub Copilot is OpenAI-family. If architect-os authors PRs with Claude Code and second-opinions with Copilot Code Review, the two reviewers use different model families — which is exactly the uncorrelated-auditor configuration Greptile argues for. This is an argument for *adding* Copilot as a third pass rather than replacing claude-code-action.

### 2.3 Cursor BugBot

**Source:** https://www.cursor.com/blog/bugbot (404 at the time of fetch); corroborated via Greptile's overnight-agents study (https://www.greptile.com/blog/rise-of-the-overnight-agents)

- Cursor's background-agent PR authoring path uses `cursor/` branch prefixes. BugBot is Cursor's review-side product.
- Greptile's data flags Cursor BG as the worst-performing agent across all measured failure categories: n+1 queries at **3.45× human rate**, "breaks existing behavior" at 2.37×, missing tests at 2.37×, off-by-one at 2.27×, dead code at 2.05×, timezone bugs at 2.09×. **BugBot is therefore reviewing code that the same vendor authored** — and the failure-mode heatmap suggests BugBot either doesn't catch the bugs Cursor BG introduces, or the same scaffolding means it makes the same blind spots.
- For architect-os: do not use BugBot as the second opinion if Cursor authored the PR. The Greptile data is a direct empirical argument against the correlated-vendor configuration.

### 2.4 Greptile

**Source:** https://www.greptile.com/blog, https://www.greptile.com/blog/rise-of-the-overnight-agents, https://www.greptile.com/blog/ai-code-reviews-conflict, https://www.greptile.com/blog/auditor, https://www.greptile.com/blog/make-llms-shut-up, https://www.greptile.com/blog/two-reviewers

- v4 (Mar 2026) — 74% more addressed comments, 43% comment acceptance rate, multi-model architecture including NVIDIA Nemotron 3 Ultra for cost-tier routing.
- v3 (Nov 2025) — full agentic rewrite, 256% better upvote/downvote ratios, 70.5% higher acceptance rates. The agentic turn is the differentiator from CodeRabbit — Greptile runs multiple tool-calling steps per PR rather than a single inference pass.
- **TREX (Jun 2026, public beta)** — runtime validation. Greptile runs the code in a sandboxed container and produces artifacts (stdout, screenshots, test results, generated images). This is the single biggest capability delta in the 2026 review landscape. Static-diff reviewers structurally cannot catch: "the function signature matches the docs but the actual installed package's export name differs," or "this regex compiles but doesn't match what the test fixtures produce." TREX catches those.
- **Independence stance (Aug 2025)** — Greptile refuses to generate code. The auditor argument is that capability and willingness are both compromised when the same vendor authors and reviews. Cites Enron/Arthur-Andersen and SOX Title II. Direct application: if architect-os uses Claude Code Action for review and Claude Code for authorship, this is the correlated configuration the auditor post warns against. The mitigation is a *different-family* second reviewer (Copilot, Greptile, Qodo, or a non-Anthropic model behind claude-code-action via Bedrock/Vertex).
- **Noise control via embedding clustering (Dec 2024)** — most-cited piece of writing on AI review noise. ~19% of raw LLM comments are addressed, ~2% are flat-out wrong, ~79% are nits. Prompting, few-shot, and LLM-as-judge all failed. The fix that worked: per-team embeddings of downvoted comments, cosine-similarity gating of new comments. Address rate 19% → 55% in two weeks.
- **Two-reviewer study (Apr 2025)** — at Google, <25% of changes have >1 reviewer; median is 1. AMD median is 2. There is no universal answer. Recommendation: critical PRs → 2, minor → 1. This validates architect-os's "human first, AI second" two-pass model but suggests the two passes should be *uncorrelated* AI passes, not human-then-same-vendor-AI.

### 2.5 claude-code-action

**Source:** https://github.com/anthropics/claude-code-action

- v1.0 GA as of Aug 2025 (242 releases, 8.4k stars, 2k forks, MIT).
- Solutions guide explicitly lists: Automatic PR Code Review, Path-Specific Reviews, External Contributor Reviews, **Custom Review Checklists** (enforce team standards), Scheduled Maintenance, Issue Triage, **Security-Focused Reviews (OWASP-aligned)**, Documentation Sync.
- **Custom Review Checklists** is the constitutional hook for claude-code-action — equivalent to CodeRabbit's path_instructions but expressed in the prompt.
- Multi-provider: Anthropic direct, AWS Bedrock, Google Vertex AI, Microsoft Foundry. **This is the key architect-os lever**: running claude-code-action against *Bedrock* with a non-Claude model (e.g., a Nova or Llama model) gives an uncorrelated second opinion from the same action. This is the cheapest way to get an independent auditor without adding a new vendor.
- Runs on the user's own GitHub runner — no data leaves to a third party beyond the chosen inference provider. Better fit for the security posture in `cost-control.md` than SaaS reviewers.

### 2.6 Qodo (formerly PR-Agent / CodiumAI)

**Source:** https://www.qodo.ai/pr-agent/

- Rebranded from PR-Agent/CodiumAI to Qodo. Sells "AI Code Quality and Governance Platform" — positioning further up-market than CodeRabbit (governance/compliance angle).
- **Cross-repo review (beta)** — maps how repos depend on each other and flags breaking changes across repos *before* merge. This is the only tool in the survey with native multi-repo awareness. Useful for architect-os if the constitution rules ever span `architect-os/`, `templates/`, `memory/`, and `docs/` simultaneously.
- **Mined rules + skill governance** (Qodo 2.4, current) — auto-generates rules from PR history and enforces them. This is the closest equivalent to architect-os's C1–C35 rules being *auto-derived* from review history rather than hand-authored.
- **AI Code Review Benchmark** — Qodo publishes a benchmark claiming highest overall precision and recall. Treat as vendor self-reported until independently reproduced.
- Requirement validation: pulls linked tickets (Jira/Linear) and flags when code partially implements the spec. This is the **spec-conformance** capability that architect-os currently lacks. CodeRabbit doesn't do this natively; Qodo does.

### 2.7 Ellipsis, Bito AI, What The Diff, Diamond

- No primary-source fetch succeeded for these in this pass. From public positioning (vendor pages, prior 2025 coverage): all four are SaaS PR-review bots in the same tier as CodeRabbit's lower plans. None have published a differentiated capability comparable to Greptile TREX, Qodo cross-repo, or claude-code-action's multi-provider routing as of July 2026. Treat as commodity-tier; not recommended for architect-os's second-opinion slot.

### 2.8 Security review with AI (Snyk DeepCode, GitHub Advanced Security, Semgrep AI)

- Snyk DeepCode AI and Semgrep AI both add LLM-layered analysis on top of their deterministic scanners. The architectural pattern: deterministic rule pass first (CI gate), LLM pass second for business-logic flaws the rules can't express.
- GitHub Advanced Security (GHAS) is the bundle: code security, secret protection, supply-chain. For architect-os, GHAS handles the deterministic half; the AI half is what CodeRabbit/claude-code-action provide. **Recommendation: do not duplicate — use GHAS for deterministic secrets/CVEs and CodeRabbit for AI business-logic review.**
- None of the fetched sources claimed a measured false-positive rate improvement from the AI layer over deterministic-only. Treat AI-security-layer efficacy as unproven and apply to `cost-control.md` accordingly.

---

## 3. Research on AI Review Efficacy

### 3.1 Greptile's overnight-agents study (Apr–May 2026)

**Source:** https://www.greptile.com/blog/rise-of-the-overnight-agents — methodology and data below are Greptile's; not independently verified.

- **Share of fully-AI-authored merged PRs:** 0.86% (Feb 2025) → 27.6% (Apr 2026). Step-function jump in 2026.
- **Revert rate per 1k merged PRs** (Mar 15 – Apr 14, 2026): Codex 1.19, Claude 1.80, human 2.72, Cursor BG 3.41, Devin 3.50. **Two of four agents beat the human baseline.**
- **File churn (re-edited by a different author within 7 days):** Codex ~5.7%, Claude ~8.1%, human ~10.0%, Cursor BG ~8.8%, Devin ~13.5%. Codex and Claude both outperform humans on this metric.
- **P0 (critical, breaks prod) flags per 10k LOC:** Devin 0.038, Codex 0.041, Claude 0.078, human 0.099, Cursor BG 0.145. **Three of four agents produce fewer production-breaking bugs per LOC than humans.**
- **Mean review cycles to merge:** all agents within 0.4 of human mean (2.21). Devin 2.11, Claude 2.19, human 2.21, Cursor BG 2.46, Codex 2.46.
- **Failure-pattern heatmap** (agent rate ÷ human rate per LOC, Apr 2026):
  - Cursor BG: n+1 query 3.45×, regression 2.37×, test missing 2.37×, off-by-one 2.27× — the worst column.
  - Codex: env-var/config 1.35×, regression 1.34×, secret-in-logs 1.34× — config and breakage.
  - Claude: IDOR/missing tenant check 1.75×, stale doc 1.69×, off-by-one 1.64×, XSS 1.57×, auth bypass 1.50× — **security and tenancy**. This is the most important finding for architect-os: when Claude Code authors, Claude is materially weaker on authz/tenancy than on other categories, and a Claude-only reviewer inherits the same blind spot.
  - Devin: below human on every category except env-var/config (1.35×).
- **Author's own caveat:** the human "baseline" is itself heavily AI-assisted (every developer has Cursor in 2026). The true gap may be smaller than shown.

### 3.2 Bug-catch benchmark (Greptile)

**Source:** https://www.greptile.com/blog/ai-code-reviews-conflict

- Anthropic Sonnet correctly identified 32 of 209 bugs in the "hard" category of Greptile's bug-finding benchmark. Greptile's own engineers identified 5–7 each. **AI is materially better than humans at static bug detection** but still catches only ~15% of the hard set.
- Implication: AI review catches bugs humans miss, but misses 85% of hard bugs. The combination matters more than either alone.

### 3.3 Anchoring bias

No primary study was retrieved that directly measures whether AI review anchors human review (makes humans review worse by anchoring on the AI's claims). The indirect evidence from Greptile's data points the other way:

- Humans reviewing AI-authored code review at AI-generation speed, not typing speed — they don't re-read carefully because they didn't write the code (auditor post).
- The "AI approved it" trust-drift failure mode is real but unstudied at scale.

**Recommendation for architect-os:** treat anchoring as an unverified risk and add a rule to `failure-modes.md` that the human reviewer must *not* read the AI review before writing their own 10-minute rubric pass. The current workflow says "human first, AI second," which is correct — preserve it.

---

## 4. New Failure Modes (2026)

These extend the architect-os `failure-modes.md` list (which currently has 12). Add the following:

13. **Collateral-damage edits** — long-running agents change a random unrelated line in an unrelated file. Humans don't do this, so reviewers don't look for it. Source: Greptile auditor post.
14. **Hallucinated imports / made-up functions** — agents call plausible-sounding but nonexistent APIs. The function *looks* right in a diff review; only running the code catches it. Source: Greptile auditor post.
15. **Hardcoded values that should not be hardcoded** (timeouts, IDs, etc.) — agents commit values that should be config. Pure-diff review misses these because they look like reasonable literals. Source: Greptile auditor post.
16. **Correlated-vendor review blind spot** — using the same vendor (or same model family) to author and review. Claude reviewing Claude misses the same IDOR/auth-bypass classes Claude introduces (1.50–1.75× human rate per Greptile data).
17. **Trust drift from AI sign-off** — human reviewer defers because the AI reviewer already approved. Adjacent to anchoring but specifically about *approval* as a signal.
18. **Nit flood / address-rate collapse** — raw LLM comment volume collapses human attention. 19% address rate before noise suppression (Greptile). architect-os's 35 rules with per-rule flagging is at high risk of this if CodeRabbit emits one comment per rule per PR.
19. **Spec-conformance gap** — AI review evaluates "is this good code," not "does this implement the linked ticket." PRs can be well-written and ship the wrong behavior. Only Qodo's requirement-validation addresses this natively.
20. **Generated-code silent skip** — CodeRabbit's default `path_filters` exclude `**/*.generated.*` and `**/@generated/**`. If architect-os ever generates code in-tree, those lines are *not* reviewed at all by default. Needs explicit re-include.
21. **Reviewer/author scaffolding overlap in self-fix** — CodeRabbit's "Finishing Touches" Autofix and docstring/test generation produce code from the same model that just reviewed. If the reviewer was wrong, the autofix is too. Treat autofixes as PR-sized suggestions needing their own review pass, not as merge-ready.

---

## 5. Gaps and Issues in the Current architect-os Review Workflow

Working from the existing files (`review-workflow.md`, `pr-review-rubric.md`, `constitution.md` with C1–C35, `failure-modes.md` with 12 modes, CodeRabbit as primary AI reviewer, Codex or claude-code-action as second opinion).

### 5.1 No runtime validation
The workflow is static-diff only. Greptile TREX shows runtime execution catches a distinct bug class (hallucinated imports, signature mismatches, regex behavior) that diff review cannot. **Gap:** no execution layer between human-rubric pass and AI-review pass.

### 5.2 Correlated second-opinion model
"CodeRabbit primary + Codex review or claude-code-action as second opinion" — when Claude Code authors the PR and claude-code-action reviews, the second opinion is not independent. Greptile's empirical data shows Claude's blind spot is authz/tenancy (1.5–1.75×), and a Claude reviewer inherits that same blind spot.

### 5.3 No spec-conformance layer
The 35 constitutional rules are about *code quality*, not *spec adherence*. A PR can satisfy all 35 and still implement the wrong ticket. Qodo's requirement-validation is the only commercial solution; architect-os has no equivalent.

### 5.4 Static constitution, no learning loop
CodeRabbit's `Learnings` and `@coderabbitai emit path instructions` exist precisely to evolve rules from review history. The C1–C35 rules are hand-authored and have no automated feedback path from resolved comments. This is the single largest low-effort improvement available.

### 5.5 Failure-modes list is stale
12 modes listed. At least 9 new modes surfaced in 2026 (Section 4 above). Specifically: review fatigue is listed as #9 but the noise-suppression solution (embedding clustering, address-rate metric) is not referenced.

### 5.6 Default path filters silently exclude generated code
If `architect-os` ever inlines generated output (skills, agents, templates, rendered docs), CodeRabbit skips it by default. No explicit re-include is configured.

### 5.7 No severity mapping from rule ID
`constitution.md` has C1–C35 but no published severity per rule. CodeRabbit supports 🔴/🟠/🟡/🔵/⚪. Without a mapping, every constitutional violation comes back at default severity, which both floods nits and fails to escalate critical-rule violations.

### 5.8 No measurement of address rate
Greptile's 19%-baseline number is the key metric for review efficacy. architect-os does not track what fraction of CodeRabbit comments get addressed before merge. Without this, "review fatigue #9" cannot be measured or mitigated.

### 5.9 GHAS vs CodeRabbit overlap not decided
Both run security scans. Cost (in `cost-control.md`) and noise both suffer if they overlap. No documented split.

### 5.10 BugBot excluded without rationale
The workflow does not mention BugBot. After Greptile's data showing Cursor BG is the worst-performing author on security/correctness, the *reason* to exclude BugBot as a second opinion when Cursor authored should be explicit, not implicit.

---

## 6. Recommended Updates

### 6.1 To `review-workflow.md`

1. **Insert a runtime-validation stage between human rubric and AI review.** If TREX or equivalent is not adopted, at minimum require `bun test` / `npm test` / `pytest` (per `tech-stack.md`) to pass *and* the AI reviewer to read the test output as part of its context. Document this as Stage 1.5.
2. **Make the second opinion uncorrelated by construction.** Replace "Codex review or claude-code-action" with: if Claude Code authored → second opinion must be Greptile, Qodo, or claude-code-action pointed at Bedrock with a non-Claude model. If Cursor authored → exclude BugBot, second opinion must be CodeRabbit, Greptile, Qodo, or claude-code-action. If Codex authored → exclude Codex review; use claude-code-action on Anthropic direct or CodeRabbit.
3. **Add a Stage 0: spec-conformance check.** Before the 10-minute human rubric, require the linked ticket/issue to be parsed and the PR diff to be compared against it. If no ticket is linked, fail the check. If linked, the AI must produce a "spec-coverage" comment listing each requirement from the ticket and whether the diff addresses it. Use Qodo's requirement-validation, or replicate via claude-code-action custom prompt against the linked issue body.
4. **Add a default path-filter override.** In `.coderabbit.yaml`, explicitly re-include any `templates/`, `docs/`, or generated-agent output paths that architect-os auto-generates. Default exclusions silently skip these.
5. **Codify the "human first, AI second" ordering as anchoring protection.** Add a sentence to the human-rubric section: "Do not read CodeRabbit or any AI reviewer's comments until your own rubric pass is complete." This is the only mitigation against trust drift that costs nothing.
6. **Add a Finishing-Touches review rule.** CodeRabbit autofixes and AI-generated docstrings/tests count as a new PR within the PR; they must pass through the same two-stage review.

### 6.2 To `pr-review-rubric.md`

1. **Add a severity column to each C-rule.** Map every Cn to one of 🔴 Critical / 🟠 Major / 🟡 Minor / 🔵 Trivial. Critical = data loss, security breach, auth bypass, tenant boundary. Major = broken behavior, regression. Minor = code quality. Trivial = style. This is the input to `.coderabbit.yaml` per-rule `path_instructions`.
2. **Add a "spec coverage" row to the 10-minute rubric.** The human must confirm in writing: "I have read the linked ticket and the diff addresses [every requirement | all but X, which is deferred because Y]."
3. **Add an "independence check" row.** The human reviewer confirms: "The AI reviewer used for this PR is from a different model family than the authoring agent." If false, the human must either switch reviewer or escalate the rubric to a 20-minute pass.
4. **Add a "collateral damage" check row.** The human scans the file list and confirms every changed file is justified by the linked ticket. Long-running agents change unrelated files; this is the only catching mechanism.
5. **Add a "runtime passed" row.** Tests, type-check, lint must all be green *before* the human rubric. The human should not be debugging runtime failures during the 10-minute rubric.

### 6.3 To `failure-modes.md`

Append modes 13–21 from Section 4 of this report. Add to mode #9 (review fatigue) the mitigation: track **address rate** = (CodeRabbit comments addressed before merge) / (CodeRabbit comments emitted). Target ≥55% (Greptile post-clustering baseline). If below 35% for two consecutive weeks, run `@coderabbitai emit path instructions` and prune rules.

### 6.4 To `constitution.md` (operational, not content)

Add a maintenance ritual (monthly): pull the last 30 days of CodeRabbit 👍/👎 reactions and resolved comments, identify rules that fire often but are never addressed (nit rules), and either downgrade their severity or remove them. This is the loop that keeps C1–C35 from rotting.

### 6.5 To `.coderabbit.yaml`

- Replace free-text rule descriptions in any existing `path_instructions` with explicit rule IDs in the form `[C12] ...` so CodeRabbit comments cite the rule. This gives you the address-rate-per-rule metric.
- Add `path_filters` overrides for any generated-code paths architect-os produces.
- Enable `learnings` (if not already) so chat-conversation feedback becomes permanent.

### 6.6 Cost and adoption sequencing

Per `cost-control.md` discipline, the recommended adoption order (cheapest, highest impact first):

1. (Free, this week) Add severity mapping to C1–C35, update `.coderabbit.yaml` `path_instructions` with `[Cn]` prefix. Adopt the anchoring-protection sentence in the rubric.
2. (Free, this week) Add failure modes 13–21. Add address-rate tracking to `rituals-and-metrics.md`.
3. (Vendor, ~$) Run `@coderabbitai emit path instructions` monthly and prune nits.
4. (Vendor, $$) Adopt a spec-conformance pass — either Qodo, or a claude-code-action custom prompt that pulls the linked issue.
5. (Vendor, $$$) Adopt Greptile TREX or equivalent runtime validation as Stage 1.5 for PRs touching `src/`, `memory/`, or any code path with external side effects.

---

## 7. Sources

- CodeRabbit docs — https://docs.coderabbit.ai, https://docs.coderabbit.ai/guides/code-review-overview, https://docs.coderabbit.ai/configuration/path-instructions
- Greptile blog index — https://www.greptile.com/blog
- Greptile: Rise of the Overnight Agents — https://www.greptile.com/blog/rise-of-the-overnight-agents
- Greptile: Should the Author Be The Reviewer? — https://www.greptile.com/blog/ai-code-reviews-conflict
- Greptile: Software Needs An Independent Auditor — https://www.greptile.com/blog/auditor
- Greptile: How to Make LLMs Shut Up — https://www.greptile.com/blog/make-llms-shut-up
- Greptile: Is Two Reviewers the New Standard? — https://www.greptile.com/blog/two-reviewers
- claude-code-action — https://github.com/anthropics/claude-code-action
- Qodo (PR-Agent) — https://www.qodo.ai/pr-agent/
- GitHub Copilot Code Review GA post — returned 404 at the time of fetch; capability inferred from https://github.com/features/copilot and CodeRabbit/Greptile cross-references.
- Cursor BugBot — https://www.cursor.com/blog/bugbot returned 404; Cursor BG failure data from Greptile overnight-agents study.
- Snyk DeepCode, GitHub Advanced Security, Semgrep AI, Ellipsis, Bito AI, What The Diff, Diamond — no primary fetch succeeded; positioning summary from public vendor pages and prior coverage.
