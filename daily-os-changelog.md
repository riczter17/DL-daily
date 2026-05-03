# Daily OS Changelog

## v4.1.1 (3 May 2026)

### Energy Score Guide (fingertip reminders)
- Added context-specific energy scoring guides accessible via small "?" buttons
- Three guide variants:
  - **Morning Energy**: 5 = jump out of bed; 1 = drowsy when alarmed (subjective wake feeling)
  - **Sleep block**: 5 = 90+ Excellent; 4 = 80-89 Good; 3 = 70-79 Fair; 2 = 60-69 Poor; 1 = below 60 (Xiaomi Smart Band 10 sleep score mapping)
  - **Other blocks**: 5 = peak / fully engaged; 4 = good / productive; 3 = okay / neutral; 2 = low / dragging; 1 = drained
- "?" button placement:
  - Next to "Morning Energy" header on the day view
  - Next to "Energy" label inside any block's expanded view
- Block-level guide is automatically context-aware: sleep category shows the sleep score guide, all other categories show the standard energy guide
- Tap "?" → modal overlay with 5-row scale and "Got it" button. Tap outside or "Got it" to dismiss.
- Helps anchor against the common underscoring tendency: 3 = neutral and functional, not "below average"

### Subtle Tweak
- Morning Energy 1/5 hint updated from "1 = drained / 5 = peak" to "1 = drowsy / 5 = jump out of bed" to match the wake-feeling framing

## v4.1.0 (3 May 2026)

