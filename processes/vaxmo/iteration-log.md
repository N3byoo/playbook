# Vaxmo — Iteration Log

Template for each cycle:
```
## Cycle N — YYYY-MM-DD
**Commit:** [hash]
**What changed:** [1–3 bullets]
**What was hard / broke:** [honest notes]
**Patterns learned:** [anything worth saving to a skill]
**APK:** [link]
```

---

## Cycle 0 — 2026-03-17
**Commit:** 9f38371
**What changed:**
- Project initialized (Expo 55, React Native 0.83)
- Dark mode set, assets verified
**What was hard / broke:** Nothing — ground zero setup
**Patterns learned:** Always set userInterfaceStyle: dark in app.json from day one
**APK:** N/A

---

## Cycle 1 — 2026-03-17
**Commit:** 8598482
**What changed:**
- Full app built: 14 src/ files (constants, services, components, screens)
- App.js wired with React Navigation stack
- All 4 screens functional: Home, TaskDetail, CreateEdit, Profile
**What was hard / broke:** App.js was never updated after src/ was built — had to fix separately
**Patterns learned:** Always update App.js as part of the same commit as screens, not after
**APK:** https://expo.dev/artifacts/eas/sjFwvbxHe3GB1gADiDUFkt.apk

---

## Cycle 2 — 2026-03-18
**Commit:** 91f1bdf
**What changed:**
- Extracted shared BottomTabBar component (Telegram-style pill highlight, rounded top corners)
- Task cards: added visible border + SURFACE_ELEVATED background for contrast
- Header bottom separators and date pill on HomeScreen
- Replaced all icon assets with geometric rock + blue arrow icon
**What was hard / broke:** SURFACE and BACKGROUND colors were too close (4-point difference) — cards were invisible
**Patterns learned:** Always verify card background vs app background has enough contrast before shipping. Add SURFACE_ELEVATED as a step up.
**APK:** https://expo.dev/artifacts/eas/9whWa7nAhJcLbBRdkwJUAP.apk

---

## Cycle 4 — 2026-03-21
**Commit:** 585e993
**What changed:**
- Fixed streak calculation: zero-padded month/day in date keys (was creating inconsistent keys like `2026-2-5` vs `2026-02-05`)
- Fixed exit modal: EXIT button now actually exits, STAY button stays (was backwards)
- Fixed ghost notifications: now cancels both notificationId and overdueNotificationId on delete
- Created shared date utility (src/utils/dateUtils.js) to deduplicate formatting across TaskCard and HomeScreen
- Added React.memo to all 8 components for render optimization
- Replaced ScrollView with FlatList on HomeScreen for virtualized task lists
- Added useMemo/useCallback throughout HomeScreen for derived state
- Added haptic feedback (expo-haptics) on toggle, save, delete, priority, reminder
- Added saving state with disabled button on CreateEditScreen
- Fixed react-native-screens version mismatch
**What was hard / broke:** The vibrant-lamarr branch was never merged to master — discovered master was 10 commits behind with no app code. Had to merge first before any work could start. Also the playbook plugin wasn't registered in Claude Code settings.
**Patterns learned:** Always verify master has the latest code before starting a cycle. Check plugin registration in all 3 config files (known_marketplaces.json, installed_plugins.json, settings.json).
**APK:** Build triggered — waiting for EAS

---

## Cycle 3 — 2026-03-20
**Commit:** 84bf9dd + 8eb177d
**What changed:**
- CLAUDE.md added at repo root with Senior Agent rules and full design system reference
- GitHub Actions EAS auto-build workflow — triggers on every push to master
- 6 memory files initialized (user profile, project state, backlog, workflow, decisions)
- Playbook repo created at github.com/N3byoo/playbook with 3 skills + 3 process docs
**What was hard / broke:** First Actions run failed — EXPO_TOKEN not passed explicitly to EAS CLI despite using expo-github-action. Fixed by adding explicit `env: EXPO_TOKEN` on the build step. Second run passed in 8m27s.
**Patterns learned:** `expo/expo-github-action` installs the CLI but EAS still needs `EXPO_TOKEN` passed explicitly as an env var on the build step. Always add it.
**APK:** Pipeline verified working — APK now builds automatically on every push to master.

