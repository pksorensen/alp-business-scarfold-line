# Why two of these look like SaaS landing pages and the rest don't

Three generations of the scarfold-line assembly produced these seven screenshots. Comparing the `Design Team` prompt (the very first station — it sets up everything downstream) makes the regression in landing-page conventionality easy to see.

## The three generations

| Gen | Projects | Design Team prompt | New skills introduced | Result |
|---|---|---|---|---|
| **1** | `agent-inbox`, `consent-service-v3` | 1,363 ch | `/design-team` only | **Classic SaaS** — hero + 3-col features + steps + pricing + FAQ |
| **2** | `consent-service-v4` | 1,363 ch (unchanged) | `/design-md-spec`, `/design-md-lint`, `/design-tokens`, `/critique-baseline-*` added downstream | Mostly still SaaS — same hero/features/pricing pattern, lighter chrome |
| **3a** | `consent-service-v5`, `agent-inbox-v5` | 1,933 ch | **`/aesthetic-fanout`**, `/archetype`, `/mood`, `/design-md` | Editorial drift — "Standing Order" legal doc, IDE-style mock |
| **3b** | `agent-inbox-v6`, `idealab` | 2,207 ch | adds **`/design-brief`** round 0 | Full divergence — corporate letter, German-archive dossier |

## What the Gen-1 prompt actually told the agent to do

```
Use the /design-team skill to scaffold or update the design team for this project and scope.
Call the skill with the project name and a CONTEXT block so it can tailor its
questions specifically to this project rather than using generic defaults
```

That's it. One instruction: "scaffold a team for this project." The LLM defaulted to landing-page convention because nothing told it to do otherwise — and convention for SaaS is hero + features + pricing + FAQ.

## What the Gen-3b prompt added

```
This station has four phases in strict order:
strategic brief → aesthetic lock → operational intake → scaffold.
Reordering produces a generic site.
```

Then:

```
/aesthetic-fanout run
…the judge cites audience/pain/positioning when explaining its pick.
Output: .agentics/AESTHETIC.md
Summarize the lock for the project owner: era, mood, archetype, signature move
```

The prompt **explicitly frames "generic" as the failure mode**. `/aesthetic-fanout` runs N candidate aesthetics in parallel and a judge picks the most distinctive "signature move." `/archetype` + `/mood` further push toward expressive choices.

## Why this regressed the landing pages

Landing pages are a **strongly-conventional medium**. Visitors arrive with a scanning pattern (hero → what does it do → social proof → pricing → CTA). Conventionality is *information-bearing* — it lets the reader find what they need without parsing the layout.

`/aesthetic-fanout` is a divergent-thinking step optimized for **memorability**, and the most memorable archetypes its judge can reach for are typically *not* SaaS archetypes:

| Project | Aesthetic the fanout/judge picked | What that looks like as a landing page |
|---|---|---|
| `consent-service-v5` | "The Standing Order" — formal legal document, single serif, Granted! masthead | Reads like a piece of legislation, not a product page |
| `idealab` | German-archive dossier with §-glyphs, stamps, Aktenzeichen voice | Reads like a court file |
| `agent-inbox-v5` | Code-editor / IDE chrome with tab strip + `Not Read \| Look Talk Read` UI mock | Reads like a screenshot of a dev tool |
| `agent-inbox-v6` | Newsletter/letter ("Rehearse the room. Ship the idea.") | Reads like a long-form essay |

When the brief is thin (which the v6 prompt itself warns about: *"thin brief produces a thin aesthetic lock"*), the fanout still has to pick *something* distinctive — and it reaches into editorial / literary / institutional archetypes because they're strong attractors in style space.

## Other amplifying changes downstream

The pattern isn't only in the Design Team station. Across the same V3 → V5 transition:

| Station | V3 system prompt | V5 system prompt | Δ |
|---|---:|---:|---|
| Brand Deliverables | 4,380 ch | 9,893 ch | **+125%** |
| Aspire Scarfold | 8,434 ch | 9,804 ch | +16% |
| Design System | 2,635 ch | 6,687 ch | +154% |

The downstream stations also grew elaborate critique loops (`/critique-baseline-a11y`, `/critique-baseline-desktop`, `/critique-baseline-mobile`, `/critique-baseline-console`). These reinforce whatever aesthetic the Design Team locked in — so once the lock goes editorial, the rest of the line polishes the editorial direction rather than pulling it back toward convention.

## Conclusion

The first two pages look like SaaS pages because their assembly line **didn't try not to**. The newer line is engineered to maximize aesthetic distinctiveness via `/aesthetic-fanout`, which over-rotates on memorability for a medium where convention is itself the value. The fix is not to remove the brief/aesthetic exploration — it's to constrain the fanout's archetype space when the deliverable is "SaaS landing page" specifically (e.g. require the judge's pick to be reachable from a hero/features/pricing/FAQ skeleton, or have a separate convention-respecting station downstream that re-flows the chosen aesthetic into the standard SaaS scan path).
