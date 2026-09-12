# Architecture decisions

## G0.1 — MVP scope

- Responsive web MVP for approximately 10–20 invited users.
- Invite-only access; no public signup.
- Private, non-commercial learning/project use.
- No native iOS/Android app, installable PWA, offline downloads or native background playback in this MVP.

## G0.2 — Database source of truth

- PostgreSQL is provided by Supabase Local for development and Supabase Cloud for production.
- Database changes use migration-only workflow in `web/supabase/migrations/*.sql`.
- `web/supabase/schemas/` and `docs/sql` are not database sources of truth.
- Prisma and other ORM migration systems are out of scope.

## Security invariants

- RLS stays enabled for user-facing tables.
- Supabase service-role credentials, provider bearer tokens and cron secrets are server-only.
- Audio and cover storage buckets remain private.
- User ownership comes from the authenticated session, never client-provided roles.

## Accepted risks

No accepted risks yet. Any exception must be recorded here with an owner and upgrade threshold.
