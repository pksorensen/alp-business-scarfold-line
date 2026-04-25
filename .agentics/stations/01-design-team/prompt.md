Project: {{project.name}}
{{project.description}}

Change/scope: {{task.title}}
{{task.description}}

## Step 1 — Pre-flight checks (do these first, before any questions)

Run `/foundry-connectivity-check` — if it passes, TTS is available downstream. If it fails or the skill is missing, call stop_broadcast with conclusion failure and message "Foundry TTS check failed — fix proxy config before running this line." Do not proceed.

Confirm `/aesthetic-fanout --help`, `/palette-retrieval --help`, `/design-md --help`, and `/design-team --help` all resolve. If any are missing, stop immediately with stop_broadcast failure and the message "Install design-team-plugin >=1.2.0 — required skills missing."

## Step 2 — Lock the aesthetic direction

Run the 3-way aesthetic fan-out:

```
/aesthetic-fanout run
```

This produces `.agentics/AESTHETIC.md` — the **locked design direction** every downstream station reads. Do NOT skip this. Without this lock, the design team scaffolds against the training-data median and produces a generic site.

After it runs, briefly summarize the lock (era / mood / archetype / signature move) for the project owner and surface any open questions the judge flagged.

## Step 3 — Scaffold the design team tailored to the lock

Call /design-team with project context AND the locked aesthetic:

```
/design-team {{project.name}}

CONTEXT:
{{project.description}}

AESTHETIC:
[full contents of .agentics/AESTHETIC.md]
```

Pass exactly that — the `CONTEXT:` and `AESTHETIC:` blocks are both required. The skill uses them to tailor each agent's persona to the specific direction (instead of generic "Principal Designer at Apple" personas that average everything back to slop).

After the skill completes and the agents are generated, create the agents and team via the API. Report back with:
- One-paragraph summary of the locked AESTHETIC
- List of agents created and their team id
- Any pending approvals
