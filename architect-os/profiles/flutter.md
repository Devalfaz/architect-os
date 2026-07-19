# Profile: Flutter / Dart Mobile

*Overlay for Flutter apps (mobile-first; desktop/web targets included). Applies on top of the OS core — see [README.md](README.md) for the layering rules. Record adoption in the repo's ADR-0001.*

---

## Stack

| Concern | Choice | Notes |
|---|---|---|
| Framework | Flutter (stable channel), Dart 3.x | Pin the Flutter version in `.fvmrc` (use fvm) — agents must not float channels |
| State management | Riverpod (default) | Bloc acceptable for event-heavy apps — pick ONE per repo in ADR-0001, never mix |
| Navigation | go_router | Declarative, deep-link-ready |
| Models/serialization | freezed + json_serializable via build_runner | Generated files are C30 territory |
| Backend | Supabase or Firebase | Per ADR-0001; wrap the SDK behind a repository interface either way |
| Monorepo (if multi-package) | melos | Only when packages >2 |
| Lint | very_good_analysis (or flutter_lints + repo additions) | The analyzer IS the style reviewer — humans and AI never argue style |

## Harness & model routing

Either named stack works; the **model note matters more than the harness note**: frontier models are measurably weaker at Dart/Flutter than at TypeScript — less training data, faster-moving widget APIs. Compensate with process, not hope:

- Implementation (S6): run one tier **up** from your web default on M tickets (e.g., default stack: Sonnet-class stays fine for widgets, use Opus-class for state-management refactors; open-frontier: prefer V4 Pro over Flash for non-trivial tickets).
- `grill-with-docs` at S2 is **non-optional** for any ticket touching platform channels, plugins, or recent Flutter APIs — hallucinated widget/plugin APIs are the #1 Flutter agent failure (failure mode #3).
- Golden/widget test output is the converge gate's evidence — see Test strategy.

## MCP set

| MCP | Purpose | When |
|---|---|---|
| **Dart/Flutter MCP server** (`dart mcp-server`) | Analyzer diagnostics, pub resolution, hot-reload state to the agent | Always — this is the Flutter equivalent of typecheck-grounding |
| **Figma MCP** | Design → widget implementation at S3/S6 | UI-heavy repos |
| **Supabase / Firebase MCP** | Schema, logs, advisors from the backend | Per backend choice |
| **GitHub MCP** (or `gh` CLI) | Issues/PR flow | Always |
| Device/simulator control (if available in your setup) | Screenshot-verify on device at S7 | Optional — `flutter drive`/integration_test covers most of it |

`.mcp.json` lives at repo root; keep it to what the repo actually uses (context budget — every MCP's tool schemas cost tokens).

## Constitution annex — F-rules

Cited in reviews exactly like C-rules, same severity semantics:

- **F1 — Widgets are presentation only.** 🟠 No business logic, no direct data access in widgets; logic lives in providers/blocs and services. The widget test should not need a network mock.
- **F2 — Every non-trivial widget ships a widget test; every screen ships at least one golden.** 🟠 "Renders without crashing" is not a widget test (C16 applies — assert behavior).
- **F3 — Goldens are updated deliberately, never by agents.** 🔴 `--update-goldens` is a human-only command; an agent that regenerates goldens to make a diff pass has weakened a test (C20).
- **F4 — Platform channels and plugins are wrapped.** 🟠 Direct plugin calls only inside an adapter with an interface — testability and the plugin-churn firewall.
- **F5 — pubspec pinned; no path/git dependencies without an ADR line.** 🔴 C8/C34 applied to pub.
- **F6 — Semantics are part of done.** 🟡 Interactive widgets carry Semantics labels; goldens include a text-scale 1.3 variant for screens with dense text.
- **F7 — Async gaps guarded.** 🟠 No `BuildContext` across await without `mounted` check; analyzer rule on, violations are bugs not nits.
- **F8 — Build runner output is committed and never hand-edited.** 🟠 C30 applied to freezed/json_serializable.

## CI & checks (replaces the Node ci.yml jobs)

Jobs — names become the ruleset's required checks: **Analyze** (`dart format --set-exit-if-changed .` + `flutter analyze`), **Test** (`flutter test --coverage` incl. goldens), **Build-Android** (`flutter build apk --debug`), **Build-iOS** (`flutter build ios --no-codesign`, macOS runner — make required only if you ship iOS every release). Integration tests (`integration_test/` via `flutter drive`) run on a schedule or pre-release, not per-PR (device-lab minutes are the expensive resource).

## Test strategy (what the FSD test-plan section means here)

| Layer | Tool | Converge evidence |
|---|---|---|
| Unit (providers, services, models) | `flutter test` | Runs per criterion |
| Widget (per non-trivial widget, F2) | `flutter test` + testWidgets | Behavior asserts, not render-asserts |
| Golden (per screen incl. empty/loading/error states) | golden_toolkit or alchemist | Human eyeballs golden *diffs* at S7 — the one review step AI can't do for you |
| Integration (critical journeys only) | integration_test | Pre-release gate, not per-PR |

## Review path-instructions (append to .coderabbit.yaml)

```yaml
    - path: "**/*.dart"
      instructions: |
        Flutter annex rules F1–F8 (architect-os/profiles/flutter.md) apply with
        C-rule severity semantics. Specifically flag: business logic inside
        build() methods [F1], BuildContext used across an await without a
        mounted check [F7], direct plugin/platform-channel calls outside
        adapters [F4], setState in StatefulWidgets where the repo's chosen
        state-management should own state [F1], and any change under
        **/*.g.dart or **/*.freezed.dart [F8/C30].
    - path: "**/test/**"
      instructions: |
        Widget tests must assert behavior, not "renders" [F2/C16]. Flag any
        golden file changes: goldens are human-approved only [F3].
```

## AGENTS.md seed

Commands block: `fvm flutter run` / `fvm flutter test <path>` / `fvm flutter analyze && dart format --set-exit-if-changed .` / `dart run build_runner build --delete-conflicting-outputs` (after model changes — output is committed, F8). Do-not-touch: `**/*.g.dart`, `**/*.freezed.dart`, `ios/Pods/`, `android/.gradle/`, goldens (`test/**/goldens/`). Starter gotchas: BuildContext-across-await (F7); plugin README code often targets newer plugin majors than pinned — verify against the pinned version's docs (grill-with-docs); goldens differ across host platforms — generate them in CI's platform or use alchemist's CI mode.

## Sizing note

Flutter tickets run one size bigger than they look: a "one screen" ticket is screen + provider + widget tests + golden + navigation entry — call it M, not S, and let the S1 sizing table do its job.
