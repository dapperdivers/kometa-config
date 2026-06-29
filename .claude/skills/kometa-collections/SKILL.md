---
name: kometa-collections
description: Author and edit Kometa collection, dynamic-collection, and metadata files (the YAML loaded via collection_files / metadata_files). Use when creating or changing collections, dynamic_collections, builders (plex/tmdb/trakt/imdb/mdblist/letterboxd/tautulli/etc.), filters, smart collections, templates/external_templates, or when applying Kometa "defaults" collection modules. Triggers on requests to add a collection, build a genre/subgenre/award/chart list, tune builders or filters, or edit item metadata. Always verifies syntax against the live Kometa nightly docs and defaults source before proposing changes.
metadata:
  version: "1.0"
  kometa_branch: nightly
---

# Kometa Collections

Author and edit **collection files** (`collections:`, `dynamic_collections:`,
`templates:`, `external_templates:`) and **metadata files** (`metadata:`) for
this repo. Kometa moves fast and this config tracks the **nightly** branch, so
the cardinal rule is: **read the current source before you write YAML.**

## Golden rule — read the source first, then write

Never propose collection YAML from memory. Builder names, attribute keys,
default template_variables, and filter syntax change between releases. Before
proposing or editing anything, pull the relevant page(s) from the nightly docs
and, when using a Kometa default, the actual default YAML from the nightly
branch. Cite what you read.

## Step 1 — Pull the authoritative source

This repo runs **nightly**. Use these bases (never `latest`/`master`):

- Docs: `https://kometa.wiki/en/nightly/<path>/`
- Raw YAML: `https://raw.githubusercontent.com/Kometa-Team/Kometa/nightly/<path>`
- Browse: `https://github.com/Kometa-Team/Kometa/tree/nightly/`

Fetch the page(s) that match the task with `WebFetch`:

| Task | Doc URL |
|------|---------|
| Collection file structure / attributes | `https://kometa.wiki/en/nightly/files/collections/` |
| Dynamic collections | `https://kometa.wiki/en/nightly/files/dynamic/` |
| Dynamic collection types | `https://kometa.wiki/en/nightly/files/dynamic_types/` |
| Builders (overview + per-service) | `https://kometa.wiki/en/nightly/files/builders/overview/` |
| Filters | `https://kometa.wiki/en/nightly/files/filters/` |
| Templates & external_templates | `https://kometa.wiki/en/nightly/files/templates/` |
| Metadata (editing item details) | `https://kometa.wiki/en/nightly/files/metadata/` |
| Defaults usage / shared variables | `https://kometa.wiki/en/nightly/defaults/collection_list/` |
| A specific default (e.g. genre, oscars) | `https://kometa.wiki/en/nightly/defaults/<category>/<name>/` |

See `references/builders.md` for the full builder catalog and where each lives
in the docs.

**When the task uses a Kometa `default:`** also read the default's real YAML to
get the correct `template_variables`, exclusions, and schedule keys:

```bash
# Find where a default module actually lives on nightly (genre is in both/, not movie/)
gh api "repos/Kometa-Team/Kometa/git/trees/nightly?recursive=1" \
  --jq '.tree[].path | select(startswith("defaults/") and contains("genre"))'
```

Then `WebFetch` the raw file, e.g.
`https://raw.githubusercontent.com/Kometa-Team/Kometa/nightly/defaults/both/genre.yml`.
Defaults are organized as `defaults/{award,both,chart,movie,show,overlays}/<name>.yml`.

**Version sanity check (optional but preferred):** confirm what's actually
installed before trusting nightly. The test harness pins `kometateam/kometa:latest`
in `testing/docker-compose.yml`; the production version is whatever the user's
runner pulls. If a feature is brand-new on nightly, flag that it may not be in
the running build yet.

## Step 2 — Inspect the existing config

Match this repo's established conventions before adding anything:

- **Docker path mapping:** files are referenced by their in-container path
  `/config/...` (the repo root is mounted at `/config`). New collection files
  under `custom/` are wired into `config.yml` as
  `- file: /config/custom/<dir>/<file>.yml`.
- **Custom collections** live in `custom/` and lean heavily on YAML anchors +
  merge keys to stay DRY. See `custom/movie_subgenre/*.yml`: a shared
  `external_templates` points at `custom/templates/movie_subgenre_template.yml`,
  and each file defines per-file `&sched` / `&tmpl` anchors that later
  collections merge with `<<: *tmpl`, overriding only what differs
  (`keyword`, `letterboxd_list`, `rating.gte`, `votes.gte`).
- **Kometa defaults** are added in `config.yml` as `- default: <name>` with an
  optional `template_variables:` block (see the award/chart blocks already there).
- Read the target file and the nearest sibling first; mirror its anchor names,
  comment density, and schedule pattern rather than inventing a new shape.

## Step 3 — Author or edit

- Every collection needs **at least one builder**. Pick the builder from Step 1's
  source, not from memory; copy attribute keys verbatim from the docs.
- Prefer extending the existing template/anchor over duplicating blocks.
- For dynamic collections, set `type:` from the dynamic_types page and use
  `template_variables` / `exclude` / `addons` exactly as documented.
- Keep `schedule` / `visible_home` / `visible_shared` consistent with the
  **kometa-scheduling** skill (schedule must run at least as often as any
  `visible_*`). Hand scheduling decisions to that skill.
- Keep `/config/...` absolute paths for any `file:`/`external_templates` refs.

## Step 4 — Validate

```bash
yamllint <file>.yml                 # config in .yamllint (line-length 200, 2-space indent)
npx prettier --check "<file>.yml"   # config in .prettierrc.yml (double quotes, no bracket spacing)
```

`*_report.yml` and `logs/` are auto-generated and prettier-ignored — don't edit
or lint those. For a real run, the `testing/` Docker harness applies the config
against a throwaway Plex (`testing/docker-compose.yml`, `kometateam/kometa:latest`).
Validate YAML before suggesting a full run.

## Output

State which source pages you read (with URLs), then give the YAML diff and the
exact `config.yml` wiring line if a new file was added. If a builder/attribute
wasn't found in the current source, say so rather than guessing.
