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
For each screenshot found:
```
mcp__vibecast__share_image({"image_path": "/tmp/landing-screenshot.png", "caption": "Landing page — E2E screenshot"})
```

Then follow the Produce Line Artifact instructions below to publish the delivery report.