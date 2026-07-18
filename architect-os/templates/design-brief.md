# Design Brief Template

*Decide what it looks and feels like before code exists. For UI features only. Headless/API-only work skips this.*

---

```markdown
# Design Brief: [FEATURE NAME]

<!-- status: draft | approved -->
<!-- date: YYYY-MM-DD -->
<!-- linked-fsd: ../../specs/<slug>/fsd.md -->

## Screens & states inventory

*Every screen and state from the FSD's "States & screens" table.*

| Screen | States | Priority |
|---|---|---|
| Settings > Export section | Idle, loading, success, error, empty | P0 |
| Toast notifications | Success, error, info | P0 |

## Design tokens

| Token | Value |
|---|---|
| Primary color | `hsl(var(--primary))` — from existing theme |
| Spacing | shadcn/ui defaults |
| Typography | Inter, 14px body, 16px headings |
| Border radius | `rounded-md` (shadcn default) |

## Component mapping (shadcn/ui)

| Screen element | shadcn component | Notes |
|---|---|---|
| Export button | `<Button>` | Primary variant, `loading` state for export in progress |
| Error message | `<Alert variant="destructive">` | |
| Loading state | `<Button loading>` or `<Skeleton>` for table preview | |
| Success toast | `<Toast>` | Auto-dismiss after 3 seconds |

## Accessibility

- All interactive elements: keyboard-navigable, focus-visible styles, aria-labels
- Loading states: `aria-busy="true"` on the export button during generation
- Error states: `role="alert"` on error messages, announced by screen readers
- Color: all states distinguishable without color (icons + text, not color alone)
- Target: WCAG AA

## Empty / loading / error states

### Empty state
[Description: what the user sees when there's no data. "A centered message: 'No data to export yet.' Export button is disabled."]

### Loading state
[Description: what happens during async operations. "Button text changes to 'Exporting...' with a spinner. Button is disabled."]

### Error state
[Description: what happens when something goes wrong. "A destructive Alert appears below the button: 'Export failed. Please try again.' Error is logged but not exposed to the user."]

### Edge cases
- Very long export (near the cap): estimated time display? (Decision: no — v1 is simple. Timeout is a soft cap at 30 seconds per NFR.)
- Export interrupted (browser close): no recovery needed — stateless GET

## Figma links (if applicable)

- [Link to Figma file]

## Prototype (if built)

- `prototypes/<slug>/` — throwaway prototype from S3 exploration
```

---

*Save as `docs/product/<slug>/design-brief.md`. The gate: every screen and state in the FSD has a design decision. Empty/loading/error states are specified — these are what agents invent badly when left unspecified.*
