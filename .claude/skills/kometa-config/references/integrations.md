# Kometa integrations (nightly)

Per-service connection docs and the placeholder variables this repo uses. **Read
the service's doc page before adding/changing its block** — required keys and
optional options change across versions. Never hardcode credentials; always use
the `<<PLACEHOLDER>>` form.

Doc base: `https://kometa.wiki/en/nightly/config/<service>/`

To list all config/integration doc pages from source:

```bash
gh api "repos/Kometa-Team/Kometa/git/trees/nightly?recursive=1" \
  --jq '.tree[].path | select(startswith("docs/config/") and endswith(".md"))'
```

## Services & placeholders used in this repo

| Service | Doc page | Placeholders (per project CLAUDE.md) |
|---------|----------|--------------------------------------|
| Plex | `config/plex/` | `<<PLEX_HOST>>`, `<<PLEX_API_KEY>>` |
| TMDb | `config/tmdb/` | `<<TMDB_API_KEY>>` |
| Trakt | `config/trakt/` | `<<TRAKT_CLIENT_ID>>`, … (OAuth block) |
| Radarr | `config/radarr/` | `<<RADARR_HOST>>` (+ token) |
| Sonarr | `config/sonarr/` | `<<SONARR_HOST>>` (+ token) |
| Tautulli | `config/tautulli/` | `<<TAUTULLI_HOST>>` |
| OMDb | `config/omdb/` | `<<OMDB_API_KEY>>` |
| MDbList | `config/mdblist/` | `<<MDBLIST_API_KEY>>` |

Other available integrations (read the doc page if a task needs them): AniDB,
MyAnimeList, SIMKL, GitHub, and notification targets — Apprise, Gotify, Notifiarr,
ntfy, Webhooks.

## Rules

- **Placeholders only.** `<<NAME>>` resolves from the environment / secret store at
  runtime. Never write a real token, URL, or key into `config.yml`, and never echo
  one back in output or logs.
- **Trakt/MAL use OAuth blocks** with multiple sub-keys — copy the full block shape
  from the doc page; don't abbreviate.
- **Notifications** (`webhooks:` + Apprise/Notifiarr/etc.) have their own pages
  (`config/webhooks/`, `config/apprise/`, `config/notifiarr/`, `config/ntfy/`,
  `config/gotify/`) — wire them per-doc.
- When adding a service, list the env vars the user must define so the
  `<<PLACEHOLDER>>` substitution resolves.

## Operations sources worth knowing (2.4.x)

Confirm exact keys on `https://kometa.wiki/en/nightly/config/operations/` before use.

- **`plex_csm` / `plex_csm0`** — sources for `mass_content_rating_update` that read the
  Common Sense Media age rating from Plex's own cached `commonSenseMedia.ageRatings`,
  **no API key**. Prefer it ahead of `mdb_commonsense` (which needs an MDbList key) in
  the priority list to cut external calls: `[plex_csm, mdb_commonsense, omdb]`.
- **`mass_poster_update` / `mass_background_update`** — accept a `language:` sub-key
  (TMDb only). `language: xx` fetches **textless** poster/background art.
  ⚠ With overlays, this resets the poster and reapplies all overlays each run → image
  bloat / longer runtime. Mitigate with `ignore_locked: true` and an infrequent
  `schedule` (e.g. `monthly(last)`); local assets still win under `prioritize_assets`.
  ⚠ **Don't use it in this repo** — posters are managed by **Posterizarr** (writes to
  `/assets`, which wins via `prioritize_assets: true`), so `mass_poster_update` is
  redundant for covered items and only adds poster churn/runtime. Do textless art in
  Posterizarr instead.
