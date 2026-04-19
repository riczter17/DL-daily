# Daily OS Changelog

## v3.3 (19 April 2026)

### Activities
- Added **BP reading** under Health
- Health now covers: Meditation, Workout, Couples workout, Walk, Stretch, Counselling, Medical appointment, Chiro, BP reading

### Sunday Template
- Added **BP reading** block at 21:00 to 21:15 (15 min, Health)
- Reading block shortened from 45 min to 30 min (20:30 to 21:00) to accommodate
- Wind down unchanged at 21:15 to 22:00

### Migration
- Non-destructive auto-migration for existing Sunday templates:
  - If Sunday has a Reading block ending at 21:15 and no BP reading block, splits it and inserts BP reading 21:00 to 21:15
  - Preserves customisations: if Reading block has been modified or moved, migration is skipped

## v3.2 (19 April 2026)

### My Activities (Persistent Custom Activities)
- Custom activity labels now auto-save to a personal activity library
- When you type a custom label and set its category, it's registered to **My Activities**
- **My Activities** appears as a new group in the activity dropdown once you have any saved
- Picking a saved custom activity auto-sets its category (no more inconsistent categorisation)
- Changing the category on an existing custom activity updates the saved mapping globally
- New **My Activities** view accessible from the home screen, lists all saved custom activities with their categories and allows removal
- Category picker is always visible when a block is in custom mode (label not in predefined library), even for saved My Activities

### Morning Energy Score
- New prominent panel at the top of the day view, above Timeline
- 1-5 rating for "How rested do you feel on waking?"
- Tap a number to log, tap Clear to reset
- Added to Weekly Summary:
  - **Average** (7 days)
  - **Days logged** (x/7)
  - **Days at 3 or above**
  - 7-day visual strip, colour-coded (green for 4+, orange for 3, red for below 3)

### Bug Fix
- Custom activities no longer silently inherit whatever category the block was created with
- Category picker now explicit and editable for all custom/My Activities entries

## v3.1 (19 April 2026)

### Activities
- Added **Chiro** under Health
- Activity count: 33 across 9 categories

### Templates
- Added **Sleep blocks** at the start and end of every default template
  - Start: 12:00am to wake time
  - End: wind-down end to 12:00am
- Applies to all 8 non-custom templates (NW-WIO, NW-WFH, W-WFH, M-WIO, M-WFH, WM-WFH, Saturday, Sunday)

### Migration
- Existing user templates are auto-upgraded with Sleep blocks if missing
- Non-destructive: preserves any customisations to other blocks

## v3.0 (17 April 2026)

### Block Status Overhaul
- Replaced **Done / Skip** with three statuses: **Planned** (scheduled and done), **Unplanned** (ad hoc and done), **Skipped** (scheduled but not done)
- Template blocks default to **Planned** automatically
- Manually added blocks default to **Unplanned** automatically
- User only needs to intervene to mark a block as **Skipped**
- Status icons: Planned (green tick), Unplanned (orange diamond), Skipped (skip icon), Unmarked (white square)
- Existing data migrated automatically (Done becomes Planned, Skip becomes Skipped)

### Summary Overhaul
- **Key Metrics card** at the top with:
  - **Adherence rate**: Planned / (Planned + Skipped) as headline percentage
  - **Disruption load**: Unplanned / (Planned + Unplanned) as percentage
  - Total hours split across Planned, Unplanned, and Skipped
- **Time by Category** now counts done blocks only (Planned + Unplanned)
- **Skipped by Category** shown as separate section highlighting where gaps cluster
- **Energy average** calculated from done blocks only

### Activities
- Renamed **Family dinner** to **Family time** under Family
- Added **Shower** under Personal (v2.5)
- Renamed **Grooming** to **Haircut** under Personal (v2.5)

## v2.5 (14 April 2026)

### Day View Reorganisation
- Timeline now appears first (primary purpose of the day view)
- Daily Checklist moved below timeline and collapsed by default; tap header to expand
- Day notes remain at the bottom

### Block Sorting
- Added Sort by time button in Timeline header

### Home Screen
- Today button now shows the date
- Hybrid layout: Today + Upcoming always visible, Past in collapsible weekly groups (28 days)

### Day Type Switching
- Custom three-option modal: Retain blocks, Replace with template, Cancel

## v2.0 (14 April 2026)

### Activity System
- Replaced flat activity dropdown + separate category selector with **grouped single dropdown**
- Activities auto-set their category when selected (no manual category tagging needed)
- Comprehensive activity list: 31 activities across 9 categories (Health, Work, Family, Commute, Personal, Chores, Rest, Fun, Sleep)
- All labels standardised as noun phrases
- Custom activity option retained for unlisted activities

### Day Types
- **Removed:** Workout WIO (unrealistic), Leave, Off-site
- **Added:** Mum WIO, Mum WFH, Workout Mum WFH
- **Renamed:** Open Schedule to Custom (user inputs label e.g. Leave, Sick, Off-site)
- Updated Non-workout WIO and WFH to remove flex mum time blocks (now in dedicated Mum templates)
- Added Walk to Non-workout WFH evening
- Combined Commute + Breakfast into single block for WIO templates

### Templates
- All blocks now in 15-minute multiples
- All wind-down blocks set to 45 minutes
- Sort by time button added to template editor
- Activity labels in templates show as placeholder text (not pre-filled text requiring deletion)
- 9 templates total: NW-WIO, NW-WFH, W-WFH, M-WIO, M-WFH, WM-WFH, Saturday, Sunday, Custom
- WFH templates: work 8am-5pm (9h), Mum WFH: work split across home (4h) and mum's (3h)
- Saturday: couples workout 9-10am, structured fun blocks, read block before wind-down
- Sunday: longer meditation (30 min), family time blocks, Monday prep, earlier wind-down

### Custom Day Type
- Custom days show a text input for labelling (e.g. "Leave", "Sick", "Off-site")
- Custom label displays on home screen day cards

### Data Migration
- Migrates from v1.x (dos-v1) storage key automatically
- Templates reset to new defaults on migration (old templates incompatible)
- Checklist and day data preserved

### Storage
- New storage key: dos-v2 (prevents conflicts with v1.x data)
