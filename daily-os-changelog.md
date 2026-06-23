## v4.16.4 (23 June 2026)

### Hotfix: stop duplicate blocks recurring after reset/replace

Fixes day blocks duplicating and reappearing after applying or replacing a day's template, and persisting even after manually resetting and replacing the blocks.

**Root cause:** applying or replacing a day's template wrote blocks by differencing against the ids held in memory. `applyDayType` collected the `_id`s of the blocks currently in memory, deleted exactly those, then inserted the template's blocks one at a time. Any block row that existed in the database for that date but was not in the in-memory list was never deleted, so it survived every replace and returned on the next load (`loadDataFromDb` reads every row for the date). Three things let orphan rows accumulate in the first place: no uniqueness on the blocks table, no guard against `applyDayType` running twice (a double tap or a re-fired modal ran the delete/insert loop twice, interleaved), and the per-block insert loop writing each new id back one at a time, which a realtime reload could interrupt by replacing the whole `data` object mid-loop and detaching the objects the loop was still updating.

**Fixes:**
- `applyDayType` now writes a day's blocks authoritatively: it deletes every block for that (user, date), then inserts the new set, mirroring how `saveTemplateToDb` already writes template blocks. Because it deletes by date rather than by known id, each replace also clears any orphan or duplicate rows for that day, so using the app normally heals duplicated days.
- The per-block insert loop is replaced by a single bulk insert (`insertBlocksToDb`), with the returned ids mapped back onto the local blocks by order. One request instead of 15 to 20, so replace is noticeably faster.
- Added a re-entrancy guard (`dayBlocksWriting`) so a double-fired apply cannot run the write twice, and a `bulkOpInProgress` flag that suppresses the realtime reload while a bulk write runs, so state can no longer be swapped mid-write.
- `applyDayType` now captures the date at call time, so navigating to another day during the async write can no longer write blocks to the wrong date.

**Notes:**
- This stops new duplicates and self-heals any day you apply or replace a template on. It does not retroactively sweep days you never touch. If a specific day stays duplicated and you would rather not re-apply its template by hand, a one-time cleanup pass (delete duplicates keeping one per logical block, with a log-first safety net) can be added separately.
- New helpers: `deleteBlocksForDateFromDb`, `insertBlocksToDb`. The single-row `insertBlockToDb` is retained for the manual "Add block" path.

---

## v4.16.3 (12 June 2026)

### Duration control: relative adjust buttons replace fixed presets

The Duration row in the expanded block view now mirrors the Move row: instead of snapping to fixed presets, it grows or shrinks the block in relative steps.

**Change:**
- The Duration preset chips (15m / 30m / 45m / 1h / 1h 30m / 2h) are replaced with four relative buttons: `− 1h | − 15m | + 15m | + 1h`.
- Start time stays pinned; each button moves the end time by the step, resizing the block. This is the Duration counterpart to the Move row, which shifts the whole block.
- The current duration still shows next to the end time, so the removed preset highlight is no loss.

**Bounds (buttons disable at the edges, with a safety net in the handler):**
- Grow can't pass midnight: `+15m` disables when the block already ends at midnight; `+1h` disables when it ends after 23:00.
- Shrink can't drop below a 15-minute block: `−15m` disables when the block is already 15 min; `−1h` disables when it is 60 min or shorter.

**Code:** `setDur(i, mins)` (absolute, preset-driven) is replaced by `adjDur(i, delta)` (relative). No other callers existed.

---

## v4.16.2 (9 June 2026)

### Hotfix: paginate database reads (1000-row cap was silently dropping blocks)

Fixes work blocks (and any other data past the first 1000 rows) disappearing from the app despite being safely stored in Supabase.

**Root cause:** the app loaded each table with a single unpaginated `select('*')`. Supabase caps a single read at 1000 rows by default. Once the `blocks` table grew past 1000 (confirmed at 1086), every load silently returned only the first 1000 rows and dropped the rest. The overflow happened to include the current day's most-recently-inserted work blocks, so they never reached the app's memory and never rendered, on every hard refresh, tab switch and token refresh. The data was never deleted; it just wasn't being read back. This was the true cause behind the "work blocks keep getting deleted" reports; the earlier empty-template and `day_types` issues were separate problems layered on top.

