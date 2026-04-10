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

### 6. Write a simple E2E test
Create a test project at `tests/e2e/` using:
- `Aspire.Hosting.Testing` (DistributedApplicationTestingBuilder)
- `Microsoft.Playwright.NUnit` for browser automation

The test should:
1. Start the Aspire host via `DistributedApplicationTestingBuilder`
2. Navigate to the Next.js landing page URL (get URL from the Aspire resource)
3. Assert HTTP 200
4. Take a full-page screenshot and save to `/tmp/landing-screenshot.png`

Example test structure:
```csharp
[Test]
public async Task LandingPage_ReturnsOkAndScreenshot()
{
    var appHost = await DistributedApplicationTestingBuilder
        .CreateAsync<Projects.AppHost>();
    await using var app = await appHost.BuildAsync();
    await app.StartAsync();

    var httpClient = app.CreateHttpClient("webapp");
    var resp = await httpClient.GetAsync("/");
    Assert.That((int)resp.StatusCode, Is.EqualTo(200));

    using var playwright = await Playwright.CreateAsync();
    var browser = await playwright.Chromium.LaunchAsync();
    var page = await browser.NewPageAsync();
    await page.GotoAsync(httpClient.BaseAddress!.ToString());
    await page.ScreenshotAsync(new() { Path = "/tmp/landing-screenshot.png", FullPage = true });
    await browser.CloseAsync();
}
```

### 7. Run the E2E test
```
dotnet test tests/e2e/
```
Fix any compilation or runtime errors before proceeding.

### 8. Share the screenshot
Once the test passes and `/tmp/landing-screenshot.png` exists, share it using the vibecast MCP tool:
```
mcp__vibecast__share_image({"image_path": "/tmp/landing-screenshot.png", "caption": "Landing page screenshot from Aspire E2E test"})
```

### 9. Commit and finish
Commit all changes (autoGit is enabled — the session will not end until the working tree is clean).

Then call:
```
mcp__vibecast__stop_broadcast({"conclusion": "success", "message": "Aspire scaffold complete. Next.js landing page created, aspire run verified, E2E test passes with screenshot."})
```