---

## Cycle 9 — 2026-09-07
**Commit:** f7c7c18
**What changed:**
- Fixed notifications duplicating 10–15×. `rescheduleTasksOnly()` runs on every foreground; it cancelled `task.notificationId`, scheduled a replacement, and discarded the returned id. Nothing wrote it back, so the stored id stayed pointing at the notification cancelled on the *first* foreground — every later run cancelled a dead id and scheduled another live one nobody held an id for. One orphan per task per foreground, forever. `rescheduleAll()` had the same defect.
- Added `persistNotificationIds()` + a shared `rebuildTaskNotifications()` so both paths write ids back, plus a one-time flag-guarded `repairScheduledNotifications()` to clear orphans already accumulated on existing installs (their ids are unrecoverable, so a single `cancelAllScheduledNotificationsAsync()` and rebuild is the only way out).
- Replaced the widget "ADD WIDGET" system Alert with an in-sheet animated guide: 250ms crossfade to a mock home-screen row with a pulsing press indicator, four numbered steps, CLOSE button. The user never leaves the app.
- Version 1.10.0 → 1.11.0.
**What was hard / broke:** The obvious hypotheses were all wrong. It looked exactly like a concurrency bug — duplicate accumulation, AppState firing repeatedly, a suspected `cancelExisting` race — but the calls are sequential and correctly awaited. An in-flight lock, the intuitive fix, would not have changed anything. The bug was a missing write, not a race. Worth remembering: "duplicates accumulate over time" points at state that never got persisted at least as often as it points at concurrency.
**Patterns learned:** Any schedule-then-store sequence must persist the returned id in the same operation, or the id is lost and the notification becomes uncancellable. When a bug has already shipped and corrupted user state, the fix needs a migration as well as a code change — fixing forward leaves existing installs broken forever. Also: check that new UI colours go through COLORS tokens *before* committing; the first draft of the guide added three raw hex literals.
**APK:** https://github.com/N3byoo/Vaxmo/releases/download/v1.11.0-51/Vaxmo-v1.11.0.apk

---

## Cycle 8 — 2026-09-07
**Commit:** 2eb48bb (also d2425d4, 4b37fdc)
**What changed:**
- Widget finally works. Two separate bugs: the launcher rejected it outright because both layouts used bare `<View>` elements as dividers, and `RemoteViews` installs an inflater filter that only accepts classes carrying `@RemoteView` — `android.view.View` does not have it. Replaced with `ImageView`. Then made it useful: tap anywhere opens the app, task text 11sp → 15sp, red left-edge critical marker per row, real strikethrough on completed tasks.
- Onboarding restructured from 11 steps to 9 — added an animated welcome intro as step 0, removed the three story slides ("The Problem" / "Why It Happens" / "The Solution") and every helper only they used. Net −485/+168 lines in one file.
- Bumped 1.9.0 → 1.10.0. Reconciled the backlog: 7 items were sitting in Active that had already shipped.
**What was hard / broke:** The C: drive hit literally 0 MB free mid-cycle and `git` itself failed to write its index lock, blocking a commit. Cleared ~23 GB (Windows Update cache alone was 13.5 GB) and relocated the Gradle cache, Android SDK and npm cache to D:. Two self-inflicted mistakes during that: the TEMP sweep deleted the agent's own task-output files, and a process-kill filter matched the very PowerShell session running it, aborting the command with `wuauserv` left stopped — restarted it. Separately, `android.jar` turned out to be a stub jar that strips `@RemotableViewMethod`, so reflection-based RemoteViews calls can't be verified locally.
**Patterns learned:** RemoteViews rejects any view class without `@RemoteView` — check with `javap` against `android.jar` before adding a view type to a widget layout; class-level annotations survive the stub jar even though method-level ones don't. `StrikethroughSpan` and `StyleSpan` are both `ParcelableSpan`, so styling via `SpannableString` through `setTextViewText` survives parcelling into the launcher process — no reflection and no checkmark-prefix hacks needed. When deleting the Windows Update cache, always restart `wuauserv` in a `finally`.
**APK:** https://github.com/N3byoo/Vaxmo/releases/download/v1.10.0-50/Vaxmo-v1.10.0.apk

