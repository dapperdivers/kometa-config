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
