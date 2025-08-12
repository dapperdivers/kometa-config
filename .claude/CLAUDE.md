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

### Scheduling and Visibility
- **Critical Rule**: ONLY use official Kometa schedule options from https://kometa.wiki/en/latest/config/schedule/#important
- `schedule` option controls when Kometa processes a collection (NOT when Kometa runs)
- Collections with no `schedule` default to `daily`
- `visible_home` and `visible_shared` only work if the collection processes when Kometa runs
- **Critical Rule**: `schedule` should run more often than the `visible_*` options
- **Problem Example**: `schedule: monthly(8)` but `visible_home: weekly(tuesday)` will skip processing on most Tuesdays

#### Official Kometa Schedule Options (THESE ARE THE ONLY VALID OPTIONS)
| Type | Description | Format | Examples |
|------|-------------|--------|----------|
| **Hourly** | Update in specific hour(s) | `hourly(Hour)` or `hourly(Start-End)` | `hourly(17)`, `hourly(17-04)` |
| **Daily** | Update once daily | `daily` | `daily` |
| **Weekly** | Update on specific days | `weekly(Days)` | `weekly(sunday)`, `weekly(sunday\|tuesday)` |
| **Monthly** | Update on specific day of month | `monthly(Day)` | `monthly(1)` |
| **Yearly** | Update on specific date | `yearly(MM/DD)` | `yearly(01/30)` |
| **Date** | Update on specific date | `date(MM/DD/YYYY)` | `date(12/25/2024)` |
| **Range** | Update within date range(s) | `range(MM/DD-MM/DD)` | `range(12/01-12/31)`, `range(8/01-8/15\|9/01-9/15)` |
| **Never** | Never updates | `never` | `never` |
| **Non Existing** | Updates if doesn't exist | `non_existing` | `non_existing` |
| **All** | All conditions must be met | `all[Options]` | `all[weekly(sunday), hourly(17)]` |

#### **CRITICAL**: DO NOT USE ANY OTHER SCHEDULE OPTIONS
- ❌ **INVALID**: `quarterly`, `biweekly`, `bimonthly`, `every_other_day`, etc.
- ✅ **VALID**: Only the options listed in the table above

#### Community Best Practices:
1. **Perfect Alignment**: `schedule: weekly(tuesday)` + `visible_home: weekly(tuesday)`
2. **Multi-Day Weekly**: `schedule: daily` + `visible_home: weekly(monday)`
3. **Broad Scheduling Window**: `schedule: weekly(monday|tuesday|wednesday)` + `visible_home: weekly(tuesday)`
- **Preferred solution**: Use broader scheduling windows around visibility days to ensure collection processes when needed

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