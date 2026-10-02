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

## Cycle 16 — 2026-10-02 (Real alarms: native module)
**Commit:** 50f3b8e (+ 1b7ec16 version bump to 1.15.0)
**What changed:**
- New local Expo module `modules/vaxmo-alarm` (Kotlin). It is linked via `expo.autolinking.nativeModulesDir`, and `expo-modules-autolinking resolve` confirms the link. AlarmManager `setAlarmClock` is used when exact alarms are allowed, otherwise `setAndAllowWhileIdle`. Alarms are persisted natively and re-registered on BOOT_COMPLETED, MY_PACKAGE_REPLACED and exact-alarm permission changes.
- Ringing: a notification on the existing `vaxmo-alarm-*` channels (ALARM stream, the task's own sound) with `FLAG_INSISTENT`, so the sound and vibration loop, plus a full-screen intent to a native `AlarmActivity` (showWhenLocked + turnScreenOn). It stops on CLOSE or REMIND ME LATER. After 60 s, `setTimeoutAfter` plus a timeout alarm end it and leave a silent "missed alarm" notification.
- All task alarm scheduling moved from expo-notifications to the module (new, edit, complete, delete, alarm off, snooze, notifications off, Clear All). A one-time migration cancels expo alarm notifications and re-registers pending snoozes at their original time.
- USE_FULL_SCREEN_INTENT added, with a one-time explainer and a Settings row "Full-screen alarms" (Android 14+). The exact-alarm tip now shows only when exact alarms are actually off.
**What was hard / broke:** A local Gradle build failed before compiling: with JDK 21, RN's gradle-plugin pulls foojay-resolver 0.5.0, which crashes on Gradle 9 (IBM_SEMERU). Building with JDK 17, as CI does, works. Android also sends a notification's deleteIntent when `setTimeoutAfter` expires. Treating every delete as a dismissal would have swallowed the missed-alarm notice, so a delete near the 60 s mark is treated as a timeout.
**Patterns learned:** Prefer FLAG_INSISTENT over a foreground service for looping alarm sound: no FGS type, no Play declaration, and it works even when the alarm fired inexactly. A native alarm screen beats a deep link into the JS app over the lock screen: it shows with no bundle load and can't expose the rest of the app. On an unlocked phone in use, Android shows a full-screen intent as a heads-up banner; that is OS policy for every alarm app.
**APK:** https://github.com/N3byoo/Vaxmo/releases/download/v1.15.0-59/Vaxmo-v1.15.0.apk (Build 59). Permissions read from the built APK: identical to v1.13.0 except + USE_FULL_SCREEN_INTENT.

---

## Cycle 15 — 2026-10-02 (Tap navigation is one slide)
**Commit:** 86e61c4
**What changed:**
- Tapping a pill or the bottom nav now looks like exactly ONE neighbour slide, however far apart the pages are. During a tap the pager's `pageMargin` is set to `width/k − width`, so ViewPager2's page transformer draws the source and target exactly one screen apart. The pager's own animated `setPage` then looks like a single slide. Pages in between are hidden, and the margin returns to 0 afterwards.
- Today ↔ On Hold: the highlight fades from the old pill to the new one and never crosses Upcoming. HOME from Profile returns to the tab that was open before. Swipes are unchanged; a test checks they are identical to v1.14.0 at every position.
**What was hard / broke:** The obvious fix (jump without animation to the neighbour, then animate) flashes the neighbour. pager-view hosts each page in a clipping FrameLayout, so a page can't be drawn outside its slot. Read ViewPager2 1.1.0's source to confirm `setPageTransformer` applies at once (`requestTransform`). That is why the in-between pages are hidden one frame before the margin is set.
**Patterns learned:** Read the native library source before choosing between hacks. The transformer offset `i − (position + offset)` made the maths exact and testable in Node.
**APK:** shipped in the Cycle 16 push (v1.15.0).

---

## Cycle 14 — 2026-10-01 (Daily build, second prompt) — v1.14.0
**Commit:** 7f701e0
**What changed:**
- One native pager (react-native-pager-view 8.0.0, the SDK 55 pin) holds Today, Upcoming, On Hold and Profile. It tracks the finger, settles past the midpoint or on a fling, and springs back otherwise.
- Header island and pill row sit over the pager, not inside a page. Everything that moves with a swipe is driven from the pager's position + offset in a Reanimated shared value on the UI thread: the header and pills slide out with On Hold on the way to Profile, one blue highlight slides under the pills, pill labels and bottom-nav tabs cross-fade, and the + button fades. No React state changes per frame. Page state updates only when a page settles.
- Every page change goes through `pager.setPage`: pill taps, bottom nav, notifications (`Home` now takes `{ page }`) and Clear All Data. The `'Profile'` route, which never existed, now maps to Home page 3.
- Removed the old system completely: `useSwipeNavigation` (PanResponder) and TabContainer's Animated.Value Home/Profile slide. The PanResponders left are DailyReview slides, ImageViewer swipe-down and SwipeableBottomSheet. None of them changes tabs.
- Reorder drag turns paging off from drag start until release, with a second re-enable in onDragEnd as a backstop.
**What was hard / broke:** On Android, a plain RN View returns true from `onTouchEvent`, so any overlay above a native pager blocks swipes that start on it. The header island is wrapped in `pointerEvents="none"` so a swipe that starts on it still pages. The FAB layer is `box-none`. The pills are still touch targets, so a swipe that starts exactly on a pill does not page. `npx expo install` was replaced by npm 10.9.3 (the lockfile hazard), and the dependency was hand-pinned to the exact `8.0.0` that expo install would have written.
**Patterns learned:** For page-linked UI, keep one scroll shared value (position + offset) and drive everything from it with small clamp/interpolate worklets. These worklets can be unit-tested in Node by pulling the functions out of the source file. Equal-width pills mean the highlight needs only a translateX. Its width is set once, at layout. Cross-fading two stacked label layers gives a smooth colour change without animating text colour.
**APK:** https://github.com/N3byoo/Vaxmo/releases/download/v1.14.0-58/Vaxmo-v1.14.0.apk (Build 58, both prompts in one push)

---

## Cycle 13 — 2026-10-01 (Daily build, first prompt)
**Commit:** c9b8148
**What changed:**
- SHALLOW is now displayed as NORMAL. This is display only, and the stored value stays `'shallow'`. Every label goes through `getPriorityLabel()`.
- AlarmScreen, Phase 1, with no native code and no full-screen intent. It shows the clock, the task title and 2 lines of notes, plus CLOSE and REMIND ME LATER (2, 5, 10, 15 or 30 min; 1, 2 or 3 h). A snooze uses a stable `alarm-snooze-<taskId>` id, so a new snooze replaces the old one and never adds a second. A snooze is cancelled when its task is completed or deleted, or its alarm is turned off (`reconcileAlarmSnoozes` on the save listener). The screen opens when an alarm is tapped, including from a cold start, and when an alarm fires while the app is in the foreground.
- Fixed a cold-start bug that was already there: routes from a notification tap were dropped while the navigator was not ready. They are now queued, replayed from `onReady` and deduped.
**What was hard / broke:** The routing logic was moved to an RN-free module so it could be tested. Default-action ids are matched against the library's real constant.
**Patterns learned:** Give anything that can be scheduled more than once a stable identifier. Then "replace" comes for free and "duplicate" can't happen.
**APK:** shipped in the same push as Cycle 14.

---

## Cycle 12 — 2026-09-27 (Daily build, part C of 3)
**Commit:** 2e1d401 — committed locally, NOT pushed
**What changed:**
- Alarm-style task reminders: three synthesized sounds (Pulse / Rise / Chime) from a committed generator script, one MAX-importance channel per sound, a sound picker with in-app preview, and a Settings default. Stable `alarm-<taskId>` identifier.
- The alarm *replaces* the task's normal time notification rather than joining it — the old "Enable Reminder" switch turned out to be dead for every timed task, so adding an alarm alongside would have fired two notifications at once.
- Migrated `reminderEnabled: true` tasks to alarms, with a one-time cold-start rebuild so they ring as alarms immediately.
- Step 4 (exact-alarm check + explainer) NOT built: needs custom native code for a reliable check.
**What was hard / broke:** Two dependency traps. (1) expo-audio defaults to a background playback service with FOREGROUND_SERVICE_MEDIA_PLAYBACK, a restricted Play declaration — disabled via plugin config. (2) expo-audio declares expo-asset as a peer with range `*`; npm resolved it to SDK **57**'s expo-asset and would have autolinked it into an SDK 55 app. Installing with raw npm (to protect the lockfile) bypasses `expo install`'s version pinning, so this needs an explicit audit. The real Gradle manifest merge failed twice on dl.google.com timeouts, so permissions were verified component by component — including fetching and reading the Media3 AARs expo-audio depends on — rather than from a true merge. Also: a missing `STORAGE_KEYS` import in App.js would have crashed at launch.
**Patterns learned:** After any `npm install` of an Expo module, audit every top-level SDK-managed package against `node_modules/expo/bundledNativeModules.json` — wildcard peer ranges pull the latest SDK. Read a library's *config plugin defaults* as well as its manifest: both expo-image-picker and expo-audio add permissions by default. `PermissionsAndroid.check()` is useless for app-op permissions like SCHEDULE_EXACT_ALARM. Generated assets should ship with their generator script — provenance becomes provable.
**Follow-up (commit 31c4ada, v1.13.0):** Alarm channels moved to the ALARM audio stream (ring on silent) — only those three, verified in native code and by test before committing, since channel sound attributes lock at creation. Step 4 built as the user's option 2: a one-time tip plus a permanent Settings row, opening the system page via expo-intent-launcher. Still no status check.
**Permissions read from the built APK (aapt2), not reconstructed:** no restricted permission. RECORD_AUDIO, SYSTEM_ALERT_WINDOW, USE_EXACT_ALARM, FOREGROUND_SERVICE_* and READ_MEDIA_IMAGES are all absent. New in this release: CAMERA, MODIFY_AUDIO_SETTINGS, ACCESS_NETWORK_STATE. The real APK also showed Maven-sourced entries a node_modules scan can't see — c2dm.RECEIVE and launcher-badge permissions (expo-notifications), and the Install Referrer bind (expo-application). All normal, all pre-existing.
**APK:** https://github.com/N3byoo/Vaxmo/releases/download/v1.13.0-57/Vaxmo-v1.13.0.apk (Build 57, pushed as B+C+follow-up in one push)

---

## Cycle 11 — 2026-09-27 (Daily build, part B of 3)
**Commit:** d32e3a0 — committed locally, NOT pushed (user now controls pushes; a push triggers the APK build)
**What changed:**
- Photos on tasks: take/choose up to 3, resized to 1600px and saved as JPEG q0.7 into app document storage. New `imageService.js`, `TaskImageThumbs`, `ImageViewer` (swipe down to dismiss). File cleanup follows the data via the `onTasksSaved` listener, with transactional editing and a cold-start orphan sweep. The widget now receives only `{title, status}`.
- Swipe navigation Today → Upcoming → On Hold → Profile and back. Home's tab lifted into TabContainer; `useSwipeNavigation` kept on the bubble phase, with a direction lock and a drag guard. Fixed the hook's stale closure.
- Blocked RECORD_AUDIO (image-picker plugin, `microphonePermission: false`) and SYSTEM_ALERT_WINDOW (`android.blockedPermissions`), both verified in a fresh prebuild manifest.
**What was hard / broke:** Three spec assumptions were wrong, and each would have shipped a bug. (1) `FileSystem.documentDirectory` from the main `expo-file-system` import throws at runtime in SDK 55 — and no gate catches it, since the names exist. (2) The swipe-to-complete/delete the brief was protecting were removed in v1.3.0 (`bf057e8`); a stale memory note still listed them. (3) expo-image-picker's config plugin is auto-applied and silently adds a microphone permission. Also caught my own test-harness bugs twice: a test that modelled an impossible sequence, and a stale-closure test that read the wrong responder and reported the old hook as fine.
**Patterns learned:** Before building on a brief's premise, verify it exists in the code — briefs inherit stale beliefs from memory. After adding any native dependency, run a fresh `expo prebuild` and read the actual manifest; config plugins auto-apply and add permissions. Code that deletes user files needs a matching key immune to formatting (file name, not URI) and must rebuild paths from its own folder. When a test result contradicts a confident claim, check the harness before accepting either.
**APK:** not built — committed locally, awaiting push instruction

---

## Cycle 10 — 2026-09-27 (Daily build, part A of 3) — v1.12.0
**Commit:** 40345af (app), 758bf1d (lint gate)
**What changed:**
- Step 0: confirmed the notification-duplication fix is live (f7c7c18). Root cause was discarded notification ids, not a race; locks, awaited `cancelExisting`, and once-per-cold-start daily/evening all verified in code. Gate passed, no changes.
- Fix 1, drag glitch on Home: two causes. (a) `await saveTasks()` ran before `setTasks()`, so the list painted the old order for at least a frame, then snapped. (b) cross-group drops landed in the wrong place, because `sortOrder` was assigned by position across the combined list but the memo always re-buckets CRITICAL → SHALLOW → done. Now: state updates synchronously, persistence runs in the background, moves are clamped into the dragged task's own priority group, completed tasks aren't draggable, and the `Math.random()` key fallback is gone.
- Fix 2, inconsistent gap on Home: the island and pill row were two separately absolute-positioned views, with the pills at a hardcoded `insets.top + 82` assuming a fixed island height. The island's height is text-driven, so it grows with the system font-size setting and OEM fonts. Now one positioned block in normal flow, a fixed `SPACING.lg` gap, and list padding derived from the block's measured height instead of the old `insets.top + 130`.
- Fix 3, multi-select onboarding on Q1–Q3: answers stored as arrays in tap order; single-select Q4–Q5 unchanged. New `utils/onboardingAnswers.js` reads both the old string format and arrays. Notification-permission personalization uses the first option tapped, still reading `challenge` (QUESTIONS[0], asked at step 1).
- Build 4, progress check-ins: new `checkinService.js`. Every 3h from morning+3h, skipping slots <60 min before evening, today's remaining slots plus tomorrow's, only for days with pending tasks. Stable `checkin-YYYY-MM-DD-HHMM` identifiers, cancel-all-checkin-* before every reschedule, own `vaxmo-checkins` channel at default importance, Settings toggle (default ON). Reschedules on cold start, foreground, morning/evening time changes — and on every task write via a new `onTasksSaved` listener in `saveTasks()`, since every task mutation in the app already goes through it.
- Notifications switch now sticks: `areNotificationsEnabled()` in notificationService is the single reader of the setting (ProfileScreen and checkinService call it too). Gated every scheduler, not just launch and foreground — task create/edit, Daily Review reschedule and the all-complete notification all ignored the switch as well. Cold start clears everything once when off; turning it back on rebuilds everything once. Reversed checkinService/notificationService so the dependency is one-way (no import cycle).
- Made `npm run lint:undef` a required Phase 5 gate in CLAUDE.md, config checked in as `eslint.undef.config.cjs`. Whole codebase passes. Lint deps installed with npm@10.9.3 to avoid the lockfile drift; `npm@10.9.3 ci` verified.
**What was hard / broke:** Nearly shipped a crash. `questionHint` used `SPACING?.sm` in a file that never imported `SPACING`. Optional chaining guards against an undefined *value*, not an undeclared *name*, so it would have thrown a ReferenceError on the first onboarding question — and `expo export` passed regardless, because bundling doesn't check for undeclared identifiers. Caught by reading, then added an ESLint `no-undef` + `react/jsx-no-undef` pass over every changed file, and proved it works by reintroducing the bug in a scratch copy. Also found a pre-existing bug: the master "Daily Reminders" toggle was only read by ProfileScreen, so turning notifications off never survived the next cold start or foreground. Fixed in the same cycle at the user's request.
**Patterns learned:** `expo export` is not a correctness check — it will happily bundle a file that crashes on first render. Run `no-undef` on changed files, and negative-test the linter once so a clean result is trusted. For a single-choke-point persistence function, a listener set beats wiring side effects into every caller. A "skip if busy" lock loses updates that arrive mid-run; use lock-plus-rerun-flag so concurrent requests coalesce without being dropped.
**APK:** not built — awaiting instruction

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
