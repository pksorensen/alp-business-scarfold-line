Project: {{project.name}}
{{project.description}}

Scope: {{task.title}}
{{task.description}}

This station is **art direction & brief**, not implementation. The Design Team agents are pre-installed.

## Inputs (must exist before you start)
- `.agentics/AESTHETIC.md` — locked direction
- `DESIGN.md` (repo root) — canonical design system
- `design-system/tokens.json` + `design-system/tailwind.theme.json` — exports
- `design-system/the-unforgettable-thing.md` — signature-move reminder

## Outputs (this station produces)

### Briefs — `design-brief/` (new dir)
Markdown specifications Station 4 reads to build the Next.js site. **Not finished HTML.**
- `SIGNATURE-MOVE.md` — the named move + concept sketch + implementation notes
- `LAYOUT.md` — IA + section-by-section structural blueprint
- `COPY.md` — every word on the site, no lorem ipsum, length-aware
- `MOTION.md` — the one orchestrated moment + micro-interactions, ≤ 80 lines budget

Each brief ends with `[ ] Owner approved.` — Station 4 gates on this.

### Brand sheets — `brand-output/` (immutable HTML)
Public-facing brand collateral. Self-contained HTML, tokens injected from `tokens.json`. No raw hex.
- `brand-identity.html` — investor-ready brand sheet (brand-identity-creator)
- `design-system.html` — developer reference (design-system-architect + ui-ux-pattern-master)
- `ui-showcase.html` — live component library (ui-ux-pattern-master + design-to-code-translator)
- `email-template.html` — production HTML email (marketing-asset-factory)
- `social-kit.html` — OG / Twitter / LinkedIn / avatar frames (figma-autolayout-expert)

**No `landing-page.html` this station.** That's Station 4's job, built from the briefs above.

Report briefs + brand sheets, drift audit (hex/font not in tokens.json), and approval-gate status.
