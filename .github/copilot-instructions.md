# Copilot Workspace Instructions

## Development Checklist

Before committing any changes, ensure:

- [ ] `dotnet build` passes with no errors
- [ ] Code follows C# conventions (PascalCase for public members, camelCase for private fields)
- [ ] No unused variables or imports
- [ ] New CSS utility classes are added to `wwwroot/css/app.css` (do NOT add a CDN link or Tailwind CLI)

## Project Overview

**Soc Ops** is a Social Bingo game built with Blazor WebAssembly (.NET 10). Players find people who match questions to mark squares and get 5 in a row. Deployed to GitHub Pages — no server, no backend.

This repo is also a **GitHub Copilot workshop** (`agent-lab-dotnet`). The `.solutions/` directory contains full project snapshots for each lab step (step-00 through step-07 plus `finished`). Do not modify `.solutions/` unless the task is explicitly about updating a lab step.

## Architecture

```
SocOps/
├── Components/     # Reusable Blazor components (BingoBoard, BingoSquare, GameScreen, StartScreen, BingoModal)
├── Data/           # Static data — Questions.cs (question bank + FREE_SPACE constant)
├── Layout/         # App shell — MainLayout.razor (minimal full-height wrapper only)
├── Models/         # Pure C# data models (BingoSquareData, BingoLine, GameState enum)
├── Pages/          # Routable pages — Home.razor is the single entry point
├── Services/
│   ├── BingoGameService.cs    # Scoped DI — owns all mutable state, localStorage persistence, OnStateChanged event
│   └── BingoLogicService.cs   # Pure static service — board generation, toggle, bingo detection (no DI, no state)
└── wwwroot/
    ├── css/app.css            # ALL styles — hand-authored utility classes (Tailwind-like naming)
    └── index.html             # WASM host page
```

## Key Commands

```bash
dotnet run --project SocOps       # Dev server at http://localhost:5166
dotnet build SocOps/SocOps.csproj # Build only
```

There are no tests in this project.

## Blazor Patterns

- **State subscription**: Components inject `BingoGameService`, subscribe to `OnStateChanged`, and call `StateHasChanged()` on updates. Always unsubscribe in `Dispose()`.
- **Immutable updates**: `ToggleSquare` returns a new `List<BingoSquareData>` — never mutate the existing list.
- **Fire-and-forget persistence**: Use `_ = SaveGameStateAsync()` to persist without blocking the render cycle.
- **EventCallback for interactions**: Parent components pass `EventCallback` or `EventCallback<T>` parameters to children; children invoke them on user actions.
- **CSS class logic in C#**: Compute conditional class strings in a C# method (e.g., `GetCssClasses()`) inside `.razor` files rather than using scoped CSS or inline styles.

## Styling

All CSS is in `SocOps/wwwroot/css/app.css`. There is **no Tailwind CLI** and **no CDN** — utility classes are manually defined. Follow these naming conventions when adding new utilities:

- Layout: `.flex`, `.grid`, `.items-center`, `.justify-between`
- Spacing: `.p-4`, `.mb-2`, `.mx-auto`
- Colors: `.bg-accent`, `.bg-marked`, `.text-gray-700`
- Do not introduce external CSS frameworks or `<link>` CDN tags.

## State & Persistence

- `BingoGameService` serializes all state to `localStorage` under key `"bingo-game-state"`.
- Storage version is controlled by `STORAGE_VERSION = 1` — increment this constant when making breaking changes to the stored shape, and add a migration branch.
- `IJSRuntime` is used for all localStorage reads/writes via JS interop.

## Common Pitfalls

- **Do not add a server project** — this is a pure WASM SPA; all logic must run in the browser.
- **Do not introduce NuGet packages** without a clear reason; the bundle is intentionally minimal.
- **Do not edit `.solutions/`** unless working on a specific lab step.
- **Do not use Tailwind CLI or CDN** — add utility classes directly to `app.css`.
- `BingoLogicService` must remain a static class with no DI dependencies; keep game logic pure and testable.
