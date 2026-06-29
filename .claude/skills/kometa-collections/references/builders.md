# Kometa builder catalog (nightly)

A builder is what populates a collection. Every collection/dynamic collection
needs at least one. **Always confirm the exact attribute keys from the doc page
before using a builder** — this list is a map to the source, not a substitute
for it.

Doc base: `https://kometa.wiki/en/nightly/files/builders/<service>/<page>/`
Overview: `https://kometa.wiki/en/nightly/files/builders/overview/`

To regenerate this list from source:

```bash
gh api "repos/Kometa-Team/Kometa/git/trees/nightly?recursive=1" \
  --jq '.tree[].path | select(startswith("docs/files/builders/") and endswith(".md"))'
```

## By service (nightly doc folders)

| Service | Builder docs folder | Notes |
|---------|--------------------|-------|
| Plex | `builders/plex/` | `all`, `search`, `smart-filter`, `collectionless`, `watchlist`, `pilots`; `search-options` + `sort-options` reference pages |
| TMDb | `builders/tmdb/` | charts (`now-playing`, `popular`, `airing-today`, …), lists, collections, people |
| Trakt | `builders/trakt/` | user lists, charts, watchlist, recommendations |
| IMDb | `builders/imdb/` | `chart`, `list`, `search`, `award`, `watchlist`, `id` |
| Letterboxd | `builders/letterboxd/` | `list`, `user_films`, `user_reviews` |
| MDbList | `builders/mdblist/` | `list` |
| Tautulli | `builders/tautulli/` | `popular`, `watched` (Plex activity-driven) |
| Radarr / Sonarr | `builders/radarr/`, `builders/sonarr/` | `all`, `taglist` |
| AniList / MyAnimeList / AniDB / SIMKL | `builders/anilist/`, `builders/myanimelist/`, `builders/anidb/`, `builders/simkl/` | anime sources |
| Box Office Mojo | `builders/boxofficemojo/` | `domestic`, `international`, `world`, `all-time`, `record` |
| ICheckMovies | `builders/icheckmovies/` | `list` |
| StevenLu | `builders/stevenlu/` | `popular` |
| Text file | `builders/textfile/` | build from a local list of titles/ids |

## This repo's common builders

The `custom/movie_subgenre/` collections build primarily from:

- **Plex `keyword` / smart filters** — driven by the `keyword` / `keyword.any`
  template variables flowing into `custom/templates/movie_subgenre_template.yml`.
- **Letterboxd lists** — via the `letterboxd_list` template variable.

`config.yml` mixes those custom files with Kometa **defaults** (`based`, award
modules like `oscars`/`bafta`, chart modules like `imdb`/`other_chart`), each of
which has its own builder baked in — read the default's YAML to see it.

## Filters vs builders

Builders *add* items; **filters** (`filters:`) *remove* items the builders found.
Don't reach for a filter when a builder attribute (e.g. `tmdb_search` params,
`plex_search` options) can scope the set directly. Filter reference:
`https://kometa.wiki/en/nightly/files/filters/`.
