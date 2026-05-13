## v4.10.5 (13 May 2026)

### Quick-Shift Buttons for Block Times
- Added "Move:" row inside expanded block view with four buttons: `← 1h`, `← 15m`, `15m →`, `1h →`
- Shifts both start and end times by the same delta, preserving duration
- Buttons auto-disable (greyed at 35% opacity) when shift would push start below 00:00 or end past 24:00
- Saves to Supabase via existing block update flow; realtime sync handled
- Built for productivity gains on minute shifts to default template-generated blocks (avoids opening both time pickers)

---

## v4.10.4 (11 May 2026)

### Bug Fix
- Fixed collapsible food category state resetting on page reload or mobile app switch. Collapse/expand state now persisted in localStorage under `dos-collapsed-cats` key via `toggleCat()` function. Categories stay collapsed across navigation, re-renders, and page reloads.

---

## v4.10.3 (11 May 2026)

### Template Fix: Prep Before Meditation
- Six templates corrected so Prep comes before Meditation in the morning sequence. Meditation immediately after waking was the previous default in these templates.

**Workout templates** (Option A: brief Prep → Workout → full Prep → Meditation):
- **nw_wfh**: Sleep → Prep 15min → Workout 45min → Prep 30min → Meditation 15min → ...
- **w_mum_wfh**: Sleep → Prep 15min → Workout 30min → Prep 30min → Meditation 15min → ...

**Simple swap templates**:
- **mum_wio**: Sleep → Prep 45min → Meditation 15min → Commute → ...
- **mum_wfh**: Sleep → Prep 30min → Meditation 15min → Chores 45min → ...
- **saturday**: Sleep → Prep 30min → Meditation 15min → Chores → ...
- **sunday**: Sleep → Prep 45min → Meditation 30min → Reading → ...

### Note
- This is a default change only. Existing user templates in Supabase keep the old order until you do Reset Templates from Edit Templates view.

---

## v4.10.2 (10 May 2026)

### Hydration Entry Editing
- Tap the ml value on any expanded hydration log entry to edit it inline
- Editable input field with save (tick) and cancel (X) buttons
- Updates the ml value directly in the database without needing to delete and re-add
- ml values now shown in accent colour to indicate they are tappable/editable

---
