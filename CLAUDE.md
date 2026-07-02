# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A personal portfolio site for Eugen Trapeznikov (Product Designer), built with Astro (no UI framework — plain `.astro` components, zero client-side JS frameworks). Static, content-driven pages showcasing case studies, initiatives, and a career timeline.

## Commands

```sh
npm run dev       # start dev server at localhost:4321
npm run build     # production build to ./dist/
npm run preview   # preview the production build locally
npm run astro ...   # run Astro CLI commands, e.g. `npm run astro check` for type-checking
```

There is no test suite, linter, or formatter configured in this repo.

## Architecture

### Two coexisting layout/design systems

The codebase currently has **two separate, incompatible styling systems** — be aware of which one a file uses before editing it:

1. **Current system** (`SidebarLayout.astro` → `Layout.astro` → `public/tokens.css`): used by `src/pages/index.astro` and `src/pages/cases/[slug].astro`. All styling goes through CSS custom properties defined in `public/tokens.css` (`--bg-page`, `--content-primary`, `--accent-default`, `--space-*`, `--text-*`, `--radius-*`, etc.), with light/dark values swapped via `[data-theme="dark"]` on `<html>`. New pages/components should use this system and these tokens — don't hardcode colors or introduce a new token scheme.
2. **Legacy system** (`Base.astro`): used only by `src/pages/cases/spotlight.astro`. Defines its own separate CSS variables (`--bg`, `--text`, `--acc`, etc.) inline in `<style>` and the spotlight page uses heavy inline `style="..."` attributes instead of tokens. This page is a known outlier — don't copy its patterns into new work, and don't assume its variables exist elsewhere.

`src/layouts/Layout.astro` (the un-prefixed one, distinct from `SidebarLayout.astro`) also appears to contain an older/alternate sidebar shell (a `/about`, `/experience`, `/contact` nav that doesn't correspond to any real pages) and is not actually wired into any current page — treat it as legacy scaffolding, not a pattern to extend.

### Page structure

- `src/pages/index.astro` — the single-page home (`/`), composed of anchor-linked sections (`#home`, `#works`, `#initiatives`, `#contact`) rendered inside `SidebarLayout`. Work/case entries are defined inline as a `works` array at the top of the file.
- `src/pages/cases/[slug].astro` — dynamic case-study route using `getStaticPaths()` with a **hardcoded `cases` object** (currently `yollet` and `livematch`) instead of a content collection. To add a new case study, add a new key to this object (and a matching entry to `getStaticPaths()`) following the existing shape (`overview`, `context`/`contextList`, `auditHeading`/`auditText`, `designSystem`, `designProcess`, `uxStrategy`, `exploration`, `results.statRows`, etc.) — the template below iterates these fields directly, so all fields are required.
- `src/pages/cases/spotlight.astro` — a standalone, one-off case study page (legacy `Base` layout, see above); not part of the `[slug]` system.
- `src/data/timeline.ts` — exports `timeline` (career history, rendered in the sidebar via `TimelineItem`) and `navItems` (sidebar nav links). Edit here to update career history or top-level navigation instead of editing components directly.

### Sidebar shell

`SidebarLayout.astro` renders a persistent left sidebar (profile summary + `NavItem` nav + `TimelineItem` career list + light/dark theme switch) alongside a `<slot />` for page content, plus `MobileHeader.astro` for the collapsed mobile nav (sidebar is hidden below 960px). Theme state is persisted to `localStorage` and toggled by adding/removing `data-theme="dark"` on `<html>`; the mobile header's theme toggle syncs with the desktop one via matching DOM IDs (`themeToggle`/`themeLabel`) — if you rename these IDs in one component, update the other.

Section-based active-nav highlighting on the home page uses an `IntersectionObserver` over `section[id]`/`main[id]` elements matched against `[data-nav]` links — new top-level home sections need both an `id` and a corresponding `navItems` entry to participate.

### Components (`src/components/`)

Presentational, prop-driven `.astro` components consumed by the pages above: `Avatar`, `WorkCard` (two layouts: `full`/`half`, with an animated stat-bar reveal on hover), `InitiativeCard` (content keyed by a fixed union type, not data-driven), `Tag`, `SocialButton`, `NavItem`, `TimelineItem`, `MobileHeader`. `Welcome.astro` is unused leftover scaffolding from the Astro starter template.

### Static assets

`public/cases/<slug>/` holds case-study images referenced by absolute path (`/cases/yollet/...`) from the `cases` object in `[slug].astro`. `public/initiatives/` holds initiative logos referenced from `InitiativeCard.astro`. `public/tokens.css` is loaded directly via `<link>` in `Layout.astro`, not imported through Astro's asset pipeline.
