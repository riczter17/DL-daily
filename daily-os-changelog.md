## v4.11.0 (14 May 2026)

### Food Category Restructure
Split the catch-all "Mains" category into protein-specific categories for easier meal-time browsing.

**New category order:**
- Eggs
- Chicken
- Fish & Seafood
- Other Meats
- Soups
- Sides
- Carbs
- Drinks
- Snacks

**Migration behaviour:**
- One-time migration runs automatically on first app load after this version
- Existing items in "Mains" are re-categorised based on name keywords:
  - "soup", "bak kut teh", "samgyetang" → Soups
  - Fish/seafood keywords (salmon, tuna, sashimi, prawn, ebi, unagi, fish maw, etc.) → Fish & Seafood
  - Chicken keywords (chicken, char siew, tori, rendang, karaage) → Chicken
  - Beef/pork/lamb/tripe keywords → Other Meats
  - Unmatched items default to Other Meats
- Existing items in Eggs/Sides/Carbs/Drinks/Snacks stay untouched
- Migration flag persisted in localStorage (`dos-cat-migrated-v1`) so it runs once per device
- Historical meal logs are unaffected (they reference food_item_id, not category)
- Any item that lands in the wrong category can be moved via the edit pencil

---
