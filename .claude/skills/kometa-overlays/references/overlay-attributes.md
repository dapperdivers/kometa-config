# Overlay positioning & precedence cheat-sheet (nightly)

A map to the source, not a replacement for it. Confirm exact key names and
accepted values on `https://kometa.wiki/en/nightly/files/overlays/` before use.

## Positioning

| Concept | Key (verify on docs) | Notes |
|---------|---------------------|-------|
| Horizontal anchor | `horizontal_align` | `left` / `center` / `right` |
| Vertical anchor | `vertical_align` | `top` / `center` / `bottom` |
| Horizontal nudge | `horizontal_offset` | pixels from the anchor |
| Vertical nudge | `vertical_offset` | pixels from the anchor |
| Back box / shape | `back_color`, `back_width`, `back_height`, `back_radius`, … | for text/badge backgrounds |
| Font | `font`, `font_size`, `font_color`, `font_style` | `font` must resolve to a real file |

Always check the **coordinate origin** and whether offsets stack with align on the
overlays page — it has changed across versions.

## Precedence: group vs weight vs queue

- **`group`** + **`weight`**: overlays in the same `group` are mutually exclusive —
  only the highest `weight` is drawn. Use this for "one of N" badges (e.g. a single
  resolution badge: 4K beats 1080p).
- **`queue`**: a named set of fixed positions so multiple overlays render *in a row*
  without overlapping. Use this for "overlays in a row" (this repo does this — see
  the `feat(overlay) overlays in row` history).
- Mixing them up is the #1 overlay bug: a new badge given the wrong `group`
  silently hides an existing one; a row badge without a `queue` slot overlaps.

## builder_level

Targets the overlay at a tier: movie (default), `show`, `season`, `episode`.
Episode-info / gradient overlays in `defaults/overlays/` show correct usage.

## Bundled overlay images & fonts (nightly)

```bash
gh api "repos/Kometa-Team/Kometa/git/trees/nightly?recursive=1" \
  --jq '.tree[].path | select(startswith("defaults/overlays/images/"))'
```

Local assets in this repo live under `overlays/images/...` and `overlays/fonts/...`
and are referenced as `/config/overlays/...`. If you need a badge image that isn't
local, pull it from `defaults/overlays/images/` on nightly rather than inventing a
filename.

## Defaults worth reading before re-implementing by hand

`audio_codec`, `aspect`, `commonsense`, `content_rating_us_movie` /
`content_rating_us_show`, `direct_play`, `episode_info`, plus ratings/resolution
modules. List them:

```bash
gh api "repos/Kometa-Team/Kometa/git/trees/nightly?recursive=1" \
  --jq '.tree[].path | select(startswith("defaults/overlays/") and endswith(".yml"))'
```

Prefer configuring a default via `template_variables` over hand-rolling an overlay
that already exists as a default.
