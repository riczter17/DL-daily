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
