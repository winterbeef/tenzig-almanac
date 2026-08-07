# CLAUDE.md

Guidance for Claude Code (or any Claude instance) working in this repo.

## What this repo is

A [Quartz](https://quartz.jzhao.xyz/) static site publishing the **Tenzig Almanac** — an in-universe, player-facing "pre-Scream travelogue" for the Tenzig sector, a Stars Without Number tabletop campaign setting (campaign: *Dusk's Long Shadow*). It is a compiled fictional document, not a wiki about the campaign — every file should read as a primary or annotated source *within* the fiction, not as GM reference material.

All page content lives under `content/`. Everything outside `content/` (config, workflow files, this file) is ordinary Quartz repo scaffolding.

## Content conventions

- **No frontmatter/properties.** Every page opens directly with a single `# H1` title — no YAML block above it. Metadata that would normally live in frontmatter (title, source type, date) is instead written *in-fiction*, inside the page body (see "Document formats" below).
- **Wikilinks**, Obsidian/Quartz style: `[[filename]]` or `[[filename|Display Text]]`. Link target is the target file's slug (filename without `.md`), not its title. When adding a new page, check for natural cross-links to existing entries and add them both ways where it makes sense.
- **Never use `[[slug|Display]]` wikilink syntax inside a Markdown table cell.** GFM table parsing splits cells on every unescaped `|`, including the one inside the wikilink's alias syntax — this tears the link in half before it can render (confirmed by direct pipeline test, not just inspection). Backslash-escaping the pipe (`[[slug\|Display]]`) fixes the table split but then breaks the wikilink's own alias parsing instead. Inside tables, use a standard Markdown link — `[Display](slug)` — which has no pipe character at all and resolves to the same page. Wikilinks are fine everywhere else (prose, field notes, blockquotes).
- File slugs are `kebab-case` and descriptive (`the-porth-belt.md`, not `porth.md`).

## Document formats

Entries are built from a small set of in-fiction document types, used as flavor, not as a rigid schema:

- **Mandate Record** — dry, bureaucratic pre-Scream survey data, in a fenced code block:
  ```
  MANDATE RECORD — SYSTEM SURVEY
  Coordinates:   ####
  Star Type:     ...
  Population:    ...
  Tech Level:    TL#
  Starport:      Class #
  Last Verified: YYYY.DDD
  ```
  Variants: `MANDATE RECORD — POLITICAL/ORGANIZATIONAL ENTITY` (factions/companies), `MANDATE RECORD — NAVAL ASSET COMMISSIONING`, `PERSONS OF INTEREST REGISTRY`.
- **Field Dispatch** — first-person, present-tense, unpolished. Header block: `FIELD DISPATCH (RECOVERED, PRE-SCREAM)` or similar, with `Location:` / `Transmission:` lines.
- **Unverified Intelligence Log** — rumors, gossip, secondhand reports. Header block with `Reliability:` (`unverified` / `secondhand` / `confirmed`), `Location:`, `Source:`, `Logged:`.
- **Field Note** — a later annotation on any of the above, signed and dated:
  `> **[FIELD NOTE — Name/Role, stardate]**` followed by the note text.
- Primary fragments (ledgers, proclamations, letters, manifests) don't need a formal header — a simple `*[hand unknown, undated]*` attribution line is enough.

## Voice

- **Many sources, not a few recurring narrators.** Don't default to the same two or three named annotators across every entry — invent new one-off voices freely (haulers, dockhands, Mandate surveyors, smugglers, Cerberus Corps agents, Archive adepts, warlords, physicians, etc.). A handful of names may recur, but no single voice should dominate the book.
- Mix pre-Scream and post-Scream sources within an entry where it makes sense. Pre-Scream fragments don't know what's coming; post-Scream field notes read them with hindsight, and should sometimes disagree with each other.
- Dry/bureaucratic for Mandate records, terse and unpolished for dispatches, personal and opinionated for field notes. Let entries contradict each other — the book is a compilation, not a single authorial voice, and unresolved contradictions are a feature.

## In-fiction dating

- **The Scream** happened ~600 years before "now." Current campaign era is `3200.xxx`.
- Pre-Scream Mandate-era dates: roughly `2580`–`2620.xxx`.
- Post-Scream field notes/annotations: roughly `3100`–`3199.xxx`, predating the live campaign's current session dates.

## Hard constraints — do not leak GM-secret material

This is a **player-facing** document. Never include, confirm, or strongly imply any of the following, even obliquely:

- The true methodology of Project Lighthouse (psionic experimentation on children).
- The Eurymedon Concern → Shipyards of Eurymem "founding crime" / cover-up.
- The Lady of Light's true identity (Dr. Seraphine Voss) or her nature.
- Cassiel Voss's precognition or any "second Scream" foreshadowing.
- The Great Archive's true origin (seeded by the Inuar / the Promiton).
- Any named NPC's hidden loyalties or secret backstory not already public in play.

When in doubt, leave it vague, contradictory, or omit it entirely — that's consistent with the book's voice anyway.

## Quartz config

- **`ContentMeta` plugin must stay disabled** (`enabled: false` on the `github:quartz-community/content-meta` entry in `quartz.config.yaml`). It renders a real-world "date · N min read" line under every title. Since content files intentionally carry no frontmatter dates, it falls back to git/filesystem timestamps and shows today's date on a document that's supposed to be six hundred years old — an immersion-breaking anachronism, not a bug. Don't re-enable it as part of an unrelated config cleanup.

## Build & publish

```
npx quartz build --serve      # preview locally at localhost:8080
npx quartz sync                # commit + push content changes
```

GitHub Pages deploys via `.github/workflows/deploy.yml` on push to the default branch. See `README.md` for full publishing steps.
