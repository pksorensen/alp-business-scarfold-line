You are setting up a new .NET Aspire project with a Next.js landing page for "{{project.name}}".

## Brand context
The brand-output/ and design-system/ directories contain the design outputs from the previous station. Use them as reference for the landing page.

## Your tasks — execute in order

### 1. Scaffold the Aspire host
Use the init-aspire skill:
```
/init-aspire
```
This will scaffold a .NET Aspire AppHost project. Follow its instructions.

### 2. Scaffold the Next.js app
Use the init-aspire-nextjs skill:
```
/init-aspire-nextjs
```
This will scaffold a Next.js app wired into the Aspire AppHost. Follow its instructions.

### 3. Create the landing page
Update the Next.js landing page (`src/app/page.tsx` or equivalent) to be a real marketing page for "{{project.name}}" — apply the brand colors, typography, and copy from the brand-output/landing-page.html created in the previous station. The page should be visually polished and match the design system.

### 4. Verify `aspire run` works
Run:
```
aspire run
```
Confirm the Next.js app starts and the Aspire dashboard is accessible. Use the Aspire MCP tools to verify resource health once both services are running.

### 5. Write E2E tests (screenshot + scroll video)
Create a test project at `tests/e2e/` using:
- `Aspire.Hosting.Testing` (DistributedApplicationTestingBuilder)
- `Microsoft.Playwright.NUnit` for browser automation

The test must produce **two artifacts**:
1. `/tmp/landing-screenshot.png` — full-page screenshot
2. `/tmp/landing-scroll.webm` — slow scroll-through video of the full page

Use this exact test structure (adapt resource name `"webapp"` to match your Aspire AppHost resource name):

