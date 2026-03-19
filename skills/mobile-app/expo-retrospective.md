# Skill: Expo Retrospective

Capture what happened in the iteration cycle and update all tracking docs.

## Steps

1. Write a cycle entry in `processes/<project>/iteration-log.md` using the template:
   ```
   ## Cycle N — YYYY-MM-DD
   **Commit:** [hash]
   **What changed:** [1–3 bullets]
   **What was hard / broke:** [honest notes]
   **Patterns learned:** [anything worth saving to a skill]
   **APK:** [link]
   ```

2. Update `processes/<project>/backlog.md`:
   - Mark completed items as done `[x]`
   - Add new items discovered during this cycle
   - Re-rank P1/P2/P3 if priorities shifted

3. If a reusable pattern was discovered, update the relevant skill file in `skills/`

4. Compose the session report to the user:
   - APK download link
   - 3–5 prioritized suggestions for the next cycle (from backlog + new ideas)
   - One honest note about what was hard or unexpected

## Priority System
- **P1** — High value, low effort. Suggest for next cycle.
- **P2** — High value, higher effort. Plan 2–3 cycles ahead.
- **P3** — Nice to have. Surface when backlog is clear.
