# Us — shared habit tracker (Mikey & Euni)

A small, free web app for two people to track the same habits together on iPhone (added to the Home Screen).

- **Preview (design testing, data saved per device):** https://claude.ai/artifact/8W4XBqFzxXNxmBsrxnVN1a
- **App file:** `index.html` (single file: HTML + CSS + JS, no build step)
- **Hosting:** GitHub Pages from `main` (root). Repo: https://github.com/eunicecho0725/us-habits — live app: https://eunicecho0725.github.io/us-habits/ (possibly Vercel later). Publish updates with `git commit` + `git push`; GitHub CLI is at `~/.local/bin/gh`.
- **Sync:** Firebase Firestore (free plan), compat SDK 10.12.2 loaded from gstatic only when `FIREBASE_CONFIG` and `ROOM_ID` are set at the top of the script. Without them, the app runs in preview mode with localStorage.

## Status
- Design is in active iteration.
- Firebase is **not set up yet**. Next step: create project (Firestore in europe-west6, production mode), publish rules scoped to the room ID, paste `firebaseConfig` + `ROOM_ID` into `index.html`, upload to GitHub, enable Pages, add to both phones.

Firestore rules:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{roomId}/{document=**} {
      allow read, write: if roomId == "ROOM_ID_HERE";
    }
  }
}
```

## Features
- **Tabs (bottom bar):** Today | Week | Log, plus a floating + button.
- **Today:** title is the full date. Only habits scheduled today, grouped by small uppercase labels (MORNING / AFTERNOON / EVENING / ANYTIME) with a hairline. One large check circle per habit. No second info line, no scoreboard.
- **Week:** title is the date range with ‹ › beside it; "↩ Back to this week" link only when off the current week. One pinned day header (weekday + date, today highlighted) aligned to 7 columns of circles below. Swipe left/right to change weeks. Streak shown as a small 🔥N pill after the habit name. Schedule pill on the right (e.g. "Mo Th Sa ›") is the only way to open edit.
- **Log:** read-only timeline of memos and logged amounts, newest first (Today / Yesterday / dates). Tapping an entry opens that day in Week with details open.
- **Tap a circle:** toggles your own check (with a pop animation). Past days are editable; only future days are locked. Logging before a habit's start date moves the start date back.
- **Hold a circle (~450 ms):** opens an inline dropdown under the habit: date, "🎉 You both did it!" banner when both checked, amount field (only if the habit tracks a unit and you checked), shared memos (each with author dot and ×), "+ Add a memo" → input with Save. Tap outside or the bottom handle to close.
- **After-check toast:** "Checked! Hold a circle to log more." + Add details button. Shows for the first 8 check-ins per device.
- **One-time tip** on first launch (tap / hold / swipe).
- **Habit form:** emoji avatar + name side by side; emoji auto-suggested from the name; emoji picker with keyword search (~1,850 emojis); quick-pick templates; days via presets (Every day / Weekdays / Weekends / Clear) + Su–Sa pills; time of day; "More options" (amount unit, start date); Enter saves; Delete is a quiet text link.
- **Settings (⋯):** who uses this phone, hand (left/right; defaults left for Euni), appearance (Auto/Light/Dark), Download a backup (JSON), and in preview mode: Simulate partner's check-ins / Clear simulated data.

## Visual design decisions (keep these)
- iPhone / Human Interface Guidelines style (system font stack, grouped background, glass header and tab bar, semantic tokens).
- Soft, gentle palette (no harsh saturated colors). Euni = soft sky blue, Mikey = soft rose, defined in oklch with light/dark variants. "Both did it" = smooth oklch gradient blue → rose with a ✓ (no half-and-half split, no smiley).
- Filled circles: person color + ✓; checkmark color `--on-fill` (white in light, dark in dark mode).
- Scheduled empty days = soft gray circle; rest days = blank (no dots, no outlines).
- Days with memos: small corner badge dot in the author's color (gradient if both).
- No color legend, no per-person score lines, no partner chips in the dropdown, no Did it / Not today buttons, no Easy/Good/Tough feel picker, no to-do checkboxes on memos.
- Left-hand mode mirrors the bottom bar (+ on the left) and puts Today's check circles on the left.
- Minimal and calm at first glance; details live behind hold.

## Data model (Firestore, under `rooms/{ROOM_ID}/`)
- `habits/{id}`: `{id, name, emoji, days:[0-6], tod:"morning|afternoon|evening|anytime", unit, start:"YYYY-MM-DD", created, order}`
- `checks/{habitId}_{person}_{YYYY-MM-DD}`: `{habitId, person:"euni|mikey", date, done, amount|null, memo, feel?}` (feel no longer used in UI)
- `todos/{habitId}_{YYYY-MM-DD}`: `{habitId, date, items:[{id, text, done:false, by}]}` — these are memos; add/remove with `arrayUnion` / `arrayRemove` so simultaneous edits don't overwrite.
- Sync loads habits fully, and checks/todos only for the last 120 days (live listeners); older weeks load on demand via `ensureWeek()` when navigating back. Offline persistence enabled.

## Working agreements
- Test design changes in the preview link first, then produce an updated `index.html` for GitHub.
- Keep data compatible with the model above; warn before any change that alters stored data and suggest downloading a backup first.
- Always keep the real `FIREBASE_CONFIG` and `ROOM_ID` in place once set.
- Check new UI in light and dark mode at iPhone width (~390 px), in both hand modes.