### Visual Reskin: Calm Classic with Grapemist Accent
- New theme: monochrome (black/white/grey) with grapemist (#8398CA) as the signature accent colour
- Roboto font (Google Fonts) replaces system default sans-serif
- Light + dark theme toggle on home screen header (sun/moon icon, top-right). Persists per device via localStorage (key: `dos-theme`).
- Default theme is dark; toggle switches to light when tapped
- Category colours desaturated to match the calm aesthetic (still distinguishable but no longer vibrant)
- Day type badge colours desaturated similarly
- Edit Templates framing card now uses the accent variable so it adapts to both themes
- All existing UI elements use CSS variables that respond correctly to theme switching

### Theme Variable Sets
**Dark (default):** background `#0e0e10`, surfaces `#18181b`, text `#f5f5f4`, accent grapemist
**Light:** background `#fafaf8`, surfaces `#ffffff`, text `#1a1a1a`, accent grapemist (slightly darker for contrast)

### Notes
- Theme preference is per-device by design: dark in low light, light in bright sun
- No data changes; purely visual

## v4.0aa (3 May 2026)

### Saturday Template Label Update
- Saturday template default block "Breakfast / Rest with wife" updated to "Wife time" to match the consolidated activities list.
- This change is to the DEFAULT only. Existing user template in Supabase still uses the old label until next Reset Templates.

## v4.0z (3 May 2026)

### Bug Fix: Realtime Race Condition Overwriting Local Edits
- Realtime subscriptions in v4.0r reloaded all data from Supabase whenever any user-scoped table changed, including changes from THIS device.
- Symptom: editing a block (e.g. setting a time on a newly-added block) would briefly show the edit, then revert as the realtime reload from the previous write landed with stale data.
- Fix: track timestamp of last local write. When realtime change events arrive within 3 seconds of a local write, skip the reload (the change is almost certainly our own; localStorage cache is already correct).
- Added `noteLocalWrite()` calls at the top of all 10 DB write helpers so any local mutation tags itself.
- Cross-device sync still works: writes from OTHER devices arrive without the 3-second window from this device, so they trigger reload normally.

## v4.0y (3 May 2026)

### Bug Fix: Add Block (revised)
- v4.0x's fix changed the predictable "append after last block" behaviour, which disrupted the user's familiar workflow of immediately adjusting the time after Add.
- v4.0y restores the original behaviour: new block starts where the last block in the array ended.
- If that would produce an invalid range (zero/negative duration, e.g. when the array's last block ends at 1440), it falls through to scanning the timeline for the earliest empty gap.
- Ultimate fallback if everything else fails: 8am-9am.
- The user can immediately adjust the time after Add as before.

## v4.0x (3 May 2026)

### Bug Fix: Add Block Constraint Violation
- `addBlk()` derived the new block's start time from the LAST block in the array's `end` field, regardless of order. When the last block in the array ended at 1440 (midnight, typical for end-of-day Sleep), the new block was created with `start=1440, end=1440`, violating the database CHECK constraint `start_min < end_min`.
- Fix: scan all blocks for the latest end time that's strictly before midnight (1440), and use that as the new block's start. If start would still be at/near midnight, fall back to 8am (480 min). Final safety check ensures non-zero duration.
- Symptom before fix: tapping "Add block" appeared to add a block locally but it disappeared on refresh because the Supabase insert failed silently; only the localStorage write persisted briefly.

## v4.0w (2 May 2026)

### Bug Fix: Reset Templates Persistence
- `resetTemplates()` only updated local state and localStorage, did not call `saveTemplateToDb()` for the reset templates
- After reset, templates would appear correct in the UI but revert to old values on next reload (because Supabase was never updated)
- Fix: after resetting local templates, iterate through all templates and call `saveTemplateToDb(key)` for each
- Brings template reset behaviour in line with checklist reset (which already saved to Supabase)

## v4.0v (2 May 2026)

### Saturday Template Update
Saturday template reworked to match the day's stated theme (Wife/Health + Parents):
- Couples workout shifted to 8:45am-10:15am (1.5h, accommodates wife's preferred timing)
- Pre-workout: 30 min Prep + 1 hour Chores (Saturday's main household chores window)
- Post-workout: Breakfast / Rest with wife block (10:15am-12pm)
- Afternoon: Lunch + 30 min Rest + 30 min Commute to parents
- 2pm-9pm: single bundled Parents / Family time block (7h, internal sequence not separately tracked)
- 9-9:30pm Commute home
- 9:30-10:15pm Wind down (includes night prep + gratitude)
- Reading block removed (per theme: focus is workout/wife + parents + rest)
- 13 blocks total (down from 16)

## v4.0u (2 May 2026)

### Goal-Aligned Templates
Following a coaching review of 20 days of logged data, the day-type templates and checklist defaults were redesigned around explicit goals:
- 5 stated life goals (Sleep, Mental health practices, Physical health, Stress management, Career evolution)
- 2 active goals for May-June (Foundation: sleep + mental health; Career exploration)
- 3 maintained goals (physical health, stress management, parents care)

### Day Type Changes
- **Dropped**: `w_wfh` (workout WFH) — workout is now incorporated by default into `nw_wfh`
- **Added**: `nw_wfh_w` (WFH with wife) — distinct template reflecting shared morning market walk and earlier evening start
- **Renamed**: `nw_wio` label from "Non-workout WIO" to "WIO" (cleaner)
- **Renamed**: `nw_wfh` label from "Non-workout WFH" to "WFH"

### Template Updates
- **nw_wio**: morning sequence reordered (Prep before Meditation), evening sequence expanded to include Cool down, Self-time / Hobby block (replacing unlabelled fun block), Together time, separate night Prep block before Wind down
- **nw_wfh**: morning workout incorporated as default, work ends at 5pm (not 6pm) to allow Outdoor walk, evening flow rebalanced with Self-time / Hobby block
- **nw_wfh_w** (new): wake at 6:30am, market walk + breakfast at 7-8am, meditation at 8am after breakfast, work 8:15am-6pm, evening Walk / Together activity, Together time with wife
- **sunday**: parents time blocks removed entirely, replaced with Leisure / hobby (with wife) afternoon block (4pm window), Personal project / Career exploration block (3h evening), Weekly reflection added before Wind down

### Checklist
- Replaced "Deliberate fun (not Netflix)" with "30 min unstructured time" — better captures protected self-time which serves stress management goal regardless of activity choice

### Aspirational Framing
- Added italicised guiding question on home screen header: "On a day with no surprise, no chaos, full health, what would my best self do?"
- Added expanded framing card at top of Edit Templates screen: same question plus explainer that templates represent aspiration, not what life forces

### Migration Notes
- Templates updated as DEFAULTS only. Existing user templates remain unchanged in user data.
- To pull in new templates: use Reset All Templates feature in Edit Templates screen.
- Custom day type and the parents-related day types (mum_wio, mum_wfh, w_mum_wfh) were not changed; consolidation of these was intentionally deferred.

## v4.0t (2 May 2026)

### Sign Out Button
- Added a subtle "Sign out" link at the bottom of the home screen, below the DATA card
- Shows "Signed in as [email]" above the sign out link for context
- Sign out flow: confirms → unsubscribes from Realtime → clears local cache → calls `db.auth.signOut()` → reloads page
- Useful for switching accounts and for testing the new login flow

## v4.0s (2 May 2026)

### iOS PWA Login Fix
- Replaced single-step magic link login with two-step email + 6-digit code flow
- Code is entered directly inside the app, eliminating the iOS PWA limitation where magic links open in Safari (separate storage context) instead of the home-screen-installed app
- Existing magic link backup remains in the email for non-PWA contexts
- Login UI now has two states: email entry, then code verification
- "Use a different email" button to restart the flow

### Required Supabase Configuration
- Email template (Magic Link) must be modified to include `{{ .Token }}` so the 6-digit code is rendered in the email body
- The same email contains both the code (for PWA) and the link (for browser)

## v4.0r (2 May 2026)

### Offline-First Cache
- App now reads from `localStorage` cache immediately on load for instant render
- Fresh data fetched from Supabase in the background, replacing cache on success
- Refresh feels near-instant even on slow connections
- App works in cached state when offline (read-only)

### Real-time Cross-Device Sync
- Subscribed to Supabase Realtime channels for `days`, `blocks`, `day_checklist`, `checklist_items`, `activities`, `templates`
- Changes made on one device propagate to other open devices within ~1-2 seconds
- 500ms debounce to batch rapid changes
- No more manual refresh needed for cross-device sync

### Required Supabase Configuration
- Realtime publication must include the 6 user-scoped tables (via `enable-realtime.sql`)

## v4.0q (1 May 2026)

### Bug Fix: Time Picker Save
- Fixed: editing block start/end times via the time picker (`pickH`, `pickM`) only saved to localStorage, not Supabase
- Time edits now correctly call `updateBlockInDb()` and persist to the database
- Affected day-block time edits only; template time picker (`tplPickH`, `tplPickM`) was already correctly hooked

## v4.0p (29 April 2026)

### Chrome Rendering Fix (final)
- Decoupled auth event from data fetch via `setTimeout(200)` to let the Supabase library settle before triggering queries
- Render now uses `requestAnimationFrame` to defer to a paint-ready frame
- Resolves Chrome-specific multi-tab lock contention that caused black screen on `F5` after magic link login

## v4.0n / v4.0o (29 April 2026)

### Auth Lock Override
- Added `lock` override to `createClient` config: `lock: function(name,acquireTimeout,fn){return fn()}`
- Disables Supabase's Web Locks API coordination between tabs (not needed for single-user app)
- Prevents `Lock "lock:sb-...-auth-token" was released because another request stole it` errors

### Auth State Detection
- Marks `initialAuthHandled` true on any auth event (`INITIAL_SESSION`, `SIGNED_IN`, `TOKEN_REFRESHED`), not just `INITIAL_SESSION`
- Prevents false "no auth event after 2s" fallback when SIGNED_IN was the actual trigger

## v4.0k - v4.0m (27-29 April 2026)

### Diagnostic Logging
- Added `[initApp]`, `[onAuthStateChange]`, `[handleAuthState]` console logs at each step
- Wrapped async calls in try/catch to surface silent failures
- Added DOM ready check (`#app` existence retry) before initial render
- Fallback: if no auth event fires within 2 seconds, render anyway with `null` user

## v4.0j (27 April 2026)

### Race Condition Fix
- Added `appInitialised` flag so `initApp()` runs only once
- Skip `INITIAL_SESSION` event in `onAuthStateChange` if already initialised
- Wrapped initial init in `DOMContentLoaded` listener
- Prevents double-load race that wiped rendered content

## v4.0i (27 April 2026)

### Phase 4d.6: Checklist Definitions Save
- Added `saveChecklistDefsToDb()` helper using wipe-and-rewrite pattern
- Hooked into `clMove`, `clDel`, `clAdd`, `clReset`
- All checklist edit operations now sync to `checklist_items` table

## v4.0h (27 April 2026)

### Phase 4d.5: Templates Save
- Added `saveTemplateToDb(templateKey)` helper that wipes and rewrites all `template_blocks` for the affected template
- Hooked into 7 template-modifying functions: `tplActSel`, `updTplBlk`, `addTplBlk`, `sortTplByTime`, `tplPickH`, `tplPickM`, `delTplBlk`
- Template edits now persist to Supabase

## v4.0g (27 April 2026)

### Phase 4d.4: Custom Activities Save
- Added `insertCustomActivityToDb`, `updateCustomActivityInDb`, `deleteCustomActivityFromDb` helpers
- Soft delete: removed activities set `is_active=false` rather than deleted
- Hooked into `updBlk` (label/category changes that register custom activities) and `delCustomActivity`
- `loadDataFromDb` now filters out soft-deleted activities

## v4.0f (27 April 2026)

### Phase 4d.3: Day Checklist Save
- Added `saveDayChecklistToDb(date, itemId, checked)` helper using upsert with `(user_id, date, item_id)` composite unique key
- Hooked into `toggleCheck` so checklist tick state syncs immediately
- Seeded default checklist items into Supabase

## v4.0e (27 April 2026)

### Phase 4d.2: Block Save
- Added `insertBlockToDb`, `updateBlockInDb`, `deleteBlockFromDb` helpers
- Hooked into `addBlk`, `delBlk`, `updBlk`, `actSel`, `applyDayType`
- Local blocks now track Supabase ID via `_id` field for update/delete operations
- `applyDayType` made async to handle delete-old-blocks + insert-new-blocks sequence

### Default Templates Seeded
- 9 default templates inserted into Supabase via SQL DO block
- 120 template_blocks total

## v4.0d (26 April 2026)

### Phase 4d.1: Day Record Save
- Added `saveDayToDb(date)` helper using upsert with `(user_id, date)` composite key
- Hooked into `setCustomLabel`, `setMorningEnergy`, `updDayNote`, `applyDayType`
- Day-level changes now persist to Supabase

## v4.0c (26 April 2026)

### Phase 4c: Read from Supabase
- Added `loadDataFromDb()` that fetches all 7 user-scoped tables in parallel using `Promise.all`
- Builds local `data` object structure expected by existing render code
- Replaces localStorage as the source of truth for reads
- App is read-only against Supabase at this stage (writes still go to localStorage)

## v4.0b (26 April 2026)

### Phase 4b: Login Screen + Magic Link
- Added login screen with email input and "Send Magic Link" button
- `sendMagicLink()` calls `db.auth.signInWithOtp()`
- `R()` router checks `authUser` and renders login screen when null
- `db.auth.onAuthStateChange` listener handles auth state changes
- Initial app boot sequence: get session, set authUser, render

## v4.0a (26 April 2026)

### Phase 4a: Supabase Client Integration
- Added Supabase JS library via CDN
- Initialised client with project URL and anon key
- Client renamed to `db` to avoid global namespace collision with `supabase`

## v4.0 Foundation (26 April 2026)

### Phase 1: Schema Design
- 10 tables designed: 7 user-scoped + 3 reference
- Foreign key constraints, check constraints, unique constraints
- Soft delete on activities via `is_active`
- Templates split into `templates` + `template_blocks` for full normalisation

### Phase 2: Schema Implementation
- Created tables in Supabase via SQL: `categories`, `day_types`, `block_statuses`, `activities`, `days`, `blocks`, `checklist_items`, `day_checklist`, `templates`, `template_blocks`
- Indexes on user+date and user+category for performance
- Row Level Security (RLS) enabled on all tables with `auth.uid() = user_id` policies

### Phase 3: Auth Setup
- Magic link email auth enabled
- Site URL: `https://dl-daily.vercel.app`
- Redirect URLs: bare domain + wildcard

### Phase 5: Data Migration
- Migrated 21 days, 329 blocks, 9 customised templates, 13 custom activities, 8 checklist items from v3 localStorage export
- Wipe-and-rewrite strategy in transaction (BEGIN/COMMIT)
- Custom activities deduplicated by label, dominant category retained for the registry

### Hosting Architecture
- Production app: Vercel (`dl-daily.vercel.app`)
- Auto-deploy from `DL-daily` GitHub repo on push
- v3 frozen snapshot retained at `DL-daily-v3` repo (GitHub Pages) for rollback

## v3.5 (21 April 2026)

### WIO Templates
- Morning commute shortened by 15 min: now **7:00-7:45am** (was 7:00-8:00am)
- Breakfast meal block moved earlier: now **7:45-8:00am** (was 8:00-8:15am)
- Work block restored to **8:00am** start (was 8:15am in v3.4)
- Applies to **NW-WIO** and **M-WIO**

### Checklist
- Removed **BP taken (3x/week)** item
- BP is now tracked solely via the weekly BP reading block in the Sunday template
- Migration removes the item from the checklist and clears it from any previously logged day data

### Migration
- Non-destructive auto-migration for existing NW-WIO and M-WIO templates
- Detects the v3.4 pattern (Commute 7-8am, Meal 8-8:15am, Work from 8:15am) and rewrites to the v3.5 pattern
- Skips if the pattern doesn't match (i.e. if templates have been manually edited)

## v3.4 (21 April 2026)

### Templates
- Added **Meal (breakfast)** block after morning commute in WIO templates
- Work block shifted to 8:15am to accommodate

### Block Energy
- Tapping the same energy score twice now **deselects** it
- Consistent with status button behaviour

## v3.3 (19 April 2026)

### Activities
- Added **BP reading** under Health

### Sunday Template
- Added **BP reading** block at 21:00 to 21:15 (15 min, Health)
- Reading block shortened from 45 min to 30 min to accommodate

### Migration
- Non-destructive auto-migration for existing Sunday templates

## v3.2 (19 April 2026)

### My Activities (Persistent Custom Activities)
- Custom activity labels auto-save to a personal activity library
- **My Activities** appears as a new group in the activity dropdown
- Picking a saved custom activity auto-sets its category
- New **My Activities** view accessible from the home screen

### Morning Energy Score
- Panel at the top of the day view, 1-5 rating
- Added to Weekly Summary with average, days logged, days at 3+, and 7-day visual strip

### Bug Fix
- Category picker now explicit for all custom/My Activities entries

## v3.1 (19 April 2026)

### Activities
- Added **Chiro** under Health

### Templates
- Added **Sleep blocks** at the start and end of every default template

### Migration
- Existing user templates auto-upgraded with Sleep blocks if missing (non-destructive)

## v3.0 (17 April 2026)

### Block Status Overhaul
- Replaced **Done / Skip** with three statuses: **Planned**, **Unplanned**, **Skipped**

### Summary Overhaul
- Key Metrics card with Adherence rate, Disruption load, and hours split
- Skipped by Category shown as separate section

### Activities
- Renamed **Family dinner** to **Family time** under Family

## v2.5 (14 April 2026)

### Day View Reorganisation
- Timeline appears first, Daily Checklist collapsible below
- Sort by time button added
- Today button shows the date
- Hybrid home screen: Today + Upcoming always visible, Past in collapsible weekly groups

### Day Type Switching
- Custom three-option modal: Retain blocks, Replace with template, Cancel

## v2.0 (14 April 2026)

### Activity System
- Grouped single dropdown replaces flat dropdown + category selector
- 31 activities across 9 categories

### Day Types
- Added: Mum WIO, Mum WFH, Workout Mum WFH
- Removed: Workout WIO, Leave, Off-site
- Renamed: Open Schedule to Custom

### Templates
- All blocks in 15-minute multiples
- All wind-down blocks set to 45 minutes
- 9 templates total

### Storage
- New storage key: dos-v2
- Auto-migrates from v1.x
