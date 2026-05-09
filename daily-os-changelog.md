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
