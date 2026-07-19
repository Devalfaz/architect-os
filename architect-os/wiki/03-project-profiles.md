# 03 — Project Profiles: Customizing per Project Type

*"I have Flutter rules, different MCPs, mobile specifics — and web projects too. How do I customize?" Answer: you never customize the OS; you overlay it. Canonical mechanism: [profiles/README.md](../profiles/README.md).*

---

## The mental model

**`harness-matrix.md` is a map, not a config file.** It tells you what tools exist and what they cost — you read it when *choosing*. The choice itself is recorded in two places: a **profile** (per project *type*) and the repo's **ADR-0001 + AGENTS.md** (per project). Nothing in the OS core ever changes per project — that's what makes ten repos maintainable by one person.

```
architect-os core        →  never changes (lifecycle, C1–C37, rubric, rituals)
profiles/<type>.md       →  changes per TYPE   (stack, MCPs, annex rules, CI, review instructions)
repo: AGENTS.md, ADR-0001,
      .mcp.json, .coderabbit.yaml, .github/
                         →  changes per REPO   (instantiated FROM the profile)
ticket                   →  changes per TICKET (the narrow context)
```

## Where each kind of customization goes

| "I want to…" | Put it in |
|---|---|
| Add Flutter-specific coding rules | The profile's **constitution annex** (F-rules in [flutter.md](../profiles/flutter.md)) — never edit C1–C37 |
| Use different MCPs for mobile vs web | The profile's **MCP set** section → repo's `.mcp.json` / `opencode.json` |
| Different test types (widget/golden vs E2E) | The profile's **test strategy** — this redefines what the FSD test-plan and converge evidence mean for that type |
| Different CI jobs | The profile's **CI & checks** → repo's `.github/workflows/` + ruleset required-check names |
| Different AI review focus per file type | The profile's **path-instructions** block → appended to the repo's `.coderabbit.yaml` |
| A different model tier for a weak language | The profile's **harness & model routing** note (e.g., Flutter bumps S6 a tier — Dart is less well-served than TS) |
| Per-repo quirks within a type | Repo's `AGENTS.md` gotchas + ADR line-items — not the profile |
| A rule that should apply to *every* project | That's a constitution amendment — ADR-gated, per [constitution.md](../constitution.md) |

## Worked example: your Flutter app + your Next.js app

Both repos share, byte-identical: the lifecycle and gates, C1–C37, the rubric and review pipeline (converge → human → CodeRabbit → C36 second opinion), labels and Delivery project, the memory architecture, the rituals.

They differ exactly where the profiles say:

| | Flutter repo ([profile](../profiles/flutter.md)) | Next.js repo ([profile](../profiles/web-nextjs.md)) |
|---|---|---|
| Annex rules | F1–F8 (widgets presentation-only, goldens human-only, wrapped plugins…) | W1–W6 (server/client boundary, Zod at boundaries, reversible migrations…) |
| MCPs | Dart MCP, Figma, Supabase/Firebase | Supabase/Neon, Figma, Playwright, Sentry |
| CI required checks | Analyze, Test, Build-Android, Build-iOS | Typecheck, Lint, Test, Build (+E2E on M) |
| Converge evidence | widget + golden tests | Vitest + Playwright |
| S6 model note | one tier up on M tickets (Dart is under-trained) | standard routing |
| Review focus | context-across-await, plugin wrapping, golden diffs | client-component data access, `as`-casts, migration safety |

## Creating a new profile (CLI tool? API service? something else)

1. Copy the closest existing profile — keep every section heading; they're the contract.
2. Write the annex rules with a fresh prefix (`A1…` for API, `T1…` for CLI tools) — additions only; never contradict a C-rule.
3. Define what "tests" and "converge evidence" mean for the type — this is the section that actually changes agent behavior most.
4. List MCPs sparingly — every connected MCP costs context tokens in every session.
5. Register it in [profiles/README.md](../profiles/README.md)'s table, and pressure-test it by instantiating one repo and running three tickets through before trusting it.

Profiles are versioned like everything else: repos record which profile *commit* they instantiated (ADR-0001), so a profile change doesn't silently retro-apply — repos re-sync deliberately, at the weekly ritual.
