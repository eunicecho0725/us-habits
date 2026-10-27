# Us — shared habit tracker (Mikey & Euni)

A small, free web app for two people to track the same habits together on iPhone (added to the Home Screen).

- **Preview (design testing, data saved per device):** https://claude.ai/artifact/8W4XBqFzxXNxmBsrxnVN1a
- **App file:** `index.html` (single file: HTML + CSS + JS, no build step)
- **Hosting:** GitHub Pages from `main` (root). Repo: https://github.com/eunicecho0725/us-habits — live app: https://eunicecho0725.github.io/us-habits/ (possibly Vercel later). Publish updates with `git commit` + `git push`; GitHub CLI is at `~/.local/bin/gh`.
- **Sync:** Firebase Firestore (free plan), compat SDK 10.12.2 loaded from gstatic only when `FIREBASE_CONFIG` is set at the top of the script **and** the phone has a room ID. Without them, the app runs in preview mode with localStorage.
- **Room ID (secret, never in the repo):** the repo and site are public, so the room ID is not in `index.html`. Each phone opens the private link `https://eunicecho0725.github.io/us-habits/#room=<id>` once; the app saves it in localStorage (`us-habits-room`). Add to Home Screen *from that link* (with `#room=` still in the address), because the Home Screen app has its own storage. The `#…` part is never sent to GitHub's servers. ID must be 16–128 chars of `A-Z a-z 0-9 _ -`.

## Status
- Design is in active iteration.
- Firebase is set up (project `us-habits`, Firestore, rules published, config in `index.html`). Remaining: open the private link on both phones and add to Home Screen.

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
- **Today:** title is the full date. Only habits scheduled today, grouped by small uppercase labels (MORNING / AFTERNOON / EVENING / ANYTIME) with a hairline. One large check circle per habit. Today's memos show under the habit name (author dot + text, secondary color); hidden while that habit's dropdown is open. No memo badge on Today's circles (the memos are visible). No scoreboard.
- **Week:** title is the date range with ‹ › beside it; "↩ Back to this week" link only when off the current week. One pinned day header (weekday + date, today highlighted) aligned to 7 columns of circles below. Swipe left/right to change weeks. Streak shown as a small 🔥N pill after the habit name. Schedule pill on the right (e.g. "Mo Th Sa ›") is the only way to open edit.
- **Log:** read-only timeline of memos and logged amounts, newest first (Today / Yesterday / dates). Tapping an entry opens that day in Week with details open.
- **Tap a circle:** toggles your own check (with a pop animation). Past days are editable; only future days are locked. Logging before a habit's start date moves the start date back.
- **Tap a habit's name** (Today: name + memo lines; Week: the name): toggles that habit's details. Today opens today with the memo field already focused (keyboard up) and no date line; Week opens today, or the latest past day of the shown week, with the date line. Rule: circle = check, name = details.
- **Week details date line:** ‹ Tue, Oct 27 › steps one day at a time (crosses into other weeks; › disabled on today). No arrows on Today. Selected day gets an inset ring (never clipped).
- **Hold a circle (~450 ms):** shortcut to open a specific day. Opens an inline dropdown tight under the habit (no card, aligned with the habit, ~6px below the circles): date, amount field (only if the habit tracks a unit and you checked), date line in secondary style (smaller than the habit name), shared memos (author dot on the left, then text, no dividers between memos; red trash icon only on your own memos → tap turns it into a red "Delete" pill, tap again to delete, tap anywhere else to cancel; no native confirm() since the preview iframe blocks it), "+ Add a memo" → compact input (36px tall, 16px text so iOS doesn't zoom) with Save. Close by tapping the name again or tapping outside; no grabber/handle.
- **After-check toast:** "Checked! Tap the name to add a memo." + Add details button. Shows for the first 8 check-ins per device.
- **One-time tip** on first launch (tap circle / tap name / swipe).
- **Habit form:** emoji avatar + name side by side; emoji auto-suggested from the name; emoji picker with keyword search (~1,850 emojis); quick-pick templates; days via presets (Every day / Weekdays / Weekends / Clear) + Su–Sa pills; time of day; "More options" (amount unit, start date); Enter saves; Delete is a quiet text link.
- **Settings (⋯):** who uses this phone, hand (left/right; defaults left for Euni), appearance (Auto/Light/Dark), Download a backup (JSON), and in preview mode: Simulate partner's check-ins / Clear simulated data.

## Visual design decisions (keep these)
- iPhone / Human Interface Guidelines style (system font stack, grouped background, glass header and tab bar, semantic tokens).
- Soft, gentle palette (no harsh saturated colors). Euni = soft sky blue, Mikey = soft rose, defined in oklch with light/dark variants. "Both did it" = smooth oklch gradient blue → rose with a ✓ (no half-and-half split, no smiley).
- Filled circles: person color + ✓; checkmark color `--on-fill` (white in light, dark in dark mode).
- Scheduled empty days = soft gray circle; rest days = blank (no dots, no outlines).
- Days with memos (Week tab): small corner badge dot in the author's color (gradient if both).
- Dividers only between habits, never between memos.
- No "You both did it!" banner (the gradient circle says it).
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
- Always keep the real `FIREBASE_CONFIG` in place once set (it is safe to be public).
- Never write the real room ID into `index.html`, `CLAUDE.md`, commits, or any file in this folder.
- Check new UI in light and dark mode at iPhone width (~390 px), in both hand modes.
