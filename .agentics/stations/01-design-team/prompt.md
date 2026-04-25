Project: {{project.name}}
{{project.description}}

Change/scope: {{task.title}}
{{task.description}}

This station has **four phases in strict order**: strategic brief → aesthetic lock → operational intake → scaffold. Reordering produces a generic site.

## Step 1 — Pre-flight checks

Run `/foundry-connectivity-check` — fail with stop_broadcast if it doesn't pass.

Confirm `/design-brief --help`, `/aesthetic-fanout --help`, `/palette-retrieval --help`, `/design-md --help`, `/design-team --help` all resolve. Stop with stop_broadcast failure if any are missing — message: "Install design-team-plugin >=1.2.0 — required skills missing: <list>."

## Step 2 — Strategic brief (BEFORE aesthetic)

```
/design-brief gather
```

Walks the owner through 3 rounds (Identity, Strategy, Audience & Product). Output: `.agentics/BRIEF.md`.

Then validate:
```
/design-brief validate
```

If any missing sections or generic-answer warnings, loop back via `/design-brief update --section <name>` until the brief is specific. **Do not proceed until the brief is clean** — thin brief produces a thin aesthetic lock.

## Step 3 — Aesthetic fan-out (consumes BRIEF)

```
/aesthetic-fanout run
```

Refuses to run without `.agentics/BRIEF.md`. Workers and judge read the brief in full; the judge cites audience/pain/positioning when explaining its pick. Output: `.agentics/AESTHETIC.md`.

Summarize the lock for the project owner: era, mood, archetype, signature move, and how it ties to the brief.

## Step 4 — Scaffold tailored to BRIEF + AESTHETIC

```
/design-team {{project.name}}

CONTEXT:
{{project.description}}

BRIEF:
[full contents of .agentics/BRIEF.md]

AESTHETIC:
[full contents of .agentics/AESTHETIC.md]
```

The skill detects all three blocks and skips questions answered in BRIEF (Round 1.2, Round 2, Round 3.1+3.2+3.4, Round 4) and AESTHETIC (Round 1.3+1.4). Asks only the operational residual: tech stack, campaign details, presentation specs.

## Step 5 — Register agents and team via API

Standard registration. Agents created with status 'hiring'.

## Output

Report: BRIEF summary, AESTHETIC summary (with judge's brief-citation), list of agents + team id, any unfilled placeholders (must be zero).
