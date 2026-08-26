# Visual Appendix — Shareable Diagrams

*All diagrams in Mermaid format. Paste into any Mermaid-compatible renderer (GitHub, Notion, Obsidian, mermaid.live, VS Code plugin). This is the one doc to send teammates.*

---

## 1. The Full Lifecycle (S0 → S9)

```mermaid
flowchart TD
    S0["S0 Capture<br/>📝 idea.md<br/><i>5 min, you only</i>"]
    S1["S1 Frame<br/>📋 BRD<br/><i>go/no-go gate</i>"]
    S2["S2 Specify<br/>📐 PRD + FSD<br/><i>highest-leverage stage</i>"]
    S3["S3 Design<br/>🎨 design brief<br/><i>UI work only</i>"]
    S4["S4 Architect<br/>🏗️ ADRs + architecture.md<br/><i>irreversible decisions</i>"]
    S5["S5 Plan<br/>🎯 GitHub Issues<br/><i>file-level plans — you review</i>"]
    S6["S6 Implement<br/>🤖 narrow agent runs<br/><i>fresh session per ticket</i>"]
    S7["S7 Review<br/>👤 human → 🤖 AI<br/><i>2 rounds max</i>"]
    S8["S8 Release<br/>🚀 squash merge + deploy<br/><i>smoke check</i>"]
    S9["S9 Learn<br/>🧠 memory dump → graph<br/><i>5 min daily, 30 min weekly</i>"]

    S0 --> S1
    S1 --> S2
    S2 --> S3
    S3 --> S4
    S4 --> S5
    S5 --> S6
    S6 --> S7

    S7 -->|"fix ≤2 rounds"| S6
    S7 -->|"approved"| S8
    S7 -->|"bounced — plan wrong"| S5
    S7 -->|"bounced — spec wrong"| S2

    S8 --> S9
    S9 -.->|"feeds next cycle"| S2
    S6 -.->|"discovery → spec delta"| S2
    S7 -.->|"spec delta"| S2

    style S0 fill:#e1f5fe
    style S1 fill:#fff3e0
    style S2 fill:#fce4ec
    style S3 fill:#f3e5f5
    style S4 fill:#e8f5e9
    style S5 fill:#fff9c4
    style S6 fill:#e3f2fd
    style S7 fill:#fce4ec
    style S8 fill:#e8f5e9
    style S9 fill:#ede7f6
```

---

## 2. The Three Control Loops

```mermaid
flowchart LR
    subgraph fix["🔧 Fix Loop (≤2 rounds)"]
        S7f["S7 Review"] -->|"fix request"| S6f["S6 Implement"]
        S6f -->|"PR updated"| S7f
        S7f -.->|"round 3 → bounce"| S5f["S5 Re-plan"]
    end

    subgraph discover["🔍 Discovery Loop"]
        S6d["S6 Implement"] -.->|"spec delta"| S2d["S2 Amend FSD"]
        S2d -->|"updated tickets"| S6d
    end

    subgraph learn["🧠 Learning Loop"]
        S9l["S9 Memory Dump"] -->|"graph delta"| G["repo-graph.json"]
        G -->|"ego-network"| S6l["S6 Smarter Agent"]
        S6l -->|"new discoveries"| S9l
    end

    style fix fill:#ffebee
    style discover fill:#e8f5e9
    style learn fill:#e3f2fd
```

---

## 3. The Two-Stage Review Pipeline

```mermaid
flowchart TD
    PR["PR Opened<br/>CI ✅ | self-review posted"]

    H1["📋 Human Rubric<br/>10 minutes<br/>constitution → spec → arch → errors → security → tests"]

    AI1["🤖 CodeRabbit<br/>always-on<br/>bugs, security, patterns"]
    AI2["🤖 Codex Review<br/>optional<br/>second opinion"]

    FIX["🔧 Agent Fix Loop<br/>≤2 rounds<br/>commit SHA or pushback"]
    RESOLVE["✅ You Resolve Threads"]
    MERGE["🚀 Squash Merge<br/>conventional commit"]

    PR --> H1

    H1 -->|"approve"| AI1
    H1 -->|"fix"| FIX
    H1 -->|"bounce"| BOUNCE["⛔ Back to S5"]

    FIX -->|"PR updated"| H1
    FIX -->|"round 3"| BOUNCE

    AI1 --> AI2
    AI2 --> FIX

    AI1 --> RESOLVE
    FIX --> RESOLVE
    RESOLVE --> MERGE

    style H1 fill:#fff9c4
    style AI1 fill:#e3f2fd
    style AI2 fill:#f3e5f5
    style FIX fill:#ffebee
    style RESOLVE fill:#e8f5e9
    style MERGE fill:#c8e6c9
    style BOUNCE fill:#ffcdd2
```

