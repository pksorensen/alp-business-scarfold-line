You are setting up a new .NET Aspire project with a Next.js landing page for "{{project.name}}".

{{tunable.design_uniqueness.injection}}

> The empirical research on this is unambiguous: when implementer agents make aesthetic decisions in a vacuum, they pull the output back to the training-data median. The slider above tunes how much of that median is welcome — but every choice still flows from the brief, never invention.

## Inputs (locked, immutable)

- **`.agentics/AESTHETIC.md`** — locked design direction (era, mood, archetype, signature move, fonts, anti-patterns)
- **`DESIGN.md`** (repo root) + **`design-system/tokens.json`** + **`design-system/tailwind.theme.json`** — design system, source of every value
- **`design-brief/SIGNATURE-MOVE.md`** — the named distinctive move with implementation notes
- **`design-brief/LAYOUT.md`** — structural blueprint
- **`design-brief/COPY.md`** — every word that appears on the site
- **`design-brief/MOTION.md`** — what moves and how
- **`brand-output/*`** — reference for visual treatment (NOT a structural blueprint to copy from)

Read all of these in full before writing any code.

## Approval gate — refuse to start if briefs aren't approved

Before scaffolding, check the four briefs in `design-brief/`. Each ends with a checkbox:

```
[ ] Owner approved.
```

If any brief still has `[ ]` (unchecked), surface this to the project owner via `AskUserQuestion` and DO NOT proceed until they confirm approval. Update the checkboxes to `[x]` after confirmation.

## Phase 1 — Scaffold

### 1. Aspire host
```
/init-aspire
```

### 2. Next.js app
```
/init-aspire-nextjs
```

### 3. Wire the design tokens

Before writing page code, import the generated theme into the Next.js app's Tailwind v4 configuration. Do **not** hand-author color/typography/spacing — they're in `design-system/tailwind.theme.json`.

Create `scripts/sync-design-tokens.mjs` that reads `design-system/tailwind.theme.json` and emits a `@theme` block to `src/styles/design-tokens.css`. Run it once now. Wire `npm run sync-tokens`. Make the sync idempotent.

Emit a TypeScript module `src/lib/design-tokens.ts` that imports `design-system/tokens.json` and re-exports typed accessors (`color(name)`, `type(role)`, `space(key)`, `radius(key)`, `component(name, prop)`).

Also copy `design-system/tokens.json` to `public/design-tokens.json` for runtime tools.

### 4. Wire the locked fonts

The fonts come from AESTHETIC.md (`{{AESTHETIC_DISPLAY_FONT}}`, `{{AESTHETIC_BODY_FONT}}`, optional `{{AESTHETIC_MONO_FONT}}`). Use `next/font` to self-host them; do not load from Google Fonts at runtime. If a font isn't on Google Fonts, document the licensing path before falling back.

## Phase 2 — Build the landing page (faithfully, from briefs)

The landing page is built **from `design-brief/`** — the four markdown files are the canonical specification.

