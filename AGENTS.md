# RetroBIOS — Base44 Development Environment

## What this project is

RetroBIOS is a **MkDocs Material documentation site** generated from data files.
It is NOT a traditional web app with a backend — it's a static site builder.

The site documents BIOS/firmware packs for retrogaming platforms (RetroArch,
Batocera, Recalbox, etc.) with source-verified metadata from `database.json`,
platform configs in `platforms/`, and emulator profiles in `emulators/`.

## How the site is built

1. `scripts/generate_site.py` reads `database.json`, `platforms/*.yml`,
   `emulators/*.json`, `provenance/*.json`, and `wiki/*.md` to generate
   **567 markdown pages** into `docs/`.
2. `mkdocs serve` (Material theme) serves `docs/` as a live website.

The CI pipeline (`.github/workflows/deploy-site.yml`) also runs
`scripts/validate_schemas.py`, `scripts/refresh_data_dirs.py`, and
`scripts/validate_site.py`, but those are validation steps — the site
generates and serves without them.

## Running locally

```bash
docker compose -f docker-compose.base44.yml up -d
```

- **Port:** 3000 (mapped from mkdocs serve)
- **Health path:** `/retrobios/` (the site uses a base path from `site_url`)
- **Startup time:** ~90 seconds (pip install + site generation + mkdocs build)

The container runs `generate_site.py` once on startup, then serves with
`mkdocs serve`. To pick up changes to source data (`database.json`,
`platforms/`, `emulators/`), restart the container.

## Key details

- `docs/` is generated and gitignored — never edit it directly.
- `generate_site.py` **rewrites `mkdocs.yml`** with a generated nav section.
  This is expected; the committed `mkdocs.yml` was produced the same way.
- The `site_url` (`https://abdess.github.io/retrobios/`) causes mkdocs to
  serve under `/retrobios/` — the root `/` 302-redirects there.
- `prime_system_icons()` in `siterender.py` fetches icon availability from
  GitHub (public, no auth). Results cache in `.cache/system_icons.json`.
- Python 3.12 is required for the build scripts (uses 3.12 syntax).
  `install.py` alone supports 3.8+.
- No external secrets or credentials are needed.