```csharp
using Microsoft.Playwright;
using Microsoft.Playwright.NUnit;

[TestFixture]
public class LandingPageTests : PlaywrightTest
{
    [Test]
    public async Task LandingPage_ScreenshotAndScrollVideo()
    {
        // Start Aspire host
        var appHost = await DistributedApplicationTestingBuilder
            .CreateAsync<Projects.AppHost>();
        await using var app = await appHost.BuildAsync();
        await app.StartAsync();

        var httpClient = app.CreateHttpClient("webapp");
        var baseUrl = httpClient.BaseAddress!.ToString();

        // Verify HTTP 200
        var resp = await httpClient.GetAsync("/");
        Assert.That((int)resp.StatusCode, Is.EqualTo(200));

        using var playwright = await Playwright.CreateAsync();
        var browser = await playwright.Chromium.LaunchAsync(new() { Headless = true });

        // ── 1. Full-page screenshot ───────────────────────────────────────────
        {
            var page = await browser.NewPageAsync(new()
            {
                ViewportSize = new ViewportSize { Width = 1280, Height = 900 }
            });
            await page.GotoAsync(baseUrl);
            await page.WaitForLoadStateAsync(LoadState.NetworkIdle);
            await page.ScreenshotAsync(new() { Path = "/tmp/landing-screenshot.png", FullPage = true });
            await page.CloseAsync();
        }

        // ── 2. Presentation scroll video ─────────────────────────────────────
        // Goal: a polished, presenter-quality walkthrough — slow smooth scroll,
        // pausing at each major section so viewers can read the content.
        // Duration is content-driven, not fixed.
        var videoDir = "/tmp/playwright-video/";
        Directory.CreateDirectory(videoDir);

        var context = await browser.NewContextAsync(new()
        {
            ViewportSize = new ViewportSize { Width = 1280, Height = 900 },
            RecordVideoDir = videoDir,
            RecordVideoSize = new RecordVideoSize { Width = 1280, Height = 900 }
        });
        var videoPage = await context.NewPageAsync();
        await videoPage.GotoAsync(baseUrl);
        await videoPage.WaitForLoadStateAsync(LoadState.NetworkIdle);
        await Task.Delay(1500); // pause at top so hero is readable

        // Collect section anchor points: top of each major section/landmark element.
        // Fall back to evenly-spaced stops if no sections are found.
        var sectionTops = await videoPage.EvaluateAsync<int[]>(@"
            (() => {
                // Target semantic section breaks — sections, articles, major divs with headings
                const candidates = [
                    ...document.querySelectorAll('section, article, [id], h1, h2, h3')
                ];
                const viewH = window.innerHeight;
                const pageH = document.body.scrollHeight;
                const seen = new Set();
                const tops = [];
                for (const el of candidates) {
                    const rect = el.getBoundingClientRect();
                    const absTop = rect.top + window.scrollY;
                    // Skip elements in the top viewport (already visible at start) and near-duplicates
                    if (absTop < viewH * 0.8) continue;
                    const bucket = Math.round(absTop / (viewH * 0.5));
                    if (seen.has(bucket)) continue;
                    seen.add(bucket);
                    tops.push(Math.min(Math.round(absTop - viewH * 0.05), pageH - viewH));
                }
                // Always end at the bottom
                tops.push(pageH - viewH);
                return tops.filter(t => t > 0);
            })()
        ");

        // If no sections found, fall back to 6 evenly-spaced stops
        var pageHeight = await videoPage.EvaluateAsync<int>("document.body.scrollHeight");
        var viewportHeight = 900;
        if (sectionTops == null || sectionTops.Length == 0)
        {
            int stops = 6;
            sectionTops = Enumerable.Range(1, stops)
                .Select(i => Math.Min((int)((double)(pageHeight - viewportHeight) * i / stops), pageHeight - viewportHeight))
                .ToArray();
        }

        // Smoothly scroll to each section stop, pausing to let viewers read
        int currentY = 0;
        foreach (var targetY in sectionTops)
        {
            if (targetY <= currentY) continue;
            int distance = targetY - currentY;
            // Micro-step smooth scroll: ~4px per frame at ~30fps feels cinematic
            int microSteps = Math.Max(20, distance / 4);
            for (int s = 1; s <= microSteps; s++)
            {
                var y = currentY + (int)((double)distance * s / microSteps);
                await videoPage.EvaluateAsync($"window.scrollTo({{ top: {y}, behavior: 'instant' }})");
                await Task.Delay(33); // ~30fps
            }
            currentY = targetY;
            // Pause at each section so content is readable (1.5–2.5s depending on distance)
            int pauseMs = distance > viewportHeight ? 2500 : 1500;
            await Task.Delay(pauseMs);
        }
        await Task.Delay(1500); // linger at bottom

        // Scroll back to top
        await videoPage.EvaluateAsync("window.scrollTo({ top: 0, behavior: 'smooth' })");
        await Task.Delay(1000);

        // Closing context flushes the video file
        var videoPath = await videoPage.Video!.PathAsync();
        await context.CloseAsync();

        // Copy to a stable path for station 05 to find
        File.Copy(videoPath, "/tmp/landing-scroll.webm", overwrite: true);

        await browser.CloseAsync();

        Assert.That(File.Exists("/tmp/landing-screenshot.png"), Is.True, "Screenshot missing");
        Assert.That(File.Exists("/tmp/landing-scroll.webm"), Is.True, "Scroll video missing");
    }
}
```

### 6. Run the E2E test
```
dotnet test tests/e2e/
```
Fix any compilation or runtime errors before proceeding. Both `/tmp/landing-screenshot.png` and `/tmp/landing-scroll.webm` must exist after the test passes.

### 7. Share both artifacts
Once the test passes, share both files:
```
mcp__vibecast__share_media({"file_path": "/tmp/landing-screenshot.png", "caption": "Landing page screenshot from E2E test"})
mcp__vibecast__share_media({"file_path": "/tmp/landing-scroll.webm", "caption": "Scroll-through video of the landing page"})
```

### 8. Commit and finish
Commit all changes (autoGit is enabled — the session will not end until the working tree is clean).

Then call:
```
mcp__vibecast__stop_broadcast({"conclusion": "success", "message": "Aspire scaffold complete. Next.js landing page created, aspire run verified, E2E test passes with screenshot and scroll video."})
```
