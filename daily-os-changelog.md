## v4.11.1 (14 May 2026)

### Collapse All / Expand All for Food Categories
- New toggle link above the food list in the meal slot view
- Single button that flips between "Collapse all" and "Expand all" based on current state
- Only acts on visible categories (those with items in the current slot)
- Hidden during search (categories auto-expand when searching)
- State persists in the same `dos-collapsed-cats` localStorage key as individual toggles

### Collapsible Category State Persistence (recovered)
- Individual category collapse state now persists in localStorage via `toggleCat()` function
- Categories stay collapsed across navigation, re-renders, and page reloads
- This was originally shipped in v4.10.4 but wasn't present in the v4.11.0 base file

---