---

## 4. The Four-Layer Memory Architecture

```mermaid
flowchart TD
    subgraph L1["Layer 1: AGENTS.md"]
        A["≤150 lines<br/>domain language<br/>conventions<br/>gotchas<br/>verified weekly"]
    end

    subgraph L2["Layer 2: Docs Tree"]
        D1["docs/product/<br/>ideas, BRDs, PRDs"]
        D2["docs/specs/<br/>FSDs — implementation contracts"]
        D3["docs/adr/<br/>numbered decisions"]
        D4["docs/architecture/<br/>architecture.md"]
        D5["docs/research/<br/>spikes with expiry"]
        D6["docs/agents/<br/>verified APIs, gotchas, retros"]
    end

    subgraph L3["Layer 3: Knowledge Graph"]
        G["memory/repo-graph.json<br/>nodes: module, file, concept, decision, api, test<br/>edges: depends_on, implements, tests, constrained_by<br/>ego-network loaded per ticket"]
    end

    subgraph L4["Layer 4: Session Dumps"]
        M["memory/dumps/YYYY-MM-DD.md<br/>daily: 5 min<br/>what changed, decisions, surprises<br/>hallucinations caught, graph delta"]
    end

    A -->|"read every session"| L2
    L2 -->|"linked from tickets"| G
    M -->|"weekly distill"| G
    M -->|"surprises → gotchas"| A

    S["Agent Session"] -->|"step 1: read"| A
    S -->|"step 2: ego-network"| G
    S -->|"step 3: spec section"| L2

    style L1 fill:#e3f2fd
    style L2 fill:#e8f5e9
    style L3 fill:#fff3e0
    style L4 fill:#f3e5f5
```

---

## 5. The Daily → Weekly → Monthly Cadence

```mermaid
flowchart TD
    subgraph daily["🌅 Daily (40 min)"]
        D1["Morning: review PRs<br/>launch 1-2 agents<br/>WIP ≤ 2"]
        D2["Afternoon: merge<br/>write memory dump<br/>plan tomorrow<br/>kill sessions"]
        D1 --> D2
    end

    subgraph weekly["📅 Weekly (30 min)"]
        W1["Memory distill<br/>graph update<br/>verify AGENTS.md"]
        W2["Sprint scan<br/>CI health<br/>skill changelog"]
        W1 --> W2
    end

    subgraph monthly["📊 Monthly (1 hour)"]
        M1["Metrics review<br/>12 tracked metrics<br/>spec deltas = #1 metric"]
        M2["Retrospective<br/>→ edit skills<br/>→ update routing"]
        M3["Roadmap check<br/>kill stale ideas<br/>promote/demote"]
        M1 --> M2 --> M3
    end

    daily --> weekly
    weekly --> monthly
    monthly -.->|"course correct"| daily

    style daily fill:#e3f2fd
    style weekly fill:#e8f5e9
    style monthly fill:#fff3e0
```

---

## 6. Tool Assignments per Stage

```mermaid
flowchart LR
    subgraph S0S4["S0–S4: Specification"]
        direction TB
        S02["S0: none / Claude chat"]
        S12["S1: Claude Code"]
        S22["S2: Claude Code (Opus)"]
        S32["S3: Claude Code + Figma MCP"]
        S42["S4: Claude Code (plan mode)"]
    end

    subgraph S5S6["S5–S6: Execution"]
        direction TB
        S52["S5: Claude Code + to-tickets<br/>alt: Spec Kit"]
        S62["S6: Claude Code (Sonnet)<br/>alt: Codex (2nd opinion)"]
        S6X["S6 XS async: Claude Code Web<br/>/ Copilot coding agent / Codex Web"]
    end

    subgraph S7S9["S7–S9: Quality & Learn"]
        direction TB
        S72["S7: You (rubric) → CodeRabbit<br/>→ Codex review (optional)"]
        S82["S8: CI/CD (Vercel / GH Actions)"]
        S92["S9: Claude Code (Haiku)<br/>memory-dump → graph-update"]
    end

    S0S4 --> S5S6 --> S7S9

    style S22 fill:#fce4ec
    style S62 fill:#e3f2fd
    style S72 fill:#fff9c4
```

