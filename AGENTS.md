# Music project instructions

Canonical plan = `D:\\music\\PLAN.md`.

- The GitHub source repository is PUBLIC at `https://github.com/vinh17491/music`.
- Application, user data, storage and runtime access remain invite-only/private.
- Database changes are migration-only under `web/supabase/migrations/`.
- `D:\\agent\\` is protected and read-only during project work.
- Do not code on `main` or `master`; use `codex/<phase>-<slug>` branches.
- Preserve unrelated user changes and never commit secrets.
- After each phase, update `web/PHASES.md`, run the phase audit, review the diff, commit and push when the remote is available.
- Debug failures with Observe → Reproduce → Gather Evidence → Root Cause → Test Hypothesis → Fix → Regression Test → Verify.
- Do not claim completion without fresh verification evidence; the project is complete only at Z7.8.
