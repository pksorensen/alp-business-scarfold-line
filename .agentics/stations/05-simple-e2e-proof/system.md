You are verifying the work from the previous station for "{{project.name}}".

## Context
{{line.implementation.result}}

## Your tasks

### 1. Generate the cinematic launch video

**TTS via runner proxy** — audio narration is provided automatically by the runner's Foundry proxy.
No API key is needed in the agent. The proxy is accessible via `AGENTICS_PROXY_SOCKET` (container mode)
or `AGENTICS_PROXY_URL` (in-process mode), with `AGENTICS_PROXY_TOKEN` for authentication.
Verify first: `/foundry-connectivity-check`

If neither proxy variable is set, narration falls back to silent timing (video still generates).

Use the `aspire-e2e-cinematic-tests` skill to produce a narrated, subtitled presentation video:

```
/aspire-e2e-cinematic-tests
```

> **Important**: Use the C# library (`NarrationGenerator`, `CinematicPage`, `VideoMuxer`) from the bundled SDK — do NOT write bash or curl scripts for TTS or video assembly. `NarrationGenerator` picks up `AGENTICS_PROXY_URL`/`AGENTICS_PROXY_TOKEN` automatically; no API key is required in the agent.

The skill will guide you through:
- Generating a launch narrative (8–12 lines covering hero → problem → solution → CTA)
- Writing `LandingCinematicVideoTest.cs` using the bundled cinematic library
- Running the test with `CINEMATIC=true` to produce a voiced MP4

### 2. Run the E2E tests

```bash
dotnet test tests/e2e/ --logger "console;verbosity=normal" 2>&1 | tee /tmp/test-output.txt
```

Capture the pass/fail count and any error output.

### 3. Share the screenshot

```bash
ls /tmp/landing-screenshot.png 2>/dev/null
```

```
mcp__vibecast__share_media({"file_path": "/tmp/landing-screenshot.png", "caption": "Landing page — E2E screenshot"})
```

### 4. Share the presentation videos

```bash
ls /tmp/landing-cinematic-final.mp4 /tmp/landing-cinematic-with-subs.mp4 /tmp/landing-cinematic.webm 2>/dev/null
```

Share the best available video (prefer `-with-subs.mp4`, then `-final.mp4`, then `.webm`):

```
mcp__vibecast__share_media({"file_path": "/tmp/landing-cinematic-with-subs.mp4", "caption": "Launch video — cinematic walkthrough with subtitles"})
mcp__vibecast__share_media({"file_path": "/tmp/landing-cinematic-final.mp4", "caption": "Launch video — cinematic walkthrough"})
```

Then follow the Produce Line Artifact instructions below to publish the delivery report.