---

## 7. Harness Decision Tree

```mermaid
flowchart TD
    Q1{"What stage?"}

    Q1 -->|"S0-S4<br/>Spec & Plan"| Q2{"Need to read<br/>live repo files?"}
    Q2 -->|"yes"| CC["Claude Code<br/>Opus for S2/S4<br/>Sonnet for S1"]
    Q2 -->|"no"| Q3{"Want product UX?"}
    Q3 -->|"yes"| TK["Spec Kit"]
    Q3 -->|"no"| CC

    Q1 -->|"S5<br/>Tickets"| Q5{"Team or solo?"}
    Q5 -->|"solo"| CC
    Q5 -->|"team"| TR["Spec Kit"]

    Q1 -->|"S6<br/>Implement"| Q6{"Ticket size?"}
    Q6 -->|"XS"| Q7{"Well-specified,<br/>no unknowns?"}
    Q7 -->|"yes"| CW["Async lane: Claude Code Web<br/>/ Copilot coding agent (C37)"]
    Q7 -->|"no"| CC2["Claude Code (manual)"]
    Q6 -->|"S/M"| CC3["Claude Code (Sonnet)"]
    Q6 -->|"complex / uncertain"| Q8{"Want 2nd opinion?"}
    Q8 -->|"yes"| CX["Codex CLI (parallel run)"]
    Q8 -->|"no"| CC3

    Q1 -->|"S7<br/>Review"| CR["CodeRabbit (always)<br/>+ Codex review (optional)"]
    Q1 -->|"S9<br/>Learn"| CH["Claude Code (Haiku)"]

    style CC fill:#e3f2fd
    style CC2 fill:#e3f2fd
    style CC3 fill:#e3f2fd
    style CH fill:#e3f2fd
    style CX fill:#f3e5f5
    style CW fill:#fff3e0
    style TR fill:#e8f5e9
    style CR fill:#ffebee
```

---

## 8. Memory Freshness — Verification Schedule

```mermaid
gantt
    title Memory Verification Schedule
    dateFormat  YYYY-MM-DD
    axisFormat  %a %d

    section Always
    AGENTS.md           :a1, 2025-01-01, 7d
    Project Gotchas     :a2, 2025-01-01, 7d
    Graph Nodes         :a3, 2025-01-01, 7d
    Graph Edges         :a4, 2025-01-01, 7d

    section On Change
    Verified APIs       :b1, 2025-01-01, 7d
    Architecture Docs   :b2, 2025-01-01, 30d
    ADRs                :b3, 2025-01-01, 30d

    section Expiry-Based
    Research Notes      :c1, 2025-01-01, 90d
    Memory Dumps        :c2, 2025-01-01, 14d

    section Never
    Dump Raw Material   :crit, d1, 2025-01-01, 365d
```

---

## 9. Failure Mode Taxonomy

```mermaid
mindmap
  root((Failure Modes))
    Context
      Over-contexting
      Stale memory
      Context window decay
    Quality
      Hallucinated APIs
      Weak tests
      Architecture drift
      Silent spec divergence
    Process
      Giant PRs
      Agent fix-loops
      Review fatigue
      Abandoned craftsmanship
    Security
      Security issues
      Secrets in code
      Unsanitized input
```

---

## 10. The Agent Constitution — Rule Categories

