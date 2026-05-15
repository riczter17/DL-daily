## v4.11.2 (15 May 2026)

### Quick-Shift Buttons for Block Times (re-merged)
- Added "Move:" row inside expanded block view with four buttons: `← 1h`, `← 15m`, `15m →`, `1h →`
- Shifts both start and end times by the same delta, preserving duration
- Buttons auto-disable (greyed at 35% opacity, not-allowed cursor) when shift would push start below 00:00 or end past 24:00
- Saves to Supabase via existing block update flow; realtime sync handled
- Built for productivity gains on minute shifts to default template-generated blocks (avoids opening both time pickers)
- **Note**: This feature was originally shipped in v4.10.5 but was lost when v4.11.0 was built from an older base. Re-merged on top of v4.11.1.

---

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
