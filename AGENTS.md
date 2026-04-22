# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Project Overview

MyDevTools is a privacy-first developer utility suite built with Blazor WebAssembly. All processing happens client-side in the browser — no data is ever sent to a server. Deployed to Azure Static Web Apps at https://blazor.mydevtools.org.

## Commands

```bash
# Build
dotnet build MyDevTools.sln

# Run with hot reload
dotnet watch run --project MyDevTools.sln

# Publish
dotnet publish MyDevTools.sln
```

Local URLs when running:
- http://localhost:5295
- https://localhost:7135

There are no test projects in this solution.

## Architecture

**Stack**: .NET 8, Blazor WebAssembly, Bootstrap 5. No backend — pure client-side WASM.

**Entry point chain**: `wwwroot/index.html` → Blazor JS runtime → `Program.cs` (DI setup) → `App.razor` (router) → `MainLayout.razor` + page component.

**Layout hierarchy**:
```
index.html
└── App.razor (router)
    └── MainLayout.razor
        ├── NavMenu.razor (sidebar)
        └── @Body → Tool page (e.g., GuidGenerator.razor)
                     └── OutputAndCopyToClipboard.razor (shared result display)
```

**Routing**: Blazor file-based routing — component filename = URL path. `Home.razor` → `/`, `GuidGenerator.razor` → `/GuidGenerator`.

**Page base class**: Most tool pages inherit from `BasePage.razor` (`Components/Pages/Base/`), which provides `UpdateMetaTagsAsync(description, keywords)` for SEO meta tag updates via JS interop into `wwwroot/js/Seo.js`.

**Shared components** (in `Components/Pages/Components/`):
- `OutputAndCopyToClipboard.razor` — renders tool output with a copy-to-clipboard button (3s confirmation via `ClipboardService`)
- `DiffOutput.razor` — renders color-coded diff results

**Services** (registered as scoped in `Program.cs`):
- `IClipboardService` / `ClipboardService` — clipboard access via JS interop
- `SeoService` — generates schema.org JSON-LD structured data

**JSON Formatter exception**: The JSON tool loads `wwwroot/iframepages/jsonformatter.html` inside an iframe and uses `wwwroot/js/JsonEditor.js` (a large third-party JS editor library) rather than the standard Blazor component pattern.

## Adding a New Tool

1. Create `Components/Pages/YourTool.razor` — inherit from `BasePage`, add `@page "/yourtool"` directive
2. Call `UpdateMetaTagsAsync(description, keywords)` in `OnAfterRenderAsync`
3. Use `OutputAndCopyToClipboard` for displaying results
4. Add a nav entry in `Components/Layout/NavMenu.razor` with a Bootstrap icon (`bi bi-*`)
5. Add to `wwwroot/sitemap.xml` and update SEO content on `Home.razor` if needed

## Key Dependencies

| Package | Purpose |
|---|---|
| DiffPlex | Text diff comparison |
| System.IdentityModel.Tokens.Jwt | JWT decoding |
| BasicSQLFormatter | SQL formatting |
| Net.Codecrete.QrCodeGenerator | QR code generation |

## Deployment

CI/CD via GitHub Actions (`.github/workflows/azure-static-web-apps-polite-pond-06e020403.yml`) — pushes to `main` auto-deploy to Azure Static Web Apps. Build app directory: `/MyDevTools`, output: `wwwroot/`.