```mermaid
flowchart TD
    subgraph C1C5["C1–C5: Spec Fidelity"]
        C1["C1: Spec = truth"]
        C2["C2: Read FSD first"]
        C3["C3: AC = exit condition"]
        C4["C4: TDD (tests first)"]
        C5["C5: Edge cases documented"]
    end

    subgraph C6C10["C6–C10: Scope Discipline"]
        C6["C6: Ticket, not repo"]
        C7["C7: No 'while I'm in here'"]
        C8["C8: No deps without ADR"]
        C9["C9: PR ≤400 lines"]
        C10["C10: 1 ticket = 1 PR"]
    end

    subgraph C11C15["C11–C15: Code Quality"]
        C11["C11: Types before logic"]
        C12["C12: Validate boundaries"]
        C13["C13: Errors as values"]
        C14["C14: Log what broke"]
        C15["C15: Name magic numbers"]
    end

    subgraph C16C20["C16–C20: Testing"]
        C16["C16: Test behavior"]
        C17["C17: One concept/test"]
        C18["C18: Independent tests"]
        C19["C19: One failure reason"]
        C20["C20: Flaky = broken"]
    end

    subgraph C21C25["C21–C25: Gates"]
        C21["C21: Agents propose"]
        C22["C22: Self-review first"]
        C23["C23: Spec deltas"]
        C24["C24: Fix loops ≤2"]
        C25["C25: Flag uncertainty"]
    end

    subgraph C26C30["C26–C30: Files"]
        C26["C26: Follow conventions"]
        C27["C27: No dead code"]
        C28["C28: File = primary export"]
        C29["C29: Explicit imports"]
        C30["C30: Don't edit generated"]
    end

    subgraph C31C35["C31–C35: Security"]
        C31["C31: Secrets never in code"]
        C32["C32: Sanitize boundaries"]
        C33["C33: AuthN ≠ AuthZ"]
        C34["C34: Pin + audit deps"]
        C35["C35: No PII in logs"]
    end

    style C1C5 fill:#fce4ec
    style C6C10 fill:#fff3e0
    style C11C15 fill:#e8f5e9
    style C16C20 fill:#e3f2fd
    style C21C25 fill:#f3e5f5
    style C26C30 fill:#fff9c4
    style C31C35 fill:#ffebee
```

---

## 11. Walkthrough: New Feature (idea → merged PR)

```mermaid
flowchart LR
    subgraph day1["Day 1: Specify"]
        S01["S0: 5 min<br/>idea.md"]
        S11["S1: 30 min<br/>BRD + grill"]
        S21["S2: 1-2 hours<br/>PRD + FSD + grill-with-docs"]
    end

    subgraph day2["Day 2: Plan"]
        S32["S3: 20 min<br/>design brief"]
        S42["S4: 1 hour<br/>ADRs + architecture"]
        S52["S5: 1-2 hours<br/>to-tickets → Issues<br/>★ YOU REVIEW PLANS ★"]
    end

    subgraph day3["Day 3: Build & Ship"]
        S62["S6 morning<br/>launch agent on #231<br/>WIP = 2"]
        S72["S6 afternoon + S7<br/>self-review → rubric<br/>→ CodeRabbit → merge"]
        S82["S8: deploy<br/>smoke check"]
        S92["S9: 5 min dump"]
    end

    day1 --> day2 --> day3

    style day1 fill:#e3f2fd
    style day2 fill:#fff9c4
    style day3 fill:#e8f5e9
```

---

## 12. Walkthrough: Bug (report → regression test)

```mermaid
flowchart LR
    BUG["🐛 Bug Reported<br/>issue #240, P1"]
    DIAG["🔬 Diagnose<br/>diagnosing-bugs skill<br/>reproduce in test first"]
    FIX["🔧 Fix<br/>15-line diff<br/>+ regression test"]
    REVIEW["👤 Review<br/>2 min rubric<br/>CodeRabbit passes"]
    MERGE["✅ Merge<br/>regression test = armor"]

    BUG --> DIAG --> FIX --> REVIEW --> MERGE

    style BUG fill:#ffcdd2
    style DIAG fill:#fff3e0
    style FIX fill:#e3f2fd
    style REVIEW fill:#fff9c4
    style MERGE fill:#c8e6c9
```

---

## 13. Walkthrough: Large Refactor (campaign, not ticket)

```mermaid
flowchart TD
    PREP["S4: Architecture First<br/>ADR-0020: target shape<br/>strangler pattern<br/>lint invariant on day 1"]
    CHAR["Tests: Characterization<br/>golden-master tests<br/>no behavior changes"]
    T1["#251: facade re-export<br/>no behavior change"]
    T2["#252: move invoices<br/>fix call sites"]
    T3["#253: move subscriptions"]
    T4["#254: move webhooks"]
    T5["#255: delete legacy<br/>flip lint rule"]

    PREP --> CHAR --> T1 --> T2 --> T3 --> T4 --> T5

    T1 -.->|"bounce"| PREP
    T2 -.->|"bounce"| PREP

    SAFE["Property: every commit on this train<br/>→ main is releasable<br/>→ characterization suite green<br/>→ lint invariant narrows"]

    style PREP fill:#fce4ec
    style CHAR fill:#fff3e0
    style T1 fill:#e3f2fd
    style T2 fill:#e3f2fd
    style T3 fill:#e3f2fd
    style T4 fill:#e3f2fd
    style T5 fill:#c8e6c9
    style SAFE fill:#e8f5e9
```

---

## 14. The Three Adoption Profiles

