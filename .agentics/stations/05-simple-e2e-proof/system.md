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

### 3. Generate a scroll preview video

Create a short scrolling video of the landing page so the design output can be shared and reviewed later.

First ensure Playwright is available:
```bash
npm install playwright 2>/dev/null | tail -3
npx playwright install chromium --with-deps 2>/dev/null | tail -5
```

Write `/tmp/scroll-video.mjs`:
```javascript
import { chromium } from 'playwright';
import { mkdirSync, readdirSync, copyFileSync } from 'fs';
import { execSync } from 'child_process';

const APP_URL = process.env.APP_URL || 'http://localhost:3000';
const TMP = '/tmp/scroll-vid-tmp';
mkdirSync(TMP, { recursive: true });

const browser = await chromium.launch();
const ctx = await browser.newContext({
    recordVideo: { dir: TMP, size: { width: 1280, height: 720 } },
    viewport: { width: 1280, height: 720 },
});
const page = await ctx.newPage();
await page.goto(APP_URL, { waitUntil: 'networkidle' });
await new Promise(r => setTimeout(r, 2000));  // let lazy content render
await page.evaluate('window.scrollTo(0, 0)');
await new Promise(r => setTimeout(r, 500));

const scrollable = await page.evaluate(
    'document.documentElement.scrollHeight - window.innerHeight'
);

// Smooth scroll with ease-in-out over 12 seconds
const STEPS = 240;
const DURATION_MS = 12_000;
for (let i = 1; i <= STEPS; i++) {
    const t = i / STEPS;
    const eased = t * t * (3 - 2 * t);  // smoothstep
    await page.evaluate(`window.scrollTo(0, ${Math.round(scrollable * eased)})`);
    await new Promise(r => setTimeout(r, DURATION_MS / STEPS));
}
await new Promise(r => setTimeout(r, 3000));  // pause at bottom

await ctx.close();
await browser.close();

const webm = readdirSync(TMP).find(f => f.endsWith('.webm'));
if (!webm) { console.error('No video file produced'); process.exit(1); }

const src = `${TMP}/${webm}`;
try {
    execSync(
        `ffmpeg -y -i "${src}" -c:v libx264 -preset fast -crf 23 -movflags +faststart -an /tmp/scroll-preview.mp4`,
        { stdio: 'inherit' }
    );
    console.log('scroll-preview.mp4 ready');
} catch {
    copyFileSync(src, '/tmp/scroll-preview.webm');
    console.log('scroll-preview.webm ready (ffmpeg not available)');
}
```

Run it (pass the actual URL the app is serving on):
```bash
APP_URL=http://localhost:3000 node /tmp/scroll-video.mjs
```

Copy the output to the work tree so it is preserved as a deliverable:
```bash
cp /tmp/scroll-preview.mp4 ./scroll-preview.mp4 2>/dev/null || \
cp /tmp/scroll-preview.webm ./scroll-preview.webm 2>/dev/null || true
```

Share the video via the image tool (supported for short clips):
```
mcp__vibecast__share_image({"image_path": "/tmp/scroll-preview.mp4", "caption": "Landing page scroll preview — {{project.name}}"})
```

Then follow the Produce Line Artifact instructions below to publish the delivery report.
Include the scroll video path (`scroll-preview.mp4` or `.webm`) in the artifact result
so downstream processes and the demo page know where to find it.
