---
name: kometa-overlays
description: Author and edit Kometa overlay files (the YAML loaded via overlay_files) and the images/fonts they use. Use when adding or changing overlays such as ratings, resolution/4K/HDR/Dolby Vision, audio codec, editions, ribbons, banners, mediastinger/direct-play, or any positioned text/image badge on posters. Triggers on overlay positioning (horizontal/vertical align, offsets), overlay templates and builder_level, weight/group precedence, queues for overlays-in-a-row, or applying Kometa overlay "defaults". Always verifies overlay attributes and default template_variables against the live Kometa nightly docs and defaults/overlays source before proposing changes.
metadata:
  version: "1.0"
  kometa_branch: nightly
---

# Kometa Overlays

Author and edit **overlay files** (`overlays:` blocks loaded via `overlay_files`)
and the supporting `images/` and `fonts/` assets. This config tracks **nightly**,
where overlay positioning keys, default `template_variables`, and the bundled
overlay images change often — so **read the source before writing YAML.**

## Golden rule — read the source first, then write

Overlay positioning (`horizontal_align`, `vertical_align`, `horizontal_offset`,
`vertical_offset`, `group`, `weight`, `queue`, `builder_level`) and the default
overlay variables are the easiest things to get subtly wrong. Never guess them.
Pull the current overlay docs and, when using a Kometa overlay default, its real
YAML, before proposing changes. Cite what you read.

## Step 1 — Pull the authoritative source

This repo runs **nightly**. Bases (never `latest`/`master`):

- Docs: `https://kometa.wiki/en/nightly/<path>/`
- Raw YAML/images: `https://raw.githubusercontent.com/Kometa-Team/Kometa/nightly/<path>`
- Browse: `https://github.com/Kometa-Team/Kometa/tree/nightly/`

Fetch the page(s) that match the task with `WebFetch`:

| Task                                                   | Doc URL                                                    |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| Overlay file structure + all overlay attributes        | `https://kometa.wiki/en/nightly/files/overlays/`           |
| Positioning, `group`/`weight`, queues, `builder_level` | `https://kometa.wiki/en/nightly/files/overlays/`           |
| Templates & external_templates                         | `https://kometa.wiki/en/nightly/files/templates/`          |
| Builders (what items an overlay applies to)            | `https://kometa.wiki/en/nightly/files/builders/overview/`  |
| Overlay defaults overview + shared variables           | `https://kometa.wiki/en/nightly/defaults/overlays/`        |
| A specific overlay default (e.g. ratings, resolution)  | `https://kometa.wiki/en/nightly/defaults/overlays/<name>/` |

See `references/overlay-attributes.md` for the positioning/precedence cheat-sheet
that maps each concept to the source.

**When the task uses an overlay `default:`** read the real default YAML to get the
correct `template_variables`, image names, and offsets. Overlay defaults live in
`defaults/overlays/<name>.yml` with images under `defaults/overlays/images/`:

```bash
# List overlay default modules and bundled images on nightly
gh api "repos/Kometa-Team/Kometa/git/trees/nightly?recursive=1" \
  --jq '.tree[].path | select(startswith("defaults/overlays/"))'
```

Then `WebFetch` the raw file, e.g.
`https://raw.githubusercontent.com/Kometa-Team/Kometa/nightly/defaults/overlays/audio_codec.yml`.

## Step 2 — Inspect the existing config

- **Docker path mapping:** overlay files and assets are referenced by their
  in-container path `/config/...` (repo root is mounted at `/config`). Local
  images/fonts live under `overlays/images/...` and `overlays/fonts/...`.
- **Overlay ordering matters.** Overlays in the same `group` compete by `weight`
  (highest wins, one shows); overlays meant to coexist in a row use `queue`
  positions. Check how existing overlays in this repo share groups/queues before
  adding a new badge so you don't silently suppress an existing one.
- This repo schedules overlays via `run_order` (`overlays` runs after
  `collections`/`metadata`/`operations`) and per-library `template_variables`.
  Read `config.yml`'s library blocks and mirror the existing overlay wiring.
- Overlay YAML under `overlays/` is formatted by the oxfmt pre-commit hook (only
  auto-generated `*_report.yml` is excluded); don't hand-format it.

## Step 3 — Author or edit

- Set positioning keys from Step 1's source, not memory. Confirm the coordinate
  origin and units (px) on the overlays page before computing offsets.
- For text overlays, point `font:` at a real file under `overlays/fonts/` (or a
  Kometa-bundled font) and verify `font_size`/`font_color` keys against the docs.
- For image overlays, ensure the referenced PNG exists under `overlays/images/`
  (or pull the right one from `defaults/overlays/images/` on nightly).
- Use `builder_level` (e.g. `show`, `season`, `episode`) per the docs when the
  overlay targets non-movie levels.
- Reuse templates/anchors already in the file instead of duplicating blocks.
- Hand `schedule` / `visible_*` decisions to the **kometa-scheduling** skill.

## Step 4 — Validate

```bash
mise run validate                                                                      # config.yml + all linked files, offline → validate.log
mise run kometa -- --validate-file /config/overlays/<file>.yml --validate-level structure   # one overlay file, targeted
oxfmt <file>.yml                                                                       # format (also auto-run by lefthook pre-commit)
```

These tasks run the pinned `kometateam/kometa:nightly` image in Docker (no local
`kometa` install needed). The repo is mounted at `/config`, so `--validate-file`
paths must be **`/config`-prefixed**. Prefer `mise run validate` — the overlay
files are linked from config.yml, so it already covers them; it's also the check
the lefthook **pre-push** hook enforces. `--validate-file` validates _Kometa_
overlay semantics against the JSON schema offline; use `--validate-level full` for
the deepest pass (connects to Plex/APIs). `oxfmt` (`.oxfmtrc.json`) replaces the
old prettier/yamllint setup.

Overlays are visual — YAML validity is necessary but not sufficient. For a real
check, the `testing/` harness runs Kometa against a throwaway Plex and
`testing/test-overlays.sh` resets then re-applies overlays so you can eyeball the
layout in Plex (`http://localhost:32400/web`). Recommend that flow for any
positioning/precedence change; don't claim a layout works from YAML alone.

## Output

Name the source pages you read (URLs), give the YAML diff, list any new
image/font assets and their `/config/...` paths, and call out any `group`/`weight`/
`queue` interactions with existing overlays. If an attribute wasn't in the
current source, say so instead of guessing.
