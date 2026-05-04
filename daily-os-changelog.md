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
