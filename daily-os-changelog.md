## v4.13.0 (21 May 2026)

### Duration preset chips in expanded block view
A new "Duration:" row sits directly below the "Move:" row. Six preset chips snap the block to a fixed duration while keeping the start time pinned:

- `15m | 30m | 45m | 1h | 1h 30m | 2h`

**Behaviour:**
- Tapping a chip sets `end = start + duration`. Start time does not move.
- The chip matching the current duration is highlighted with the accent style (same treatment as an active Energy button).
- Any chip that would push the end past midnight is disabled (greyed out).
- Durations beyond 2h still use the existing end-time picker.

**Why this matters:** the Move buttons shift the whole block; this row resizes it. Common short-to-medium block durations are now a single tap.

### Templates: Together time consolidated into Wife time
"Together time" was a duplicate label that only existed in templates, with no matching entry in the My Activities library. Three templates now use the canonical "Wife time" activity instead:

- `nw_wio` (8:30pm-9:30pm)
- `nw_wfh` (8:30pm-9:30pm)
- `nw_wfh_w` (8:30pm-9:30pm)

Label changed: `Together time` → `Wife time`. Category changed: `rest` → `family`.

The "Walk / Together activity" entry in `nw_wfh_w` (6:15pm-7pm, health) was left untouched, as it's a distinct walking activity.

Templates only affect future applications. Existing days already logged with "Together time" blocks are not retroactively changed.

**Heads-up:** the Saturday template's "Wife time" block (10:15am-12:00pm) still has `cat:"rest"` rather than `cat:"family"`. This predates v4.13.0 and was not in scope for this change. Flag if you want it aligned.

---

## v4.12.4 (15 May 2026)

### Dashboard: Floor vs Ceiling Logic
The protein bar and calorie bar now behave differently to match how each metric should actually be read.

**Protein (floor metric — under is bad, over is achievement):**
- Below target: accent colour fills as you go, "Xg remaining"
- At target: green bar with "✓ Target hit"
- Over target: green bar with "✓ +Xg over target" (celebrate, not warn)
- Flame icon also turns green when target is hit/exceeded

**Calories (ceiling metric — under is fine, over is bad):**
- Below target: muted fill, "X kcal remaining"
- At target: green
- Over target: red bar with "X kcal over" (warning, unchanged from before)

**Why this matters:** Hitting 151g protein on a 145g target should feel like a win, not a warning. The UI now reflects that.

---
