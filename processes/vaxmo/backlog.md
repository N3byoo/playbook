# Vaxmo — Improvement Backlog

Priority: P1 = high value + low effort | P2 = high value + medium effort | P3 = nice to have

## Active

- [ ] P1: Stop the lockfile re-breaking Play builds. EAS runs Node 20 / npm 10.9.3 with `npm ci`; this machine runs npm 11.6.2, whose `npm install` strips node_modules/tinyglobby/node_modules/picomatch@4.0.7 and makes the next EAS build fail at Install dependencies. Either align local Node to 20, or switch the Direct workflow to `npm ci` so drift fails on push instead of at release time.
- [ ] P1: Upload AAB v1.11.0 versionCode 5 to Play Console internal testing. versionCode 4 is already consumed on the track, so every new upload needs a strictly higher number — eas.json autoIncrement handles this, but note that failed builds consume numbers too.
- [ ] P1: Upgrade the 10 packages `expo install --check` flags — react-native
      0.83.2 → 0.83.10 and expo 55.0.9 → 55.0.31 among them. Needs its own cycle:
      two patch-package patches pin exact versions
      (react-native-draggable-flatlist 4.0.3, @bittingz/expo-widgets 3.0.2) and
      an RN bump can silently invalidate them.
- [ ] P1: Fix or delete the "EAS Android Build" workflow. It fails at `Setup EAS`
      on 100% of pushes because EXPO_TOKEN is invalid ("Authorization header
      bearer token format is invalid"). Right now a red check on master means
      nothing, which hides real failures.
- [ ] P1: Confirm the widget renders correctly on a physical device — tap-to-open,
      strikethrough on done rows, and 4→3→2 row degradation when resized.
- [ ] P1: Verify morning/evening notifications actually fire on hardware
      (CALENDAR trigger + repeats — still untested on a real device).
- [ ] P2: Replace the ~109 raw hex literals outside `src/constants/` with COLORS
      tokens. 20 files. CLAUDE.md requires zero; 424 uses already follow the
      sanctioned `COLORS?.X ?? '#fallback'` form, so this is the remainder.
- [ ] P2: Sort tasks within priority groups by due time (ascending)
- [ ] P2: Task count badge on Android app icon
- [ ] P2: Add search/filter for tasks (needed once task count grows)
- [ ] P2: Evening review time picker in Settings (same pattern as morning picker)
- [ ] P2: Haptic feedback on swipe-complete, drag-reorder, and button taps
- [ ] P2: Onboarding answers personalize the app (default task count, reminder
      time derived from the workTime answer)
- [ ] P3: Export tasks as text/CSV
- [ ] P3: AMOLED pure-black mode (#000000 background)
- [ ] P3: Replace Unicode emoji icons with proper icon library

## Completed

- [x] P1: Haptic feedback on checkbox toggle — Cycle 4
- [x] P1: Fix streak calculation bug — Cycle 4
- [x] P1: Fix exit modal UX (buttons were backwards) — Cycle 4
- [x] P2: Performance: React.memo + FlatList + useMemo — Cycle 4
- [x] P2: Shared date utility to deduplicate code — Cycle 4
- [x] P1: Fix ghost notifications on task delete — Cycle 4
- [x] P1: Custom time picker in Settings (replace Android default) — Cycle 5
- [x] P1: Default date to today on Create Mission — Cycle 5
- [x] P2: Swipe-to-dismiss on all bottom sheet modals — Cycle 5
- [x] P1: DailyReview slideshow screen (light + full modes) — Cycle 6
- [x] P1: Onboarding flow (5 questions, story slides, splash) — Cycle 6
- [x] P1: Weekly tracker component on ProfileScreen — Cycle 6
- [x] P1: Fix notification duplicates (cancel-before-schedule, separate daily vs task reschedule) — Cycle 7
- [x] P1: DailyReview swipe navigation (PanResponder) — Cycle 7
- [x] P2: "Close Your Day" button when all today's tasks done — Cycle 7
- [x] P2: "Add mission for tomorrow" pre-fills tomorrow date — Cycle 7
- [x] P1: WeeklyTracker empty-day guard + single/double tap UX — Cycle 7
- [x] P2: TaskDetailScreen notes read-only (removed inline edit) — Cycle 7
- [x] P1: Swipe-to-delete on task cards — v1.2.0 (was still listed Active)
- [x] P1: Error boundary in App.js — v1.2.0 (was still listed Active)
- [x] P1: Notification tap opens relevant screen — v1.4.0 (was still listed Active)
- [x] P3: Widget support (Android) — Cycle 8 (was still listed Active as P3)
- [x] P1: Widget "Couldn't add widget" — bare `<View>` is illegal in RemoteViews — Cycle 8
- [x] P1: Widget tap-to-open + glanceable 15sp task text — Cycle 8
- [x] P1: Onboarding restructure — welcome intro added, 3 story slides removed, 11 steps → 9 — Cycle 8
- [x] P1: Notifications duplicating 10-15x — reschedule scheduled new notifications
      and discarded the returned ids, orphaning one per task per foreground — Cycle 9
- [x] P2: Widget "how to add it" guide replaces the system Alert — animated,
      in-sheet, never leaves the app — Cycle 9
- [x] P1: First Play Store production AAB — package lowercased to com.n3byoo.vaxmo, USE_EXACT_ALARM removed, lockfile synced for npm 10, EAS keystore created — 2026-09-17
