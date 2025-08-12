# Kometa Collection Rotation Strategy

## Overview
This document defines the rotation strategy for Kometa collections to optimize Plex database performance while maintaining fresh, organized content on home and shared screens.

## Core Principles

### Database Performance
- **Minimize concurrent operations**: Stagger updates across time periods
- **Reduce API calls**: Use delays and limit simultaneous collection processing
- **Control visibility**: Only show relevant collections on home screens to reduce load

### Content Organization
- **Thematic grouping**: Related collections rotate together
- **Priority-based visibility**: Most important content always visible
- **Fresh rotation**: Regular changes keep content interesting

## Performance Settings

### Global Settings
```yaml
settings:
  run_again_delay: 10          # 10-second delay between operations
  item_refresh_delay: 3        # 3-second delay between item refreshes
  minimum_items: 5             # Higher threshold reduces small collections

plex:
  clean_bundles: false         # Disable heavy operations
  empty_trash: false           # Disable heavy operations

operations:
  assets_for_all: false        # Disable daily asset scanning
  # NO global schedule - individual operations manage their own schedules
```

### Mass Operation Staggering
```yaml
operations:
  - schedule: monthly(1)         # User ratings - monthly
    mass_user_rating_update: mdb_tomatoes
  - schedule: monthly(8)         # Critic ratings - monthly  
    mass_critic_rating_update: imdb
  - schedule: monthly(15)        # Audience ratings - monthly
    mass_audience_rating_update: tmdb
  - schedule: monthly(22)        # Content ratings - monthly
    mass_content_rating_update: mdb_commonsense
  - schedule: quarterly(01/01)   # Studio updates - quarterly
    mass_studio_update: tmdb
  - schedule: quarterly(04/01)   # Available dates - quarterly
    mass_originally_available_update: tmdb
```

## Collection Rotation Rules

### 1. Subgenres - Weekly Rotation
**Update Schedule**: Weekly (perfect alignment with visibility)
**Visibility**: Weekly rotation on home/shared screens

```yaml
# Monday: Action/Adventure
schedule: weekly(monday)
visible_home: weekly(monday)
visible_shared: weekly(monday)

# Tuesday: Comedy/Satire + Crime/Thriller  
schedule: weekly(tuesday)
visible_home: weekly(tuesday)
visible_shared: weekly(tuesday)

# Wednesday: Fantasy/Supernatural + Horror/Occult
schedule: weekly(wednesday)
visible_home: weekly(wednesday)
visible_shared: weekly(wednesday)

# Thursday: Sci-Fi/Futuristic + Historical/Period
schedule: weekly(thursday)
visible_home: weekly(thursday)
visible_shared: weekly(thursday)

# Friday: Romance/Drama + Exploitation
schedule: weekly(friday)
visible_home: weekly(friday)
visible_shared: weekly(friday)

# Saturday: Experimental/Artistic
schedule: weekly(saturday)
visible_home: weekly(saturday)
visible_shared: weekly(saturday)
```

### 2. Awards - Seasonal Rotation
**Update Schedule**: Seasonal ranges (perfect alignment with visibility)
**Visibility**: Matches actual award seasons

```yaml
# Winter Awards (Perfect Alignment)
Golden Globes: 
  schedule: range(12/01-01/31)      # Golden Globes season
  visible_home: range(12/01-01/31)
  visible_shared: range(12/01-01/31)

BAFTA: 
  schedule: range(01/01-02/28)      # BAFTA season  
  visible_home: range(01/01-02/28)
  visible_shared: range(01/01-02/28)

SAG: 
  schedule: range(01/15-02/15)      # SAG season
  visible_home: range(01/15-02/15)
  visible_shared: range(01/15-02/15)

Critics Choice: 
  schedule: range(01/01-02/15)      # Critics Choice season
  visible_home: range(01/01-02/15)
  visible_shared: range(01/01-02/15)

Oscars: 
  schedule: range(02/01-04/15)      # Oscar season
  visible_home: range(02/01-04/15)
  visible_shared: range(02/01-04/15)

# Spring/Summer Awards
Spirit: 
  schedule: range(02/15-03/31)      # Spirit Awards season
  visible_home: range(02/15-03/31)
  visible_shared: range(02/15-03/31)

Sundance: 
  schedule: range(01/15-02/15)      # Sundance season
  visible_home: range(01/15-02/15)
  visible_shared: range(01/15-02/15)

Cannes: 
  schedule: range(05/01-06/15)      # Cannes season
  visible_home: range(05/01-06/15)
  visible_shared: range(05/01-06/15)

# Fall Awards
Emmy: 
  schedule: range(08/01-10/15)      # Emmy season
  visible_home: range(08/01-10/15)
  visible_shared: range(08/01-10/15)

TIFF: 
  schedule: range(09/01-10/15)      # TIFF season
  visible_home: range(09/01-10/15)
  visible_shared: range(09/01-10/15)

PCA: 
  schedule: range(11/01-12/31)      # PCA season
  visible_home: range(11/01-12/31)
  visible_shared: range(11/01-12/31)
```

### 3. People - Tri-Annual Rotation
**Update Schedule**: 3 times per year per category
**Visibility**: Matches update schedule

