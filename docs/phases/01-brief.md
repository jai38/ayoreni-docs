# Phase 1 Brief: Stabilize and set up the AI process

**Risk:** High. **Roadmap:** `docs/project/ROADMAP.md`, section 6.
**Goal:** local work cannot touch production, fake Taken history is gone, and the build process is proven on one task.

## Hard rules

1. **Do not push the backend to `main` until task 2 is decided.** A push auto-deploys to Render, and Render runs `prisma migrate deploy` on boot, so it would apply the pending migration to production.
2. No model opens `.env` or receives a key, AWS credential, production database access or real patient data.
3. Never run `prisma migrate dev`, `migrate deploy`, `migrate reset`, `db push` or the integration suite until local `.env` points at Docker (task 3).
4. All work goes on a `phase-1/*` branch in each repo. Commits follow the existing conventional style.
5. Any task that fails twice goes to the next model up, or to the owner.

## State found on 2 November 2026

- `myhealthapp-backend`: 506 changed lines across 6 tracked files, plus untracked `prisma/migrations/20261019175900_add_auth_tokens_audit/`, `prisma/migrations/migration_lock.toml` and `src/modules/auth/email-service.ts`.
- `myhealthapp` and `landing`: clean, on `main`.
- `generateTrailingWeekDoses` is in `services/medications/medication-store.ts`.
- `render.yaml` holds a committed storage key, and `CLIENT_URL` is `*`.

## Tasks

| # | Task | Who | Done when |
|---|---|---|---|
| 1 | Read-only production check: migration status and row counts | **Owner** | Results recorded here |
| 2 | Decide how the pending migration ships, per the table below | **Owner + Opus** | Decision recorded |
| 3 | Point local `.env` at Docker Postgres and MinIO | **Owner** | `docker compose up -d`, then `migrate deploy` runs locally only |
| 4 | Review the uncommitted backend work, split into commits, remove the third-party sign-in route, helpers and any matching env values | **Owner**, Sonnet for auth fixes | Typecheck and tests pass; no such route remains |
| 5 | Add tests for the kept auth work: reset token reuse and expiry, unknown email gives same reply, account deletion scope | Sonnet | Tests pass |
| 6 | Rotate the storage key, remove it from `render.yaml`, set `sync: false`, restrict `CLIENT_URL` | **Owner** for the key, Haiku for the file | Old key revoked; no secret in the file |
| 7 | Remove fabricated dose history: `generateTrailingWeekDoses` and the backfill in `addMedication()` | Sonnet | Regression test proves no past dose becomes Taken without a user action |
| 8 | Run the integration suite in CI with Postgres and MinIO services | Haiku | CI runs it instead of skipping |
| 9 | Correct `CLAUDE.md` to the new direction: AWS plan, sign-in, models, real state | Opus writes, Haiku applies | Matches this roadmap; shorter than today |
| 10 | Invariant test kit: patient isolation, 404 on cross-patient access, storage key format, no PHI in logs, notifications never mark Taken | Sonnet | Runs in CI |
| 11 | Synthetic document set, including handwriting | **Owner**, Haiku for printed samples | At least 20 documents, no real names |
| 12 | Templates: brief, review, per-phase usage log; screenshot script | Haiku | In `docs/phases/` and `scripts/` |
| 13 | Set the Claude API spending limit | **Owner** | Limit set |
| 14 | Haiku pilot: build screen Y2 (global error and no connection) from a short brief | Haiku | Result and token use recorded; decides the screen-work model |

## Task 2 decision table

| Task 1 result | Action |
|---|---|
| Migration **not applied**, little or no real data | Rework the migration to drop third-party sign-in fields, then ship it |
| Migration **not applied**, real user data exists | Same rework, but take a backup first and apply it deliberately |
| Migration **already applied** | Do not edit it. Add a new migration to remove the fields, and restore the matching history locally |
| Status shows drift or failure | Stop. Opus reviews before anything else |

## Review

