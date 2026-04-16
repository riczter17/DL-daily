# Daily OS Changelog

## v2.5 (14 April 2026)

### Day View Reorganisation
- **Timeline now appears first** (primary purpose of the day view)
- **Daily Checklist moved below timeline** and collapsed by default; tap header to expand, header shows checked count (e.g. "3/5")
- **Day notes remain at the bottom**

### Block Sorting
- Added **Sort by time** button in the Timeline header (appears when there's more than one block)
- Reorders blocks chronologically by start time

### Home Screen
- **Today button now shows the date** (e.g. "Today · Thu, 16 Apr · 2/5 checked")
- Start Today button also shows the date when no day has been started yet

### Day Type Switching
- Custom three-option modal: **Retain blocks**, **Replace with template**, **Cancel**
- Tapping outside the modal cancels

### Home Screen Layout
- Hybrid layout: **Today** button always at top, **Upcoming** (7 days) always visible, **Past** days in collapsible weekly groups
- Past days grouped by week (Monday to Sunday), collapsed by default
- Each week header shows date range and logged day count
- Past history extended to 28 days (4 weeks)

### Activities
- Added **Shower** under Personal
- Renamed **Grooming** to **Haircut** under Personal
- Activity count: 32 across 9 categories

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
