# Vaxmo — Key Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Storage | AsyncStorage only | Single user, local-first. No backend, no auth needed for MVP. |
| Deploy | EAS preview → APK | Fastest path to device. Play Store later when ready for public launch. |
| Navigation | Manual bottom tab bar | Full control over Telegram-style pill design. React Navigation tab navigator doesn't support this easily. |
| Design system | All values in src/constants/ | Prevents drift. Optional chaining (`COLORS?.X ?? fallback`) protects against undefined. |
| Skills repo | n3byoo/playbook (separate repo) | Reusable across all future projects. Not siloed in Vaxmo. |
| Build automation | GitHub Actions on push to master | Removes manual EAS command. Every push = new APK in ~20 min. |
| Font | System default | No custom fonts to keep bundle small and startup fast. |
