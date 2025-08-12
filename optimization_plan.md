Database locking issues with Kometa and Plex are commonly caused by too many rapid API calls or concurrent operations. Here are the key configuration options you should adjust in your Kometa config to resolve this:

## Rate Limiting and Request Management

**In your `config.yml`, add or modify these settings:**

```yaml
plex:
  url: YOUR_PLEX_URL
  token: YOUR_PLEX_TOKEN
  timeout: 60
  clean_bundles: false
  empty_trash: false
  optimize: false
```

## Critical Performance Settings

**Add these under the `settings` section:**

```yaml
settings:
  cache: true
  cache_expiration: 60
  asset_depth: 0
  asset_folders: false
  asset_directory: 
  create_asset_folders: false
  dimensional_asset_rename: false
  download_url_assets: false
  show_missing_assets: false
  show_missing_season_assets: false
  show_missing_episode_assets: false
  show_asset_not_needed: false
  sync_mode: append
  minimum_items: 1
  default_collection_order:
  delete_below_minimum: false
  delete_not_scheduled: false
  run_again_delay: 2
  missing_only_released: false
  only_filter_missing: false
  show_unmanaged: false
  show_filtered: false
  show_options: false
  show_missing: false
  show_missing_assets: false
  save_report: false
  tvdb_language: eng
  ignore_ids:
  ignore_imdb_ids:
  item_refresh_delay: 0
  playlist_sync_to_users: all
  playlist_exclude_users:
  playlist_report: false
  verify_ssl: true
  custom_repo:
  check_nightly: false
```

## Most Important Adjustments

**Focus on these specific settings that directly impact database locking:**

1. **`run_again_delay: 2`** - Increase this to 5-10 seconds to slow down operations between runs

2. **`item_refresh_delay: 0`** - Set this to 1-3 seconds to add delay between item refreshes

3. **Add rate limiting to operations:**
```yaml
operations:
  delete_collections: false
  delete_labels: false
  mass_genre_update: false
  mass_audience_rating_update: false
  mass_critic_rating_update: false
  mass_content_rating_update: false
  mass_originally_available_update: false
  mass_imdb_parental_labels: false
  mass_poster_update: false
  mass_background_update: false
  mass_collection_mode: false
  update_blank_track_titles: false
  remove_title_parentheses: false
  split_duplicates: false
  radarr_add_all_existing: false
  radarr_remove_by_tag: false
  sonarr_add_all_existing: false
  sonarr_remove_by_tag: false
```

4. **Disable or limit overlay operations** if you're using them:
```yaml
overlays:
  reapply_overlays: false
  remove_overlays: false
```

5. **Run operations sequentially** rather than in parallel by setting:
```yaml
libraries:
  Movies:
    run_order:
      - operations
      - metadata
      - collections
      - overlays
    minimum_items: 5  # Increase minimum items for collections
```

## Additional Recommendations

- **Schedule runs during off-peak hours** when Plex isn't actively being used
- **Run Kometa less frequently** - instead of hourly, try every 6-12 hours
- **Disable Plex's scheduled tasks** during Kometa runs (Scheduled Tasks in Plex settings)
- **Increase Plex database cache** if possible (requires Plex Pass)
- **Run one library at a time** rather than all libraries simultaneously

## If issues persist:

1. Check your Plex database integrity:
   - Stop Plex
   - Back up your database
   - Run database repair/optimization
   
2. Consider using the `test` mode first:
```yaml
settings:
  test_mode: true
```

3. Enable debug logging to identify specific operations causing locks:
```yaml
settings:
  log_level: DEBUG
```

Try implementing these changes gradually, starting with the rate limiting settings, and monitor which adjustments provide the most improvement for your specific setup.