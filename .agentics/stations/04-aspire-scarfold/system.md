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

### 2. Init Aspire agent (MCP)
Run:
```
aspire agent init --non-interactive
```
This registers the Aspire MCP server with your development environment so Claude can query and interact with the running Aspire instance (resources, logs, traces) during this session. `--non-interactive` is required as this station runs headlessly.

### 3. Scaffold the Next.js app
Use the init-aspire-nextjs skill:
```
/init-aspire-nextjs
```
This will scaffold a Next.js app wired into the Aspire AppHost. Follow its instructions.

### 4. Create the landing page
Update the Next.js landing page (`src/app/page.tsx` or equivalent) to be a real marketing page for "{{project.name}}" — apply the brand colors, typography, and copy from the brand-output/landing-page.html created in the previous station. The page should be visually polished and match the design system.

### 5. Verify `aspire run` works
Run:
```
aspire run
```
Confirm the Next.js app starts and the Aspire dashboard is accessible. Use the Aspire MCP tools to verify resource health once both services are running.

### 6. Write E2E tests (screenshot + scroll video)
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

        // ── 2. Scroll-through video ───────────────────────────────────────────
        // Playwright records video per-context; file is written on context close.
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
        await Task.Delay(800); // let any animations settle

        // Slow scroll through the full page height
        var pageHeight = await videoPage.EvaluateAsync<int>("document.body.scrollHeight");
        const int steps = 30;
        for (int i = 0; i <= steps; i++)
        {
            var scrollY = (int)((double)pageHeight * i / steps);
            await videoPage.EvaluateAsync($"window.scrollTo({{ top: {scrollY}, behavior: 'smooth' }})");
            await Task.Delay(200);
        }
        await Task.Delay(800); // pause at bottom

        // Scroll back to top
        await videoPage.EvaluateAsync("window.scrollTo({ top: 0, behavior: 'smooth' })");
        await Task.Delay(600);

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

### 7. Run the E2E test
```
dotnet test tests/e2e/
```
Fix any compilation or runtime errors before proceeding. Both `/tmp/landing-screenshot.png` and `/tmp/landing-scroll.webm` must exist after the test passes.

### 8. Share both artifacts
Once the test passes, share both files:
```
mcp__vibecast__share_media({"file_path": "/tmp/landing-screenshot.png", "caption": "Landing page screenshot from E2E test"})
mcp__vibecast__share_media({"file_path": "/tmp/landing-scroll.webm", "caption": "Scroll-through video of the landing page"})
```

### 9. Commit and finish
Commit all changes (autoGit is enabled — the session will not end until the working tree is clean).

Then call:
```
mcp__vibecast__stop_broadcast({"conclusion": "success", "message": "Aspire scaffold complete. Next.js landing page created, aspire run verified, E2E test passes with screenshot and scroll video."})
```
