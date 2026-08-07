# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is **The Tenzig Almanac**, a personal digital-garden/wiki (fantasy worldbuilding notes) published with **Quartz v5** — a static site generator forked from [jackyzha0/quartz](https://github.com/jackyzha0/quartz). Two things live in this one repo:

- `quartz/` — the site-generator engine (TypeScript/Preact, plugin pipeline, CLI). This is framework code.
- `content/` — the actual almanac content (Markdown notes: `avire.md`, `cantho.md`, `lycos.md`, etc.). This is what gets published.

`docs/` contains upstream Quartz's own documentation pages (unrelated to the almanac content) and `public/` is build output (gitignored).

## Commands

```bash
npm install                    # install deps
npm run install-plugins        # (also runs as `prebuild`) fetch/install plugins declared in quartz.config.yaml
npx quartz build                # build site to public/
npx quartz build --serve --watch  # local dev server with live reload (default port 8080)
npm run check                   # tsc --noEmit && prettier --check
npm run format                   # prettier --write
npm test                        # tsx --test  (runs all **/*.test.ts)
npx tsx --test quartz/util/path.test.ts   # run a single test file
npx quartz sync                 # commit/push/pull content via git
npx quartz plugin list|add|remove|enable|disable|config|prune   # manage plugins
```

CI (`.github/workflows/ci.yaml`) runs, in order: `npx quartz plugin install`, `npm run check`, `npm test`, `npx quartz build --bundleInfo -d docs`. Deploy (`deploy.yaml`) builds with `npx quartz build` and publishes `public/` to GitHub Pages on push to `v5` (the default branch, not `main`).

## Architecture

### Config-driven plugin system (the key deviation from upstream Quartz)

Unlike stock Quartz (which declares plugins in `quartz.config.ts`), **this fork loads plugins declaratively from `quartz.config.yaml`** at build time via `quartz/plugins/loader/config-loader.ts` (`loadQuartzConfig` / `loadQuartzLayout`). To change site behavior, edit `quartz.config.yaml`, not TypeScript.

- Each entry under `plugins:` in the YAML names a `source` (npm package like `@quartz-community/search`, or a git/local source), `enabled`, `order`, `options`, and optionally a `layout` block.
- Plugins are real npm/git packages (the `@quartz-community/*` and `@quartz-themes/*` scopes), installed into `.quartz/plugins/` and resolved at load time — they are not part of this repo's source tree.
- `config-loader.ts` determines each plugin's category (`transformer` | `filter` | `emitter` | `pageType`, or `component`) from its `package.json` `"quartz"` field (preferred) or a `manifest.ts` export, sorts by `order` within category, then instantiates them via a `default`/`plugin` export convention.
- `quartz.config.default.yaml` is a reference/template config (not the active one) — `resolveConfigPath()` only falls back to it if `quartz.config.yaml` is missing.
- `quartz.plugins.json` is a legacy pre-YAML format still supported as a fallback.

### Layout system

Page layout (where components render: `header`/`left`/`right`/`beforeBody`/`afterBody`/`footer`) is also declared per-plugin in YAML via each entry's `layout:` block (`position`, `priority`, optional `group`/`groupOptions` for flex groupings like the `toolbar` group, `display: mobile-only|desktop-only`, `condition`). Top-level `layout.byPageType` in the YAML overrides positions/exclusions per page type (`404`, `content`, `folder`, `tag`, `canvas`, `bases`). This resolution happens in `buildLayoutForEntries`/`resolveGroups` in `config-loader.ts`.

### Build pipeline

Entry point `quartz.ts` → `quartz/build.ts` (`buildQuartz`). For each content file: `processors/parse.ts` (Markdown → HAST via the configured `transformers`' `markdownPlugins`/`htmlPlugins`) → `processors/filter.ts` (drop pages per each `filter` plugin's `shouldPublish`) → `processors/emit.ts` (each `emitter` plugin writes output files, e.g. HTML pages, RSS, sitemap, assets). `pageType` plugins (content/folder/tag/canvas/bases/404) determine per-page `match`/`body`/`layout`. Watch mode (`--watch`) re-runs this incrementally per changed file via `chokidar`.

### CLI

`quartz/bootstrap-cli.mjs` (yargs) exposes `create`, `build`, `sync`, `upgrade`/`update`, `restore`, `tui` (interactive plugin manager), and `plugin <install|add|remove|list|enable|disable|config|prune>`. Handlers live in `quartz/cli/handlers.js`; plugin install/git resolution logic is in `quartz/plugins/loader/gitLoader.ts`.

### Content conventions

Content Markdown uses Obsidian-flavored syntax (wikilinks, callouts, mermaid, tags, block references — see the `@quartz-community/obsidian-flavored-markdown` options in `quartz.config.yaml`). `content/index.md` is the site root/home page. The active theme is `alien` (`@quartz-themes/*`), configured via the `@quartz-themes/core` plugin entry and `configuration.theme` colors in `quartz.config.yaml`.
