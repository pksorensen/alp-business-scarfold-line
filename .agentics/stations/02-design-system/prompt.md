Project: {{project.name}}
{{project.description}}

Scope: {{task.title}}
{{task.description}}

The Design Team agents are pre-installed from Station 1, tailored to the locked aesthetic in `.agentics/AESTHETIC.md`.

## Your goal

Produce `DESIGN.md` at the repo root — the canonical design system in `@google/design.md` format, **aligned to `.agentics/AESTHETIC.md`** (locked direction, locked fonts, locked anti-pattern list).

## Output contract

1. `DESIGN.md` — authored by design-system-architect using `palette-retrieval` (no invention). Must pass `@google/design.md lint` with zero errors. Must include the 8 canonical sections AND 4 extended sections (Voice & Tone, Motion Principles, The Unforgettable Thing, Competitive Differentiators).
2. `design-system/tokens.json` — generated via `@google/design.md export --format dtcg`
3. `design-system/tailwind.theme.json` — generated via `@google/design.md export --format tailwind`
4. `design-system/` companion docs (index, brand, colors, typography, spacing, components, icons, motion, accessibility, the-unforgettable-thing) — each marked as generated-from-DESIGN.md.
5. Re-run the anti-generic palette validator on the committed colors as a defense-in-depth check.

Attach the lint JSON to your final report for the programmatic gate.
