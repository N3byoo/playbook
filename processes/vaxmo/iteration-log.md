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