Opus reviews the diffs and gate results once at the end. Owner demo: fresh install, add a medicine, and confirm no past dose appears as Taken.

## Results

- Task 1: production has 11 users, 20 documents, 18 medications. Migration `20261019175900_add_auth_tokens_audit` is not applied, with no drift reported. Whether the users are real is unconfirmed, so they are treated as real.
- Task 2: the owner chose to start fresh on AWS, with no data moved from Supabase and no backup needed. The migration is reworked and stays off production.
- Task 3: done. Local database is Docker on port 5433, because another project holds 5432. The reworked migration applied cleanly on an empty local database. Local `.env` still holds the Supabase storage key, so local uploads reach that bucket until Phase 2.
- Task 6: key rotation deferred by the owner. Supabase storage is retired in Phase 2, so the key must be revoked at that point at the latest.
- Tasks 4, 5: built on `phase-1/auth-rework` in the backend, uncommitted. Third-party sign-in code is removed. Tokens are hashed and rotating, and account deletion now purges data. 137 tests pass, 6 skipped. The migration SQL is read and adds only one nullable column to `users`, new tables, and drops two id defaults; it has not been run.
- The owner chose to push directly to `main` and accept the production effects, with data treated as disposable. Temporary bridge: `EMAIL_VERIFICATION_REQUIRED=false`, so signup works until Phase 5. `AI_PROVIDER_APPROVED=true` is set by owner decision for test data only, and Phase 2 removes that provider. Both must be gone before the V1 gate.
- Follow-ups: Phase 2 replaces Resend with SES and removes the extraction provider. Phase 5 adds rate limits, 15-minute tokens, verification screens, and a landing page exchange for emailed tokens. `AuthProvider` still has two unused enum values from the applied first migration; remove them in a planned migration. Audit rows keep `user_id` after deletion; the lawyer decides.
- Task 8: CI workflow updated to run Postgres and MinIO. Not yet run on GitHub.
- Task 9: `CLAUDE.md` rewritten.
- Task 10: invariant kit added to both repos, with every route now required to be classified. Open defect, marked `it.fails`: the in-memory repository checks "already confirmed" before ownership. Prisma appears correct by reading, but this is unproven against Postgres. Fixed: `confirm-upload` returned 500 on a non-string key. Gap: the sweep runs on in-memory repositories, so the Prisma paths need the same test before the V1 gate.
- Task 12: templates, usage log and screenshot script added. The script has not captured a screenshot, because Playwright is not installed.
- Task 14: Haiku built Y2 first try, about 76K total tokens, and it passed Opus review. One simple screen, not rendered on a device. Decision: Haiku keeps screen work, and Phase 3 tests it on a harder screen.
- Tasks 11, 13: pending, owner.
- Live deploy: CI green on `3a3dc3a`, APK and web published, but the Render backend still serves old code as of 2 November (`/auth/verify-email` returns 404). Probable cause, unconfirmed: Render lacks `EMAIL_VERIFICATION_REQUIRED=false` and `AI_PROVIDER_APPROVED=true`, so the new version refuses to boot. Check Render's logs.
- Local end-to-end test: sign-up, login, refresh rotation and reuse, and account deletion all pass. A new medicine had 16 doses, none Taken. Bug for Phase 12: a medicine added at 4 AM IST got doses on the previous day, because the date is handled in UTC. Those doses stay PENDING.
- Noted for Phase 3: sign-in claims "encrypted with AES-256" and the dashboard claims "Encrypted Health Vault"; these wordings are unverified.
- Task 7: built on `phase-1/remove-fabricated-doses` in `myhealthapp`, uncommitted. Fabricated past doses are removed and purged on load by exact signature. Opus also fixed the dial and metric showing 100%, All Clear or 0% for days with no doses. Not yet run on a device.
- Known limit, for Phase 12: doses are generated only for 7 days when a medicine is added, and past unanswered doses stay PENDING until the Not Confirmed state exists.