**Fixes:**
- Added a `fetchAllRows` helper that pages through a table in 1000-row batches until every row is fetched.
- `loadDataFromDb` now paginates the tables that grow with daily use: `blocks`, `days`, `day_checklist`. Bounded tables (templates, template_blocks, activities, checklist items, goals) stay as single reads.
- `loadMealDataFromDb` now paginates `food_items`, `meal_logs`, `weight_logs` and `hydration_logs` for the same reason, with their existing ordering preserved. `hydration_cups` and `hydration_settings` stay as single reads (bounded / single-row).

**Notes:**
- Duplicate blocks were cleaned up separately in the database; pagination alone would have displayed them all regardless.
- Each load now makes one extra request per paginated table only when that table exceeds 1000 rows (e.g. blocks currently needs two requests). Negligible for personal use. If load time ever becomes a concern at much larger volumes, scoping the block read to a date range is an option.

---

## v4.16.1 (8 June 2026)

### Hotfix: stop templates and work blocks being wiped (empty-category data loss)

Fixes the bug where several templates dropped to 0 blocks on their own (WIO, WFH, WFH with wife, Saturday, Sunday) while the Mum templates survived, and where work blocks were deleted from the current day and prior days even after being manually recreated.

**Root cause:** the v4.16.0 migration converted removed blocks (Self-time/Hobby, Meditation, evening Wife time, Reading, Leisure, Personal project) into open slots carrying an empty category (`cat:""`). The Supabase write paths delete before they insert with no rollback:
- `saveTemplateToDb` deletes all of a template's blocks, then inserts the new ones.
- `applyDayType` deletes a day's existing blocks, then inserts the template's blocks.

The empty category was rejected on insert, so after the delete succeeded the insert failed and nothing was written back, leaving the template or day empty. Only templates containing the removed labels carried empty-category slots, which is why the Mum templates (no such labels) were untouched.

**Fixes:**
- Open slots now carry the `personal` category instead of an empty string. Open-ness is still determined by the empty label, so they display and behave as open time. `migrateTpl416` emits `cat:"personal"` for the three open-slot conversions.
- All three database write points (`insertBlockToDb`, `updateBlockInDb`, `saveTemplateToDb`) sanitise the category with `b.cat||'personal'`, so no empty category can reach Supabase even from older local data. Every template and day-block insert now succeeds, which removes the failure that caused the wipe.
- Open (empty-label) and category-less blocks are now excluded from "Time by Category" and "Skipped by Category", so open time is never read as a signal in reviews, and the analysis view can no longer crash on a category-less block.

**Restoring the wiped templates:** after deploying this build, tapping Reset All Templates is safe and rebuilds every template from defaults. Note this overwrites all templates with current defaults, including the three surviving Mum templates.

**Not addressed (by design for this hotfix):**
- Lost day blocks are not restored.
- Transactional safety on `saveTemplateToDb` (so a failed insert can never follow a successful delete) would need a server-side Supabase function. The sanitisation removes the actual trigger, so this is deferred.
- `migrateTpl416` is not idempotent (it splits 15 minutes off Wind down for Meditation on every run, and the `mig416` completion flag is discarded on database reload). Low risk today because the re-migrated local copy is overwritten by the database before it is saved, but flagged as a follow-up.

---

## v4.16.0 (2 June 2026)

### Templates: open up self-directed time, relocate meditation into wind-down
A template overhaul to lighten the day. Driven by the Daily OS review: self-directed blocks were being skipped 90-100% of the time, so they're now removed and that time is left open rather than scheduled-then-skipped.

**Self-directed blocks removed (left as empty open slots):**
- Weekday templates: "Self-time / Hobby" removed.
- Sunday: "Reading", "Leisure / hobby", and "Personal project" removed.
- Each freed window becomes a single empty "Tap to set" slot, not a deleted gap, so the time reads as deliberately open.

**Wife time removed from weekday nights:**
- The three weekday templates no longer carry an evening "Wife time" block.
- Saturday's morning "Wife time" is kept (only weekday-evening Wife time was in scope).

**Meditation relocated to night practice:**
- Existing meditation blocks (weekday early-evening, weekend morning) removed; their slots become open.
- Meditation is now the last 15 minutes of the wind-down block on every template.
- On weekday office/WFH days this lands at exactly 22:15-22:30 (10:15-10:30pm). On other day types it sits at the end of that template's wind-down. No sleep time changed on any template.

