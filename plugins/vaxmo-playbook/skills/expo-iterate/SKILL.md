---
name: expo-iterate
description: Use when the user asks to update, improve, iterate on, or build features for an Expo/React Native app. Triggers on any task that involves modifying app code and deploying. This is the core 8-phase iteration skill.
version: 1.0.0
---

# Skill: Expo Iterate

Full iteration loop for Expo/React Native apps. Follow all 8 phases in order.

## Phase 1: Understand
- Read all memory files before touching code
- Read `processes/<project>/backlog.md`
- Parse the user's request completely — what they said AND what they implied
- Note anything missing from the request that matters

## Phase 2: Enrich
- Identify gaps the user missed (edge cases, design system violations, missing states)
- Cross-check request against the design system defined in CLAUDE.md
- Add anything that is clearly needed even if not explicitly asked
- Document what was added and why (briefly, in the commit message)

## Phase 3: Plan
- Write out implementation steps before writing any code
- Identify every file that needs to change
- Confirm no existing utility/component already solves the problem
- Gate: plan must be clear enough that someone else could execute it

## Phase 4: Execute
- Implement all changes
- Zero TODOs. Zero hardcoded colors or sizes outside src/constants/
- Every constants access: `COLORS?.ACCENT ?? '#2F8EF4'`
- One logical commit per iteration cycle (can be multiple files)

## Phase 5: Verify
- Run: `npx expo install --check` — fix any version mismatches
- Grep for TODO in src/: must return zero results
- Grep for hardcoded hex colors outside constants files: must return zero
- Manually trace: does the changed feature work end-to-end? (read the code path)

## Phase 6: Deploy
- Commit with message format: `<type>: <summary>\n\n<bullet list of changes>`
- Push to `master` — GitHub Actions EAS build triggers automatically
- Monitor with: `eas build:list --platform android --limit 1`

## Phase 7: Retrospective
- Run expo-retrospective skill
- Update iteration-log.md, backlog.md in playbook repo

## Phase 8: Report
- Post APK download link to user
- List 3–5 suggestions for next cycle (P1 first)
- One honest note on what was unexpectedly hard

## Non-Negotiables
- Never skip phases
- Never mark done until APK link is confirmed available
- Always update backlog after every deploy
