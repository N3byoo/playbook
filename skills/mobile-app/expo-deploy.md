# Skill: Expo Deploy

Deploy an Expo app as an Android APK via EAS and deliver the download link.

## Steps

1. Ensure all changes are committed and pushed to `master`
2. Run: `eas build --platform android --profile preview --non-interactive`
3. Monitor with: `eas build:list --platform android --limit 1`
4. When status is `finished`, retrieve APK URL from `Application Archive URL` field
5. Deliver URL to user in the session report

## Notes
- Profile `preview` produces an unsigned APK for direct install
- Build takes ~15–20 min on EAS free tier
- If GitHub Actions is configured, push to `master` triggers the build automatically
- Validate EAS auth: `eas whoami` before first run in a new environment
- Required token scopes: `build:create`, `build:read`