---

## Cycle 7 — 2026-04-30
**Commit:** 6eeaab6 / 5ccb911
**What changed:**
- Notification duplicate prevention — `cancelExisting()` helper; daily reminders only on mount, task notifications only on foreground resume (`rescheduleTasksOnly`)
- DailyReview swipe navigation — PanResponder horizontal swipe to advance/retreat slides; stale closure solved with `currentSlideRef` + `slideCountRef`
- "Close Your Day" button — gradient footer on HomeScreen today list when all tasks done, opens full DailyReview
- "Add mission for tomorrow" pre-fills CreateEdit with tomorrow 9am date
- WeeklyTracker: empty days silent on tap; single tap = inline stats, double tap = navigate to day; `mode: 'light'` always passed from ProfileScreen
- TaskDetailScreen notes made read-only (removed inline TextInput edit, replaced with LinearGradient card)
**What was hard / broke:** GitHub Actions artifact storage quota was full — Direct APK upload failed (code was fine). Two duplicate EAS builds stacked in queue from session boundary; had to cancel them. PanResponder stale closure on `currentSlide` required ref pattern.
**Patterns learned:** Separate daily reminder scheduling (mount) from task notification rescheduling (foreground) to prevent duplicate notifications. PanResponder created via `useRef` captures initial state — always mirror changing values into refs for use inside the closure.
**APK:** https://expo.dev/artifacts/eas/csZeZY3TY8pHzCHU8KmmJe.apk

---

## Cycle 6 — 2026-04-06
**Commit:** 450ef28
**What changed:**
- DailyReview slideshow screen (light + full modes), notification fixes, inline notes, Outfit fonts loaded via expo-font
- Full onboarding flow: 5 question screens, loading, 4 story slides + splash screen; rebuilt with safe area insets and screen-relative sizing
- SVG VaxmoMark logo on onboarding; tab-aware empty states; weekly tracker component
- Nav architecture fixes, drag reorder on HomeScreen, bottom sheet improvements
- SDK 55 upgrade (expo + expo-notifications); Reanimated 4 crash fixed by replacing DraggableFlatList; BackHandler cleanup crash fixed
**What was hard / broke:** Reanimated 4 incompatibility with DraggableFlatList caused crash — replaced with custom FlatList drag. EAS build needed babel-preset-expo in devDependencies.
**Patterns learned:** Always check library compatibility against Reanimated version before using drag libs. Screen-relative sizing (`Dimensions.get('window')`) is the correct approach for onboarding fullscreen layouts.
**APK:** https://expo.dev/artifacts/eas/64PvcwUVraePtkBmZHfZxT.apk

---

## Cycle 5 — 2026-03-28
**Commit:** c94760b
**What changed:**
- Replaced Android native DateTimePicker with custom VaxmoDateTimePicker (`mode="time"`) in Settings for morning/evening reminder times — no more Android default popups anywhere
- New missions auto-set date to today (user only picks execution time manually)
- Created SwipeableBottomSheet component (PanResponder drag-to-dismiss) and applied to all 3 bottom sheet modals (Settings, History Task, Task Detail)
- Bumped version to 1.5.0
**What was hard / broke:** Nothing major — clean implementation. Had to split date vs time-set state in CreateEditScreen to distinguish "date defaulted to today" from "user explicitly picked a time."
**Patterns learned:** When adding a `mode` prop to an existing picker, make sure `handleConfirm` returns a different shape per mode (datetime returns `{date, timeType}`, time-only returns `{time: "HH:MM"}`). Consumers need to handle both.
**APK:** EAS build triggered — waiting for completion
