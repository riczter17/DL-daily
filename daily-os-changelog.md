## v4.12.0 (15 May 2026)

### Quick Picks for Meal Slots
Curated short list of go-to foods per meal slot, shown at the top of each slot view for faster logging.

**Behaviour:**
- New "Quick Picks" section sits between the "Add item" header and the search bar
- Dashed accent border + star icon makes it visually distinct
- Items shown are filtered to the current slot (breakfast / lunch / snack / dinner)
- Tap to log directly, same as the categorised list
- Collapsible per slot, state persists in localStorage key `dos-qp-collapsed`
- Hidden during search so search results take the full screen
- Hidden when the slot has no quick picks flagged

**Flagging items as quick picks:**
- Edit pencil on any food item now has 4 toggles: "Breakfast / Lunch / Snack / Dinner"
- An item can be flagged for multiple slots (e.g. Oatside in both Breakfast and Snack)
- When adding a new food item, the current slot is pre-ticked
- Visual: ticked slots show in accent colour with accent border

**Data model:**
- New `quick_pick_slots` TEXT[] column on `food_items` table
- Defaults to empty array — no items pre-flagged
- You flag manually after deploying

### SQL Migration Required

Run `daily-os-quick-picks.sql` in Supabase SQL editor before deploying:

```sql
ALTER TABLE food_items 
ADD COLUMN IF NOT EXISTS quick_pick_slots TEXT[] DEFAULT '{}';
```

Safe to run multiple times.

---
