# Daily OS Changelog

## v3.5 (21 April 2026)

### WIO Templates
- Morning commute shortened by 15 min: now **7:00-7:45am** (was 7:00-8:00am)
- Breakfast meal block moved earlier: now **7:45-8:00am** (was 8:00-8:15am)
- Work block restored to **8:00am** start (was 8:15am in v3.4)
- Applies to **NW-WIO** and **M-WIO**

### Checklist
- Removed **BP taken (3x/week)** item
- BP is now tracked solely via the weekly BP reading block in the Sunday template
- Migration removes the item from the checklist and clears it from any previously logged day data

### Migration
- Non-destructive auto-migration for existing NW-WIO and M-WIO templates
- Detects the v3.4 pattern (Commute 7-8am, Meal 8-8:15am, Work from 8:15am) and rewrites to the v3.5 pattern
- Skips if the pattern doesn't match (i.e. if templates have been manually edited)

## v3.4 (21 April 2026)

### Templates
- Added **Meal (breakfast)** block after morning commute in WIO templates
- Work block shifted to 8:15am to accommodate

### Block Energy
- Tapping the same energy score twice now **deselects** it
- Consistent with status button behaviour

## v3.3 (19 April 2026)

### Activities
- Added **BP reading** under Health

### Sunday Template
- Added **BP reading** block at 21:00 to 21:15 (15 min, Health)
- Reading block shortened from 45 min to 30 min to accommodate

### Migration
- Non-destructive auto-migration for existing Sunday templates

## v3.2 (19 April 2026)

### My Activities (Persistent Custom Activities)
- Custom activity labels auto-save to a personal activity library
- **My Activities** appears as a new group in the activity dropdown
- Picking a saved custom activity auto-sets its category
- New **My Activities** view accessible from the home screen

### Morning Energy Score
- Panel at the top of the day view, 1-5 rating
- Added to Weekly Summary with average, days logged, days at 3+, and 7-day visual strip

### Bug Fix
- Category picker now explicit for all custom/My Activities entries

## v3.1 (19 April 2026)

### Activities
- Added **Chiro** under Health

### Templates
- Added **Sleep blocks** at the start and end of every default template

### Migration
- Existing user templates auto-upgraded with Sleep blocks if missing (non-destructive)

## v3.0 (17 April 2026)

### Block Status Overhaul
- Replaced **Done / Skip** with three statuses: **Planned**, **Unplanned**, **Skipped**

### Summary Overhaul
- Key Metrics card with Adherence rate, Disruption load, and hours split
- Skipped by Category shown as separate section

### Activities
- Renamed **Family dinner** to **Family time** under Family

## v2.5 (14 April 2026)

### Day View Reorganisation
- Timeline appears first, Daily Checklist collapsible below
- Sort by time button added
- Today button shows the date
- Hybrid home screen: Today + Upcoming always visible, Past in collapsible weekly groups

### Day Type Switching
- Custom three-option modal: Retain blocks, Replace with template, Cancel

## v2.0 (14 April 2026)

### Activity System
- Grouped single dropdown replaces flat dropdown + category selector
- 31 activities across 9 categories

### Day Types
- Added: Mum WIO, Mum WFH, Workout Mum WFH
- Removed: Workout WIO, Leave, Off-site
- Renamed: Open Schedule to Custom

### Templates
- All blocks in 15-minute multiples
- All wind-down blocks set to 45 minutes
- 9 templates total

### Storage
- New storage key: dos-v2
- Auto-migrates from v1.x
