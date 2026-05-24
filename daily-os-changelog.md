## v4.14.1 (24 May 2026)

### Daily reflection collapsible
The Daily reflection section on the day view now follows the same collapse pattern as Goals, Upcoming, and Past sections. Tap the header to expand or collapse.

**Default state:** collapsed.

**State persistence:** localStorage via the existing `collapseState` mechanism, alongside the other three sections.

**Header hint:** the right side of the header shows "Not started" or "N fields filled" so you can see at a glance whether anything has been captured without expanding.

**Why this matters:** the section was always visible and added vertical length to every day view, even on days when reflection isn't the priority. Now it's tucked away until summoned.

### Recent weight entries
A new "Recent entries" card has been added to the Weight History view, sitting between the chart and the Key dates card. Shows the last 10 weight logs in reverse chronological order (most recent first).

**Each row shows:**
- **Date** of the log
- **Weight** in kg
- **Delta from the previous entry** in the list, colour-coded:
  - Green for a decrease
  - Red for an increase
  - Dim dash for unchanged (within 0.05 kg)
- The oldest entry in the list of 10 shows no delta (nothing earlier to compare against in the visible list)

**Why this matters:** the chart and stats are good for trend understanding across months, but they don't help when you want to see the most recent few logs at a glance. This card fills that gap.

---

## v4.14.0 (22 May 2026)

### Daily reflection replaces Day notes
The free-form "Day notes" textarea is replaced with a structured reflection section, drawn from the May 2026 tarot reading's "force focus on positives" practice. Existing day notes content is preserved as free-form notes at the bottom of the new section.

**New structure:**
- **Three wins (any size):** three single-line inputs for things that went well today, small or large
- **Reframe:** two fields — a negative thought from today, and what's also true alongside it
- **Gratitude (optional):** a one-line entry
- **Notes:** the original free-form textarea, preserved at the bottom

**Storage:** structured fields are encoded as JSON inside the existing `notes` column (no Supabase schema change required). If any structured field is filled, the column holds JSON; if everything is empty except the free-form field, the column reverts to plain text. Legacy plain-text content is auto-migrated into the free-form notes field on first read.

### Activity reminders
Selected activities now carry a contextual reminder that appears in the expanded block view, drawn from the May 2026 tarot reading. The reminder shows as a single italic line with a left accent border, directly under the block header. Passive display only, no interaction needed.

| Activity | Reminder |
|----------|----------|
| Meditation | Today's focus: surrender. |
| Walk | Walk with intention. Release burdens to nature. |
| Parents time | Focus on her happiness, not improvement. |
| Wind down | Real rest is no thinking. Notice rumination. |
| Rest | Real rest is no thinking. Notice rumination. |

Reminders are baked into the predefined activity definitions. Custom activities do not carry reminders in this version. If a reminder needs editing, change the `r:` field on the relevant entry in the `ACTIVITIES` array.

### Manual setup: coaching exploration goal
This is not a code change. Add the following yourself via Home → Goals → Edit goals:

**Goal name:** Explore coaching/counselling path

**Description (paste into the goal's description field):**
> Starter actions:
> - Reach out to Chris, Shirleen, or Seto about restructuring sessions around coaching exploration
> - Read about ICF certification requirements
> - Identify three practising coaches for informational chats
>
> Later actions:
> - Listen to one coaching podcast a week
> - Attend a coaching webinar or workshop
> - Journal about what draws me to coaching specifically
> - Try a coaching session as a client
> - Identify a coaching specialty interest (executive, life, career, counselling)

Goals are user data stored per-account in Supabase, so they can't be seeded via the HTML file. The setup is one-time.

---

## v4.13.1 (21 May 2026)

### Weekday template overhaul: later wake, evening meditation
All six weekday templates updated. Two themes: wake 15 min later, meditation moved from morning to evening (where a Cool down already existed).

**Wake time: 6:00am → 6:15am** in five of six weekday templates. `nw_wfh_w` stays at 6:30am (the WFH-with-wife day with the 7am market walk). The 15 min comes from the deleted morning Meditation block; everything else after wake stays put. Work still starts at 8:00am across the board.

**Workout templates (`nw_wfh`, `w_mum_wfh`):** workout slides 15 min later (now 6:30am start, previously 6:15am) to preserve a 15-min pre-workout Prep. Post-workout flow unchanged.

**Evening meditation** replaces the existing Cool down block in the three templates that had one:

| Template | Slot | Notes |
|----------|------|-------|
| `nw_wio` | 6:30-6:45pm | Direct swap |
| `nw_wfh` | 6:00-6:15pm (15 min) | Was a 30-min Cool down. Meditation kept at 15 min; the freed 15 min extended the preceding evening Walk from 45 to 60 min. |
| `nw_wfh_w` | 6:00-6:15pm | Direct swap |

Templates without a cool down (`mum_wio`, `mum_wfh`, `w_mum_wfh`) do not get an evening meditation slot, by design.

Category changed across all three: `rest` → `health`.

**Untouched:** Saturday and Sunday templates retain their morning meditation slots.

**Existing days unaffected.** Template edits only apply to future template applications. Logged days with the old structure are not retroactively changed.

---

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