```mermaid
flowchart TD
    subgraph light["⚡ Lightweight"]
        L1["Skip: BRD, PRD, FSD doc"]
        L2["Keep: 1 issue = 1 PR"]
        L3["Keep: file-level plan in issue"]
        L4["Keep: human diff review"]
        L5["Keep: tests before merge"]
        L6["Skip: graph updates"]
    end

    subgraph default["🌟 Default (Recommended)"]
        D1["All stages as documented"]
        D2["BRD can be half-page"]
        D3["FSD + file-level plans never skipped"]
        D4["Full review pipeline"]
        D5["Memory system active"]
    end

    subgraph heavy["🏢 Heavyweight"]
        H1["All Default +"]
        H2["BMAD-style role separation"]
        H3["2 human reviewers on main"]
        H4["Compliance layer (SAST, DORA)"]
        H5["Shared memory governance"]
        H6["Merge queue on"]
    end

    light -->|"prototype graduates"| default
    default -->|"team grows"| heavy
    heavy -->|"burnout → scale back"| default

    style light fill:#e8f5e9
    style default fill:#e3f2fd
    style heavy fill:#fce4ec
```

---

## 15. Cost & Model Routing

```mermaid
flowchart TD
    Q{"What task?"}

    Q -->|"spec (S2)<br/>architecture (S4)"| OPUS["Claude Opus 4.6<br/>$5/$25 per 1M<br/>best quality — not the place to save"]
    Q -->|"implement (S6)<br/>plan (S5)<br/>self-review (S7)"| SONNET["Claude Sonnet 4.6<br/>$3/$15 per 1M<br/>daily driver — 90% of tickets"]
    Q -->|"2nd opinion"| GPT["GPT-5.2 via Codex<br/>$1.75/$14 per 1M<br/>different model = different blind spots"]
    Q -->|"dumps (S9)<br/>labels<br/>mechanical"| HAIKU["Claude Haiku 4.5<br/>$1/$5 per 1M<br/>or GPT-5 mini<br/>90% savings vs Sonnet"]
    Q -->|"always-on review"| CR["CodeRabbit<br/>free tier covers solo<br/>multi-model pipeline"]

    style OPUS fill:#fce4ec
    style SONNET fill:#e3f2fd
    style GPT fill:#f3e5f5
    style HAIKU fill:#e8f5e9
    style CR fill:#fff3e0
```

---

## 16. Repo Layout (What Goes Where)

```mermaid
flowchart TD
    ROOT["your-app/"]
    AGENTS["AGENTS.md<br/>≤150 lines<br/>agent entrypoint"]
    CLAUDE["CLAUDE.md → AGENTS.md<br/>symlink"]
    GH[".github/<br/>issues, PR template<br/>CI workflows<br/>branch rulesets"]
    SKILLS[".claude/skills/<br/>Pocock + OS-native<br/>versioned in git"]
    CODERABBIT[".coderabbit.yaml<br/>review config"]
    DOCS["docs/<br/>product/ specs/<br/>adr/ architecture/<br/>research/ agents/"]
    MEMORY["memory/<br/>repo-graph.json<br/>dumps/"]
    SRC["src/<br/>app/ server/<br/>components/ lib/"]

    ROOT --> AGENTS
    ROOT --> CLAUDE
    ROOT --> GH
    ROOT --> SKILLS
    ROOT --> CODERABBIT
    ROOT --> DOCS
    ROOT --> MEMORY
    ROOT --> SRC

    style AGENTS fill:#e3f2fd
    style GH fill:#fff3e0
    style SKILLS fill:#f3e5f5
    style DOCS fill:#e8f5e9
    style MEMORY fill:#ede7f6
```

---

## How to use this appendix

1. **Share with team:** Send `visual-appendix.md`. GitHub renders Mermaid natively — every diagram is visible inline on github.com.
2. **Paste into tools:** Copy any diagram into [mermaid.live](https://mermaid.live) for editing and PNG/SVG export, Notion (paste as Mermaid block), Obsidian (with Mermaid plugin), or VS Code (Markdown Preview Mermaid extension).
3. **Print:** Export the lifecycle diagram (#1) as a poster for the wall. Export the daily cadence (#5) as a desk reference.
4. **Present:** Walk through diagrams #1, #3, #4, and #7 in order for a 15-minute overview of the system.

---

*Diagrams updated alongside the documentation they reference. If a doc changes, check if the visual needs updating. Mermaid syntax is intentionally simple — no custom CSS, no external dependencies.*
