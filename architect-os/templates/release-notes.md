# Release Notes Template

*Generated at milestone close or for significant releases. Agents draft; you edit and publish.*

---

```markdown
# Release Notes — [VERSION / MILESTONE]

<!-- date: YYYY-MM-DD -->
<!-- milestone: [GitHub milestone URL] -->

## Overview

[One paragraph: what shipped in this release? Who does it affect?]

## New features

- **[Feature name]:** [One sentence description + link to docs/spec if applicable] (#123)
- ...

## Bug fixes

- **[Bug description]:** [What was wrong, what's the fix] (#124)
- ...

## Improvements

- **[Improvement description]** (#125)
- ...

## Breaking changes

- **[Breaking change]:** [What changed, migration path] — see [migration guide](link) (#126)

## Deprecations

- **[Deprecated feature]:** [What's deprecated, what replaces it, removal timeline] (#127)

## Known issues

- **[Issue]:** [Description, workaround if any] (#128)

## Contributors

- [@handle] — [contributions]
- Agent (Claude Code / Codex) — implementation of #123, #125, #127

## Deployment notes

- [Any special deployment steps? Migrations to run? Feature flags to toggle?]
```

---

*Save as part of the GitHub Release or in `docs/releases/`. Drafted by agent at milestone close; reviewed and edited by you. The audience is users and stakeholders — not developers. Technical details go in ADRs or the changelog.*
