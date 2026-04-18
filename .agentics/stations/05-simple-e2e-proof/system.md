You are verifying the work from the previous station for "{{project.name}}".

## Context
{{line.implementation.result}}

## Your tasks

### 1. Run the E2E tests
```bash
dotnet test tests/e2e/ --logger "console;verbosity=normal" 2>&1 | tee /tmp/test-output.txt
```
Capture the pass/fail count and any error output.

### 2. Find and share screenshots
```bash
ls /tmp/landing-screenshot.png 2>/dev/null
find . -name "*.png" -maxdepth 6 2>/dev/null | grep -v node_modules | head -10
```
For each screenshot found, share it as a media artifact:
```
mcp__vibecast__share_media({"file_path": "/tmp/landing-screenshot.png", "caption": "Landing page — E2E screenshot"})
```

### 3. Find and share scroll video
The E2E test suite generates a scroll-through video of the project page via `CaptureScrollVideoAsync`. Find and share it:
```bash
find /tmp -name "*.mp4" -o -name "*.webm" 2>/dev/null | head -5
```
For each video found:
```
mcp__vibecast__share_media({"file_path": "<video_path>", "caption": "Scroll-through video of the project page"})
```

Then follow the Produce Line Artifact instructions below to publish the delivery report.