{{#tunable brief_concreteness left mid}}
Station 3 also produced `brand-output/landing-page.html` as a **visual reference** (not a structural blueprint). Read it for ornament density, section rhythm, and copy texture. Match its level of polish in your implementation, but do not copy its DOM structure verbatim — the briefs are what the page is built from.
{{/tunable}}
{{#tunable brief_concreteness right}}
There is no `brand-output/landing-page.html` reference this run. Build the page from briefs alone.
{{/tunable}}

For each section in `LAYOUT.md`:

1. Implement the structure as described — same blocks, same order, same content density.
2. Write the copy from `COPY.md` verbatim. No lorem ipsum, no embellishment, no "improving" the headline.
3. Apply tokens via Tailwind utility classes (`bg-primary-9`, `text-foreground`, `font-display`, `rounded-lg`). **Never raw hex.** **Never raw font-family.**
4. Implement motion ONLY where `MOTION.md` specifies it, with the exact easing/duration/elements specified.
5. Implement the **signature move** from `SIGNATURE-MOVE.md` faithfully — this is the brand's distinctive claim; if your implementation hedges or simplifies it, the page becomes generic.

**Token-integrity rule:** any hex code, raw font-family, or raw spacing number that appears in your JSX/CSS but is not in `tokens.json` is a bug. Add a build-time check (a small `scripts/check-token-integrity.mjs` that scans `src/**/*.{tsx,css}` for hex literals and reports unknowns).

## Phase 3 — Verify the build runs

```
aspire run
```

Confirm Next.js starts and the Aspire dashboard is accessible. Use the Aspire MCP tools to verify resource health.

## Phase 4 — Adversarial critique panel (4 personas, fresh contexts, hard-cap 3 iterations)

> Single-agent self-critique restates the original error in 40%+ of trials (MAR research). The panel below uses **separate sub-agent contexts** with named-rubric personas. Each persona produces *diffs against the current artifact*, not prose. The orchestrator merges non-conflicting diffs; conflicts surface to the project owner.

### Take a baseline screenshot

Use the Playwright MCP server (or our existing Playwright harness) to navigate to the landing page and capture:

- A full-page screenshot at 1280×900 (`/tmp/critique-baseline-desktop.png`)
- A full-page screenshot at 375×812 mobile (`/tmp/critique-baseline-mobile.png`)
- The accessibility tree (`/tmp/critique-baseline-a11y.json`)
- A console-error capture (`/tmp/critique-baseline-console.txt`)

If Playwright MCP is not yet configured for this project, fall back to the existing Aspire E2E test pattern (Station 5 has the template) and produce the artifacts the same way.

### The 4 critic personas (run in parallel via the Task tool)

Each is a separate sub-agent invocation with a **fresh context** and a named rubric. Each receives: the screenshots + a11y tree + console errors, AESTHETIC.md, DESIGN.md, and the four briefs. Each produces a `critique-<persona>.md` with **diffs**, not prose.

#### Critic A — Art Director
Persona: an art director from the AESTHETIC's specific lineage (e.g., "from an independent print magazine in the editorial / literary tradition"). Rubric: distinctiveness, restraint, signature-move execution, font discipline, layout rhythm. Anti-patterns from AESTHETIC.md are this critic's checklist. Diffs name the violation and propose specific replacements.

#### Critic B — Brand Strategist
Persona: a brand strategist whose only loyalty is to the brand brief. Rubric: does the page communicate the brand's positioning? Does the Voice & Tone read as the brand, or as generic SaaS? Does the signature move land within 3 seconds of page load? Diffs target copy and information hierarchy.

#### Critic C — Conversion / Product
Persona: a conversion lead at a B2B SaaS — pragmatic, ruthless about clarity. Rubric: is the value proposition unambiguous within the hero? Are CTAs visible and actionable? Are friction points obvious? Diffs target the hero, CTAs, and the trust/proof section.

#### Critic D — Automated Lint
Not an LLM — a deterministic check:
- `npx -y @google/design.md lint DESIGN.md --format json` → must be errors:0
- `node scripts/check-token-integrity.mjs` → must be 0 violations
- `npx -y axe-cli` (or `@axe-core/cli`) on the rendered page → 0 violations
- Lighthouse `--only-categories=accessibility,performance` (or equivalent) → a11y ≥ 95, performance ≥ 85

Diffs from this critic are *required fixes*, not suggestions.

### Orchestrator merges and applies diffs

Read all 4 critique files. Apply non-conflicting diffs (zero new decisions — diffs are mechanical replacements). Surface conflicts to the project owner. Then take a fresh screenshot and repeat **up to 3 total iterations**, halting early if the rubric delta is small (no critic finds new violations).

If the critic panel hits 3 iterations and Critic A or B is still flagging genericness, that means the implementation is silently averaging the briefs. Escalate to the project owner — the fix is in the briefs, not in another implementation pass.

## Phase 5 — Write E2E tests

Create a test project at `tests/e2e/` using:
- `Aspire.Hosting.Testing` (DistributedApplicationTestingBuilder)
- `Microsoft.Playwright.NUnit` for browser automation

Produce two artifacts on every test run:
1. `/tmp/landing-screenshot.png` — full-page screenshot
2. `/tmp/landing-scroll.webm` — slow scroll-through video pausing at each major section

(Use the existing E2E test pattern from earlier versions of this station — the test code is unchanged, only the page-under-test now reflects the brief-driven implementation.)

## Phase 6 — Run E2E and share artifacts

```
dotnet test tests/e2e/
```

Both `/tmp/landing-screenshot.png` and `/tmp/landing-scroll.webm` must exist after the test passes.

```
mcp__vibecast__share_media({"file_path": "/tmp/landing-screenshot.png", "caption": "Landing page screenshot"})
mcp__vibecast__share_media({"file_path": "/tmp/landing-scroll.webm", "caption": "Scroll-through video"})
```

Also share the critique artifacts — the project owner should see what the panel found:
```
mcp__vibecast__share_media({"file_path": "/tmp/critique-baseline-desktop.png", "caption": "Baseline screenshot before critique"})
```

## Phase 7 — Commit and finish

Commit all changes (autoGit is enabled). Then:

```
mcp__vibecast__stop_broadcast({"conclusion": "success", "message": "Aspire scaffold complete. Landing page built faithfully from briefs, critique panel ran N iterations, E2E test passes with screenshot + scroll video."})
```

## Quality bar (station exit)

- `aspire run` works and Next.js serves the landing page
- Token integrity: zero raw hex, zero raw font-family, zero raw spacing in source code
- All 4 brief files reflected: signature move present, layout matches LAYOUT.md, copy matches COPY.md verbatim, motion matches MOTION.md
- Critic D (automated lint) passes: design-md lint 0 errors, token integrity 0 violations, axe 0 violations, Lighthouse a11y ≥ 95
- E2E test passes; screenshot + video produced
{{#tunable brief_concreteness right}}
- No "ports" of `brand-output/landing-page.html` (it doesn't exist this version)
{{/tunable}}
{{#tunable brief_concreteness left mid}}
- The implementation matches the polish level of `brand-output/landing-page.html` (the reference) without copying its DOM structure verbatim
{{/tunable}}
