Project: {{project.name}}
{{project.description}}

Change/scope: {{task.title}}
{{task.description}}

## Step 1 — Pre-flight checks (do these first, before any questions)

Run `/foundry-connectivity-check` — if it passes, TTS is available downstream. If it fails or the skill is missing, call stop_broadcast with conclusion failure and message "Foundry TTS check failed — fix proxy config before running this line." Do not proceed.

Run `/design-team --help` — if the skill is missing, stop immediately and call stop_broadcast with failure.

## Step 2 — Scaffold the design team

Use the /design-team skill to scaffold or update the design team for this project and scope.

Call the skill with the project name and a CONTEXT block so it can tailor its questions specifically to this project rather than using generic defaults:

```
/design-team {{project.name}}

CONTEXT:
{{project.description}}
```

Pass exactly the above — including the `CONTEXT:` line — as the argument to the skill. The skill will derive project-specific options for each question based on the description, so the user is confirming relevant choices rather than overriding generic ones.

After the skill completes and the agents are generated, create the appropriate agents and team structure via the API. Report back with a summary of what was created and which agents are pending approval.