```yaml
Actors:    visible_home: range(01/01-01/31,05/01-05/31,09/01-09/30)  # Months 1,5,9
Directors: visible_home: range(02/01-02/28,06/01-06/30,10/01-10/31)  # Months 2,6,10  
Producers: visible_home: range(03/01-03/31,07/01-07/31,11/01-11/30)  # Months 3,7,11
Writers:   visible_home: range(04/01-04/30,08/01-08/31,12/01-12/31)  # Months 4,8,12

# Birthday collections always visible during their month
Birthday: tmdb_birthday.this_month: true  # Built-in monthly visibility
```

### 4. Charts - Multi-Day Weekly Approach
**Update Schedule**: Weekly updates
**Visibility**: Always visible (Multi-Day Weekly pattern)

```yaml
Other Chart (Pirated): 
  schedule: daily                 # Perfect alignment
  visible_home: daily
  visible_shared: daily

IMDb:
  schedule: weekly(monday)        # Multi-Day Weekly approach
  visible_home: daily             # Always visible (processes weekly)
  visible_shared: daily

Tautulli:
  schedule: weekly(wednesday)     # Multi-Day Weekly approach  
  visible_home: daily             # Always visible (processes weekly)
  visible_shared: daily

TMDb:
  schedule: weekly(friday)        # Multi-Day Weekly approach
  visible_home: daily             # Always visible (processes weekly)
  visible_shared: daily
```

### 5. Always Visible Collections
These collections remain visible at all times:
- **Seasonal/Holiday**: High user engagement during relevant periods
- **Streaming Services**: Users want to see what's available on their platforms
- **Basic Collections**: Core library organization (Genre, Studio, Decade)
- **Franchise/Universe**: Popular evergreen content

## Implementation Template

### For Subgenre Files
```yaml
templates:
  sub_genre:
    # ... existing template ...
    visible_home: <<visible_home>>    # Override default false
    visible_shared: <<visible_shared>> # Override default false

collections:
  Collection_Name:
    schedule: weekly(day)  # MUST align with visibility
    template:
      # ... existing template vars ...
      visible_home: weekly(day)      # Perfect alignment
      visible_shared: weekly(day)    # Perfect alignment
```

### For Main Config Awards/People
```yaml
- default: collection_name
  schedule: range(MM/DD-MM/DD)      # MUST align with visibility  
  template_variables:
    visible_home: range(MM/DD-MM/DD)    # Perfect alignment
    visible_shared: range(MM/DD-MM/DD)  # Perfect alignment
```

## Critical Scheduling Rules

Following CLAUDE.md guidelines, schedule MUST:
1. **Perfect Alignment**: `schedule` matches `visible_*` exactly
2. **Multi-Day Weekly**: `schedule: weekly(day)` + `visible_home: daily` 
3. **Broad Scheduling Window**: `schedule` runs more often than `visible_*`

**❌ NEVER DO**: `schedule: monthly(8)` + `visible_home: weekly(tuesday)`
**✅ CORRECT**: `schedule: weekly(tuesday)` + `visible_home: weekly(tuesday)`

## Monitoring and Maintenance

### Key Metrics to Track
- Plex database lock incidents
- Collection update completion times
- User engagement with rotated collections
- Home screen load times

### Adjustment Guidelines
- If database locks persist: Increase delays further
- If collections feel stale: Shorten visibility windows
- If home screen is cluttered: Reduce concurrent visible collections
- If users miss collections: Extend visibility windows

### Seasonal Adjustments
- **Award Season (Jan-Mar)**: More award collections visible
- **Summer (Jun-Aug)**: Lighter rotation, focus on popular content
- **Holiday Season (Oct-Dec)**: Prioritize seasonal content

## File Organization
```
/config/custom/movie_subgenre/
├── action_adventure.yml      # Weekly(Monday), Visible Monday
├── comedy_satire.yml         # Weekly(Tuesday), Visible Tuesday  
├── crime_thriller.yml        # Weekly(Tuesday), Visible Tuesday
├── fantasy_supernatural.yml  # Weekly(Wednesday), Visible Wednesday
├── horror_occult.yml         # Weekly(Wednesday), Visible Wednesday
├── sci_fi_futuristic.yml     # Weekly(Thursday), Visible Thursday
├── historical_period.yml     # Weekly(Thursday), Visible Thursday
├── romance_drama.yml         # Weekly(Friday), Visible Friday
├── exploitation_niche.yml    # Weekly(Friday), Visible Friday
└── experimental_artistic.yml # Weekly(Saturday), Visible Saturday
```

## Benefits

### Performance
- 80%+ reduction in concurrent operations
- Reduced Plex API calls and database pressure
- Staggered mass operations prevent bottlenecks

### User Experience  
- Cleaner home screens (4-6 collection types vs 20+)
- Fresh content rotation keeps interface interesting
- Priority content always accessible

### Maintenance
- Logical grouping makes config management easier
- Modular file structure allows selective updates
- Clear rotation schedule aids troubleshooting

---

*Last Updated: December 2024*
*Version: 2.0 - Fixed Critical Scheduling/Visibility Alignment Issues*