# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Kometa (formerly Plex Meta Manager) configuration repository for managing Plex media library metadata, collections, and overlays. Kometa automates metadata management, creates dynamic collections, and applies visual overlays to media items.

## Key Architecture

### Configuration Structure
- **config.yml**: Main configuration file defining libraries, settings, and integrations
- **overlays/**: Custom overlay templates for visual enhancements (ratings, media info)
- **custom/**: Custom collection definitions (e.g., movie subgenres)
- **logs/**: Kometa execution logs (meta.log and rotated logs)
- **Report files**: YAML reports generated for each library (Movies_report.yml, etc.)

### Libraries Configuration
The config uses YAML anchors (&) and references (*) for reusable configuration blocks:
- `&movie_config` / `&tv_config`: Library-specific settings
- `&run_order`: Shared execution order
- `&movie_collections` / `&movie_overlays`: Collection and overlay file references

## Common Development Tasks

### Validate YAML Configuration
```bash
# Check YAML syntax
yamllint config.yml
yamllint overlays/*.yml
yamllint custom/*.yml

# Or use Python
python -m yaml config.yml
```

### Run Kometa
```bash
# Test configuration without making changes
kometa --run-tests --config /path/to/config.yml

# Run specific libraries
kometa --config /path/to/config.yml --library "Movies"

# Run with specific operations
kometa --config /path/to/config.yml --operations
```

### Check Logs
```bash
# View latest log
tail -f logs/meta.log

# Check for errors
grep -i error logs/meta.log
grep -i warning logs/meta.log
```

## Important Configuration Notes

### Template Variables
- Uses `<<VARIABLE>>` syntax for environment variables (e.g., `<<PLEX_HOST>>`)
- Template variables in collection/overlay files use `<<variable_name>>` syntax
- Conditionals use nested dictionary structure with `conditions` key

### Overlay System
- **media_info.yml**: Creates overlays for resolution, video format, audio format, and editions
- **audience_rating.yml**: Shows color-coded audience ratings (green >8, yellow >6, red <6)
- Requires TRaSH naming convention for proper media detection

### Collection Files
- Uses Kometa defaults with `default:` prefix
- External templates from git repositories with `git:` prefix
- Custom collections defined in `custom/` directory
- Schedule collections using cron-like syntax (e.g., `weekly(monday)`)

### Known Issues to Check
1. **YAML Syntax**: Line 380 in config.yml has trailing space after `tmdb:`
2. **Collection Names**: Line 1395 in movie_subgenre.yml has extra quote in `Superhero"`
3. **Indentation**: Inconsistent indentation in some template blocks
4. **Playlist Settings**: Misplaced `run_order` block under `playlist_exclude_users`

## API Integrations
The configuration integrates with multiple services requiring API keys:
- **Plex**: Media server (`<<PLEX_HOST>>`, `<<PLEX_API_KEY>>`)
- **TMDb**: Movie/TV metadata (`<<TMDB_API_KEY>>`)
- **Trakt**: Collection and list data (`<<TRAKT_CLIENT_ID>>`, etc.)
- **Radarr/Sonarr**: Media management (`<<RADARR_HOST>>`, `<<SONARR_HOST>>`)
- **Tautulli**: Plex statistics (`<<TAUTULLI_HOST>>`)
- **OMDb**: Additional metadata (`<<OMDB_API_KEY>>`)
- **MDbList**: Ratings and lists (`<<MDBLIST_API_KEY>>`)

## Testing Changes
1. Always validate YAML syntax before committing
2. Test overlay changes on a small subset first
3. Use `--dry-run` or `--test` flags when available
4. Check logs for warnings about missing assets or failed operations
5. Verify template variable substitution is working correctly

## Asset Management
- Assets stored in `/assets/` with library-specific subdirectories
- Original posters backed up in `overlays/4k-Movies Original Posters/` etc.
- Overlay images in `overlays/images/` organized by type
- Font files in `overlays/fonts/` for text overlays