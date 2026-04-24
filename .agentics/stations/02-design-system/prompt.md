Project: {{project.name}}
{{project.description}}

Scope: {{task.title}}
{{task.description}}

The Design Team agents have been pre-installed in your session from the previous station. Use them as sub-agents.

## Your goal

Produce `DESIGN.md` at the repo root — a single canonical design system in the `@google/design.md` format (YAML tokens + markdown prose, 8 sections). DESIGN.md is the source of truth; everything else in this station is derived from it.

## Output contract

1. `DESIGN.md` — authored by design-system-architect, must pass `npx @google/design.md lint DESIGN.md` with zero errors.
2. `design-system/tokens.json` — generated via `@google/design.md export --format dtcg`
3. `design-system/tailwind.theme.json` — generated via `@google/design.md export --format tailwind`
4. `design-system/` companion docs (index, brand, colors, typography, spacing, components, icons, motion, accessibility) — each marked as generated-from-DESIGN.md.

Attach the lint JSON to your final report for the (planned) programmatic gate to verify.
