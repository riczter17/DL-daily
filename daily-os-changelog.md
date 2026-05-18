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