**How it's applied:**
- `migrateTpl416()` transforms templates by block label (not by template key), so it correctly covers all template ids including the legacy `w_wfh` key in existing data.
- Defaults (`DEF_TEMPLATES`) are transformed for new installs and resets.
- Existing saved templates are migrated once, guarded by a `data.mig416` flag.
- Adjacent empty slots are merged into one.

**Scope notes:**
- Existing logged days are not changed. Template edits only affect future day-type applications, consistent with prior template changes.
- Sunday becomes largely open by design (three open windows through the day). Flag if you want some structure returned to it.

**Important for evaluations:** the new empty open slots carry a blank label and no category. They are intentional open time and must not be read as skipped activities or as any kind of signal in Daily OS reviews.

---

## v4.14.3 (24 May 2026)

### Recent entries: anchored to tracking baseline (1 May 2026)
The Recent entries chart now filters to entries on or after 1 May 2026, marking the start of the period where calorie and protein tracking became central to Daily OS (Meal Tracker shipped in v4.2.0 on 4 May; 1 May used as the cleaner month-boundary anchor).

**Behaviour:**
- Weight entries before 1 May 2026 are excluded from the Recent entries chart
- The "last 10" cap still applies, taken from the filtered set
- When fewer than 10 entries exist since the baseline, the chart shows whatever's available (3, 5, 8 entries, etc.); as more weight is logged, the chart fills up to 10 and then rolls forward like the original Last 10 behaviour
- Existing chart rendering unchanged (line chart, directionally-coloured dots, latest highlighted, summary footer)

**Fallback states:**
- 0 entries since 1 May: card shows "No entries since 1 May. Log a weight to see your journey."
- 1 entry since 1 May: card shows "Only one entry since 1 May. Log a few more to see the chart."
- 2+ entries: chart renders normally

**Why the change:** the previous "Last 10" view could pull entries from before active nutrition tracking began, muddying the narrative. Anchoring to 1 May ties the weight chart to the period where it actually carries signal (tracked nutrition vs. observed weight change).

**Note:** the Key dates card and the summary stats at the top of Weight History still use all available data, including pre-baseline entries. Only the Recent entries chart applies the filter.

---

## v4.14.2 (24 May 2026)

### Recent entries: list replaced with compact chart
The list-based Recent entries card introduced in v4.14.1 has been replaced with a graphical version, validated via mockup before build.

**New card:**
- Compact line chart showing the last 10 individual weight logs (no weekly aggregation, unlike the main chart)
- Y-axis grid lines at integer kg values, padded to a minimum 2 kg range for readability
- X-axis date labels at up to 4 anchor points (first, ~1/3, ~2/3, last) to avoid clutter
- Dots are directionally colour-coded against the previous entry: green for decrease, red for increase, dim for unchanged (within 0.05 kg)
- The latest entry is highlighted with a larger dot, white border, and inline weight value
- Footer line shows entry count, date range span, and net change across the visible 10 entries

**Edge cases:**
- 0 entries: card not shown (existing upstream check)
- 1 entry: fallback message ("Only one entry so far, log a few more to see the chart")
- 2-9 entries: chart drawn with fewer points; date label anchors adapt

**Why the change:** the list version surfaced the same data but didn't add anything the brain wasn't already doing with text. The chart shows trajectory at a glance.

---

## v4.14.1 (24 May 2026)

### Daily reflection collapsible
The Daily reflection section on the day view now follows the same collapse pattern as Goals, Upcoming, and Past sections. Tap the header to expand or collapse.

**Default state:** collapsed.

**State persistence:** localStorage via the existing `collapseState` mechanism, alongside the other three sections.

**Header hint:** the right side of the header shows "Not started" or "N fields filled" so you can see at a glance whether anything has been captured without expanding.

**Why this matters:** the section was always visible and added vertical length to every day view, even on days when reflection isn't the priority. Now it's tucked away until summoned.

### Recent weight entries (later replaced in v4.14.2)
A "Recent entries" card was added to the Weight History view as a list. Replaced in v4.14.2 with a chart after mockup review.

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
