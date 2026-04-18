You are verifying the work from the previous station for "{{project.name}}".

## Context
{{line.implementation.result}}

## Your tasks

### 1. Run the E2E tests
```bash
dotnet test tests/e2e/ --logger "console;verbosity=normal" 2>&1 | tee /tmp/test-output.txt
```
Capture the pass/fail count and any error output.

### 2. Find and share the screenshot
```bash
ls /tmp/landing-screenshot.png 2>/dev/null
find . -name "*.png" -not -path "*/node_modules/*" -maxdepth 6 2>/dev/null | head -10
```
Share the screenshot:
```
mcp__vibecast__share_media({"file_path": "/tmp/landing-screenshot.png", "caption": "Landing page — E2E screenshot"})
```

### 3. Find and share the scroll video
The E2E test records a slow scroll-through video of the landing page and saves it to `/tmp/landing-scroll.webm`. Find and share it:
```bash
ls /tmp/landing-scroll.webm 2>/dev/null
find /tmp -name "*.webm" -o -name "*.mp4" 2>/dev/null | head -5
```
Share the video:
```
mcp__vibecast__share_media({"file_path": "/tmp/landing-scroll.webm", "caption": "Scroll-through video of the landing page"})
```

If neither path exists, check `/tmp/playwright-video/` for any `.webm` files left by Playwright and share the most recent one.

Then follow the Produce Line Artifact instructions below to publish the delivery report.
