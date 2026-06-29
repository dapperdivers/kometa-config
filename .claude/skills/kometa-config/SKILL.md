---
name: kometa-config
description: Author and edit the main Kometa config.yml — libraries, global/library settings, operations, playlists, and service connections (plex, tmdb, trakt, radarr, sonarr, tautulli, mdblist, omdb, webhooks/notifications). Use when wiring a new library, adding/adjusting settings or operations, connecting or troubleshooting an API integration, managing template_variables at the library level, or wiring collection_files/overlay_files/metadata_files into a library. Triggers on changes to config.yml structure, integration credentials/placeholders, asset settings, or run order. Always verifies setting/operation keys against the live Kometa nightly docs before proposing changes.
metadata:
  version: "1.0"
  kometa_branch: nightly
---

# Kometa config.yml

Author and edit the **main `config.yml`**: `libraries`, `settings`, `operations`,
`playlist_files`, and the service connection blocks. This config tracks
**nightly**, where setting names, operation keys, and integration options change
between releases — so **read the source before writing YAML.**

## Golden rule — read the source first, then write

Global `settings`, per-library overrides, and `operations` have many keys whose
names and defaults drift across versions. Never propose them from memory. Pull
the matching nightly doc page and confirm each key exists and behaves as you
expect before changing it. Cite what you read.

## Step 1 — Pull the authoritative source

This repo runs **nightly**. Bases (never `latest`/`master`):

- Docs: `https://kometa.wiki/en/nightly/<path>/`
- Raw: `https://raw.githubusercontent.com/Kometa-Team/Kometa/nightly/<path>`
- Browse: `https://github.com/Kometa-Team/Kometa/tree/nightly/`

Fetch the page(s) that match the task with `WebFetch`:

| Task | Doc URL |
|------|---------|
| Config file overview / top-level structure | `https://kometa.wiki/en/nightly/config/overview/` |
| Libraries block | `https://kometa.wiki/en/nightly/config/libraries/` |
| Global & library settings | `https://kometa.wiki/en/nightly/config/settings/` |
| Operations (split roles, mass updates, etc.) | `https://kometa.wiki/en/nightly/config/operations/` |
| Playlists | `https://kometa.wiki/en/nightly/config/playlists/` |
| Schedule (run-time scheduling concepts) | `https://kometa.wiki/en/nightly/config/schedule/` |
| A specific integration | see `references/integrations.md` |

`references/integrations.md` has the per-service doc URLs and the placeholder
variables this repo uses.

## Step 2 — Inspect the existing config

`config.yml` in this repo is anchor-heavy and intentionally DRY. Before editing:

- **Read the anchors first.** Shared blocks are defined once and reused:
  `&run_order` (`collections → metadata → operations → overlays`), library-level
  `template_variables` (e.g. `sep_style`, `collection_mode`, `language`,
  `placeholder_imdb_id`), and per-section visibility anchors like
  `&daily_visibility`. Reuse the existing anchor (`<<: *anchor`) rather than
  pasting a literal block.
- **Credentials are placeholders, never literals.** Integrations use
  `<<PLEX_HOST>>`, `<<TMDB_API_KEY>>`, `<<RADARR_HOST>>`, etc. — these resolve
  from the environment/secret store. Never hardcode a real key or URL, and never
  print secrets. Keep new integrations on the same placeholder pattern.
- **In-container paths.** File references use `/config/...` (repo root mounted at
  `/config` in the Kometa container). Wire new files as
  `collection_files: / overlay_files: / metadata_files: - file: /config/...`.
- **Don't touch generated files.** `*_report.yml` and `logs/` are auto-generated
  (prettier-ignored) — never hand-edit them.

## Step 3 — Author or edit

- Add libraries/settings/operations using keys confirmed in Step 1. Quote the doc
  line you relied on for any non-obvious key.
- Keep `run_order` and visibility via the existing anchors; if a new pattern is
  truly needed, define a new anchor rather than duplicating literals.
- For a new integration, add the minimal connection block from that service's doc
  page using `<<PLACEHOLDER>>` variables, then note which env vars must be set.
- Defer collection/overlay *content* to the **kometa-collections** /
  **kometa-overlays** skills; this skill owns the wiring and global behavior.
- Defer `schedule` / `visible_*` timing to the **kometa-scheduling** skill.
- **Recommendation Hub ordering** (Plex Pass): `auto_sort_hubs` (global/library
  setting — `configured` / `alpha` / `random`, each ±`.desc`) sorts the home-screen
  hub rows after processing; it only affects collections promoted via `visible_home`
  / `visible_shared`. Pin individual collections to the top with the `hub_priority`
  attribute (lower = higher; also available as a `hub_priority` template variable on
  defaults, and `hub_priority_<<key>>` for dynamic keys). Confirm on the settings doc
  and `defaults/templates.yml` source before use.

## Step 4 — Validate

```bash
yamllint config.yml                 # .yamllint: line-length 200, 2-space indent, key-duplicates on
npx prettier --check config.yml     # .prettierrc.yml: double quotes, no bracket spacing
kometa --validate-file config.yml   # Kometa-native config schema check → validate.log
```

`kometa --validate-file` validates the config against `config-schema.json` (unknown
keys, wrong types, missing required fields) — far stronger than yamllint/prettier,
which only check syntax/format. Use `--validate-level full` for the deepest pass.
Note: `--validate-file` is standalone, but bare `--validate` implies an immediate
run, so don't use it as a dry check.

`yamllint` here has `key-duplicates: enable` — duplicate keys are a real error,
common when copy-pasting library/integration blocks; watch for it. For a live
check, the `testing/` Docker harness (`testing/docker-compose.yml`,
`kometateam/kometa:latest`) runs the full config against a throwaway Plex.
Validate YAML before suggesting a run, and never run against production Plex to "test."

## Output

Name the source pages you read (URLs), give the `config.yml` diff, list any new
`<<PLACEHOLDER>>` env vars introduced, and confirm anchors were reused rather than
duplicated. If a setting/operation key wasn't in the current source, say so
instead of guessing.
