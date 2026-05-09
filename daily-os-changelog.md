## v4.10.0 (9 May 2026)

### Goals-Checklist Mapping
- Each checklist item can now be mapped to one primary goal (1:1) or left as Others
- Goals displayed with G-numbering throughout: G1, G2, G3... based on order_index
- Checklist items displayed with priority numbering: 1, 2, 3... based on order_index
- Note added in Edit Checklist: "Order represents priority, not when in the day to do them."

### Edit Checklist View
- Items grouped visually by goal: G1 group, G2 group, ..., Others at bottom
- Each item shows its priority number (#1, #2, etc.) as a small accent pill
- Per-item dropdown to set goal: "Others" or any of the named goals
- New items default to Others; user maps them after adding

### Day View Checklist
- Each item shows priority number prefix: "1. Meditation"
- Each item shows mapped goal as small tag on the right: G1, G2, Others, etc.
- Mapped tags use accent colour, Others uses dim style

### Home Goals Cards
- Each goal card now shows habit count: "2 habits", "1 habit", "0 habits"
- Surfaces gaps where a goal has no supporting checklist items

### Database
- Added `goal_id` column to `checklist_items` (nullable FK to goals.id, ON DELETE SET NULL)
- Index on goal_id for efficient lookups
- Initial mappings seeded based on obvious matches:
  - Meditation, 30 min unstructured time → G2 (Mental health practices)
  - Good breakfast, Good lunch, Good dinner, 8k steps → G4 (Physical health)
  - Phone down by 10pm, In bed by 10:30pm → G1 (Sleep)
- All seed mappings can be re-mapped in Edit Checklist

### Required: Run `checklist-goals-migration.sql` in Supabase SQL Editor before deploying

---

## v4.9.0 (9 May 2026)

### Hydration Tracking
- New hydration card inside Meal Tracker home, sitting alongside calories and protein as the third daily intake metric
- Cup library: each cup has a name and ml value. Tap a cup button to log a drink. Logs are timestamped.
- Seeded 7 cups: Water 200ml, Water 350ml, Coffee 250ml, Protein powder 250ml, Oatside protein 250ml, Office mug 750ml, Home mug 1.1L
- Daily target: 3500ml (editable)
- "Undo last" button to revert the most recent log without confirmation
- Progress bar with ✓ tick when target hit (no animation, no streaks, no surveillance pressure)
- ml field represents hydration contribution, not strictly volume. Most beverages count 1:1 (modern research). Set lower if you want to discount specific drinks (e.g. coffee at 80%).

### Cup Management View
- Accessed via ⚙ Manage in the hydration card
- Edit daily target
- Per cup: rename, edit ml, reorder, delete
- "+ Add cup" button
- Deleting a cup is a soft delete (is_active=false) so historical logs are preserved

### Hydration History View
- Accessed from Assessment hub
- **This Week**: 7-day strip, each day marked ✓ (hit) or — (missed). Hits/total summary below.
- **Last 30 Days**: heatmap-style 10x3 grid, days marked hit/missed. Percentage hit shown.
- **Last 12 Months**: monthly bars showing % days target hit per month. Bar colour: green (≥70%), grapemist (≥40%), grey otherwise.
- Goal-attainment focused, not amount focused. No daily totals or averages displayed in history (per design intent: consistency tracking, not optimisation).

### Database
- New `hydration_cups` table: id, user_id, name, ml, order_index, is_active, timestamps. RLS + realtime.
- New `hydration_logs` table: id, user_id, date, ml, cup_id (nullable, ON DELETE SET NULL), created_at. RLS + realtime.
- New `hydration_settings` table: user_id PK, daily_target_ml, updated_at. RLS + realtime.

### Required: Run `hydration-migration.sql` in Supabase SQL Editor before deploying

---

## v4.8.2 (9 May 2026)

### Spacing Fix
- Added 16px top margin to the Today button to give breathing room from the Goals section above it (when expanded, the Edit goals button was sitting right against Today).
- Increased Edit goals button top margin from 4px to 8px for slightly better separation from the last goal card.

---

## v4.8.1 (9 May 2026)

### Home Screen Restructure
- Meal Tracker elevated to primary CTA (visual weight equal to Today). The two daily logging actions now sit together at the top of the home screen.
- New "Assessment" hub view, accessed via secondary button. Initially contains Weekly Summary; designed to grow with monthly review, goal reflection, sleep trends, etc.
- New "Manage" view, accessed via secondary button. Consolidates Edit Templates, Edit Checklist, My Activities, DATA card (Export/Import/Reset), and Sign out (with email footer).
- Removed from home screen: standalone Edit Templates / Edit Checklist / My Activities buttons, DATA card, "Signed in as" footer, Sign out link, standalone Weekly Summary button.

### Padding Fix
- Section header padding adjusted from `10px 0` to `10px 4px` for slight inset breathing room. Applied to all three collapsible sections (Goals, Upcoming, Past) for consistency.

---

## v4.8.0 (9 May 2026)

### Goals
- New top-level "Goals" section on the home screen as an aspirational anchor
- Each goal has a name, description, and status (Active or Maintained)
- Active goals are what's prioritised this period; Maintained goals are in rhythm and don't need active surveillance
- Visual cues: filled circle (●) + grapemist accent for Active, hollow circle (○) + dim text for Maintained
- 5 goals seeded: Sleep, Mental health practices, Career evolution (active for May-June), Physical health, Stress management (maintained)

### Goals Edit View
- Accessed via "Edit goals" button inside expanded Goals section on home
- Per goal: rename, edit description, toggle Active/Maintained status, reorder up/down, delete
- "+ Add goal" button at bottom creates a new goal stub
- All edits save live to Supabase with realtime sync across devices

### Collapsible Home Sections
- Goals, Upcoming, and Past sections now collapsible via tappable headers
- Default state: all collapsed (showing summary count)
- Collapse state persists per-section in localStorage under `dos-collapse` key
- Today button stays prominent and never collapses

### Database
- New `goals` table with columns: id, user_id, name, description, status (active/maintained), order_index, timestamps
- RLS enabled with standard auth.uid() policies
- Realtime publication includes goals table for cross-device sync

### Required: Run `goals-migration.sql` in Supabase SQL Editor before deploying

---

## v4.7.0 (9 May 2026)

### Collapsible Food Categories
- Category headers in the food list are now tappable to expand/collapse
- Shows item count per category (e.g. "MAINS (8)")
- Arrow indicator: ▸ collapsed, ▾ expanded
- Collapse state persists within the session (resets on app reload)
- When searching, all categories auto-expand to show search results regardless of collapse state
- Reduces scrolling when food list grows large

---

## v4.6.3 (8 May 2026)

### Bug Fix
- Fixed deactivated food items disappearing from past meal logs. App now loads all food items (active and inactive) from the DB. Selection list still only shows active items. Past logs correctly display deactivated items.

---

## v4.6.2 (8 May 2026)

### Decimal Quantities
- Quantity +/- buttons now step by 0.5 instead of 1
- Supports half portions (e.g. 0.5 for half a plate of chicken rice)
- First tap to add an item still logs qty 1; use - button to reduce to 0.5
- Decrement to 0.5 shows remove (X) button; cannot go below 0.5
- Calorie and protein totals rounded to whole numbers for clean display
- DB column `qty` changed from integer to numeric(4,1)

### Required: Run `decimal-qty-migration.sql` in Supabase SQL Editor before deploying

---

## v4.6.1 (7 May 2026)

### Weight History Time Range Selector
- Four range buttons above chart: 3M, 6M, 1Y, All
- Default view is 3M (last 3 months) for actionable mobile-friendly view
- Chart auto-scales y-axis and x-axis labels to fit selected range
- Adaptive x-axis label density: shows every month for 3M/6M/1Y, every other month for All
- Range change indicator shows weight difference from start to end of selected period (e.g. "-1.2 kg")
- Y-axis grid step adapts to range: 1kg steps for tight ranges, 5kg for wide ranges
- Summary cards always show all-time stats; chart reflects selected range
- Fixed timezone bug in weekly aggregation within chart (was still using toISOString)

---

## v4.6.0 (7 May 2026)

### Weight History Chart
- New "Weight History" view accessible via link in weight card (visible when 2+ entries exist)
- SVG line chart showing full weight trajectory from Jan 2024 to present
- Weekly averages plotted for clean, readable line
- Gradient fill under the curve
- Month labels on x-axis with year markers on January
- Grid lines and y-axis weight labels (auto-scaled)
- Green dot marks lowest weight, accent dot marks current weight
- Summary cards: Current, Lowest, Highest
- Key dates card with exact values and dates for lowest, highest, current
- Distance from lowest and highest shown at bottom

### Sides Category
- Added "Sides" food category for items like sambal chilli, stir-fried vegetables, condiments
- Full category list now: Eggs, Mains, Sides, Carbs, Drinks, Snacks
- Existing items can be re-categorised using the edit food feature (pencil icon)

---

## v4.5.1 (7 May 2026)

### Body Composition Import
- Imported 554 historical body composition records from Omron HBF-222T scale (Jan 2024 to May 2026)
- Extended `weight_logs` table with 6 new columns: body_fat_pct, visceral_fat, skeletal_muscle_pct, resting_metabolism, bmi, body_age
- Weight card now shows body composition metrics (body fat %, muscle %, visceral fat, BMI) when data is available
- Deduplication: 5 duplicate-date entries removed during import, keeping first entry per day
- ON CONFLICT upsert: safely merges with any existing manual entries

### Required: Run `body-composition-import.sql` in Supabase SQL Editor (adds columns + imports 554 records)

---

## v4.5.0 (6 May 2026)

### Weight Logging
- Weight card on meals home screen (today view only)
- Log weight in kg with one decimal place (e.g. 78.5)
- Shows latest weight with date last logged
- When 2+ entries exist, shows change from previous entry (green for loss, red for gain)
- Edit today's entry by tapping "Edit", or log new entry via "Log today"
- First-time prompt: "Log your first weigh-in"
- Upsert logic: one entry per date, update if already logged

### Database
- New `weight_logs` table with unique constraint on (user_id, date)
- RLS enabled, realtime subscription added
- Weight data loaded alongside food items and meal logs

### Required: Run `weight-logs-migration.sql` in Supabase SQL Editor before deploying

---

## v4.4.0 (6 May 2026)

### Edit Food Items
- Pencil icon on each food item in the slot view opens edit form
- Edit name, calories, protein, and category of any food item
- Tapping the food item name/details still adds it to the meal (unchanged)
- "Remove from food list" button with confirmation step to soft-delete items
- Deactivated items hidden from food list; existing meal logs remain unaffected
- Edit state variable (`editFoodId`) tracks which item is being edited

### Technical
- New DB helpers: `updateFoodItem()` and `deactivateFoodItem()` (soft delete via `is_active` flag)
- Edit food route added to router (`edit_food` view)

---

## v4.3.1 (5 May 2026)

### Bug Fix
- Fixed timezone bug in weekly scorecard and date navigation: `toISOString()` converts to UTC, causing dates to shift back one day in UTC+8 (Singapore). Replaced with local time formatting using `getFullYear()`/`getMonth()`/`getDate()`. Weekly scorecard now correctly shows Monday to Sunday.

---

## v4.3.0 (5 May 2026)

### Previous Day Logging
- Date navigation on meals home screen with left/right arrows
- Navigate between Today, Yesterday, and 2 Days Ago
- Date label shows "Today", "Yesterday", or "2 days ago" for clarity
- Cannot navigate into the future or beyond 2 days back
- Meal slot view shows date tag when logging for a past day
- All logging, qty adjustment, and deletion work correctly for past dates

### Weekly Scorecard
- New view accessible via "Weekly Scorecard" button on meals home screen
- Shows current week (Monday to Sunday) with two compliance summary cards:
  - Calories in range (within 85%-105% of 1,800 target)
  - Protein hit (at or above 145g target)
- Daily averages for calories and protein across logged days
- Daily breakdown showing each day's totals with status indicators
- Tap any day to navigate directly to that day's meal log
- Legend: green tick = calories in range, red cross = calories off target, muscle = protein hit, warning = protein missed
- Days with no logged data shown as faded with "No data" label

### Copy Refinements
- Calorie allocation range shown per meal slot (e.g. "240-320 kcal")
- Slot totals colour-coded: green when in range, red when exceeded
- Food items grouped by category: Eggs, Mains, Carbs, Drinks, Snacks
- Category selector added to "Add new food item" form
- Rice labels standardised: "White Rice - Half", "Black Rice - Half", etc.
- Location labels standardised with pipe + caps: "Item Name | LOCATION"
- All food items set to slot "any" (no meal restrictions)
- Protein nudge only appears for today, not past days
- Placeholder in add food form updated to match label convention (e.g. "Grilled Salmon | ICHIBAN")

### Bug Fixes
- Fixed seed duplication: now checks DB count directly before seeding
- Fixed slot filtering: all food items appear in every meal slot
- Fixed qty controls using wrong date reference for past day logs
- Fixed add food form losing input on mobile screen switch (form state now persists in JS variables)

### Required: Run `cleanup-duplicates.sql` in Supabase SQL Editor (adds `cat` column, deduplicates, renames labels, sets categories)

---

## v4.2.0 (4 May 2026)

### Meal Tracker
- New standalone meal tracking feature accessible from home screen via "Meal Tracker" button
- Dashboard view with real-time calorie and protein progress bars against daily targets (1,800 kcal / 145g protein)
- Four meal slots: Breakfast, Lunch, Snack, Dinner displayed as tappable cards with subtotals
- Per-unit food item model with adjustable quantity (no hardcoded combos)
- Tap a food item to add it to the current slot; tap again to increment quantity
- Quantity controls (+/-) on each logged item; decrement to 1 then X to remove
- Smart nudge: time-aware protein recommendation when daily target has remaining gap
- Search/filter within each slot's food list
- Add new food items on the fly (name, calories, protein, meal slot) saved to Supabase for future selection
- Food items tagged by slot (breakfast/lunch/snack/dinner/any) for filtered display

### Seed Data
- 26 pre-loaded food items across proteins, carbs, drinks, shakes, and snacks
- Based on curated meal plan (Ya Kun, Aston's, Mdm Yap's, Subway, Thien Kee, Raden Lina, Pung Pung Kitchen, Cold Storage/FairPrice, Donki, home-cooked options)
- Auto-seeded on first load; new items added by user persist alongside seed data

### Database
- 2 new Supabase tables: `food_items` (library) and `meal_logs` (daily entries)
- RLS enabled on both tables
- Realtime subscriptions for cross-device sync
- localStorage cache under `dos-meals` key for offline-first rendering

### Technical
- Follows existing app patterns: noteLocalWrite, debounced realtime reload, localStorage cache + background fetch
- Meal data loaded in parallel with main data during auth flow
- Sign out clears meal cache alongside main cache

### Required: Run `meal-tracker-migration.sql` in Supabase SQL Editor before deploying
