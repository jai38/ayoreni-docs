# Ayoreni status

Each session reads this first and updates **only its own section** at the end of a piece of work: one or two lines, dated, with links to PRs and issues. The owner edits "Now" and "Owner decisions".

## Now (owner)

- **Current phase:** 4b, simple and modern screens. Brief: `docs/phases/04b-brief.md` (re-based on the designer's handoff on 19 November 2026).
  - Tasks 1–4 are done. The design is final.
  - **Design sources:** the designer's branch [`designes/initial-upload`](https://github.com/jai38/myhealthapp/tree/designes/initial-upload) in `jai38/myhealthapp`. Start with [`designes/DEV-HANDOFF.md`](https://github.com/jai38/myhealthapp/blob/designes/initial-upload/designes/DEV-HANDOFF.md), then `designes/design/screens.md` and `designes/design/colour-shape-spec.md`. Where they differ from the canvas, they win. Canvas: https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9. Never merge the design branch into app `main`.
  - **Next (UI):** foundations first: v4 tokens in light and dark (task 7), Plus Jakarta Sans (8), four tabs (Today, Records, Medications, Profile; Health Plan, Reports and Timeline no longer tabs but still reachable) (9), render tests (10), shared pieces (11). Then S06 Today (13) and S07 Dose update (14).
  - **Next (backend):** task 16, for S06/S07: accept a user-chosen Missed, refuse a future taken-at time, add a structured skip reason, and return `fullName` from `/auth/me`. The UI raises it as a `from-ui` issue (task 15).
  - **Designer:** fix the light-mode contrast failures listed in the brief; mark S07 locked; add the "no WhatsApp on this number" state to S01/S02; then design Medication Details, then Records, Medications and Profile.
  - **Owner decisions made 19 November 2026:** new look accepted; search and bell hidden; S07 locked (Missed only from the user); Today may scroll; ring without a percentage; S01–S05 move to Phases 5–6, with allergies stored in Phase 6 and no allergy safety checks until Phase 16.
  - **D1 decided:** WhatsApp OTP only (see "Owner decisions made"). S01–S03 are built in Phase 5, which waits on the owner's Meta verification (`docs/ops/whatsapp-otp-setup.md`).
- **Next phase:** 5, accounts and sign-in with WhatsApp OTP. Draft brief: `docs/phases/05-brief.md`. **P1–P7 stay open** (owner, 19 November 2026: still under discussion), so no Phase 5 build work starts yet; 4b carries on. Owner tasks 1–3 can start now (Meta verification, the template, a real release key and a domain). Going live is blocked on Meta approval. The designer updates S01/S02 for WhatsApp, including the "no WhatsApp on this number" state (brief task 4).
- **Moved to cloud sessions on 19 November 2026.** There is one lead, one UI and one backend session; see `WORKING-AGREEMENT.md`.

## What's live

- **Backend:** AWS Mumbai (`ap-south-1`), ECS Fargate (API plus an on-demand AI worker), RDS Postgres, private S3 with KMS, SQS, API Gateway.
  - Deploys run through the manual `deploy-aws.yml` workflow. Pushing to `main` only runs CI.
  - ECS settings come from `infra/ecs.tf`. The owner applies Terraform, then runs the deploy.
- **Extraction:** Gemini `gemini-3.5-flash-lite`, with no `temperature`. It replaced the deprecated `gemini-2.5-flash` on 6 November 2026.
  - The owner checked it on a phone with a cursive prescription. `gemini-3.8-flash` is the step up if quality drops.
  - The worker scales from 0, so the first upload after a quiet spell waits 2–3 minutes.
- **App status:** `GET /api/v1/app-status` is live: minimum 1.0.0, latest 1.0.0, maintenance off.
  - `APP_MIN_VERSION` and `APP_LATEST_VERSION` must be `x.y.z`, and the minimum can't exceed the latest. A blank or invalid value stops the API at startup.
  - Don't raise the minimum before there's a real store listing.
- **App:** `main` is the Phase 4 build. The APK pre-release is `v0.3.0-phase4-preview`; the GitHub release `v0.2.0-aws-preview` is overwritten by CI on every app push to `main`.
- **Dev copy on the same AWS account** (backend PRs #1–#3, 7 November 2026): schema `dev`, its own bucket, `api-dev`, and a synthetic seed (`npm run seed:dev`).
  - App switch: `npm run start:dev|prod|local`.
  - The owner still has to switch it on (`dev_enabled = true`, apply, set secrets and GitHub variables, run the dev workflow). Check before assuming it's live.
  - Guide: `docs/dev-data/README.md`.

## Owner decisions made

- 2026-11-19, D1 (revised the same day): sign-in uses a **mobile number plus a one-time code sent by WhatsApp only**, as designed in S01–S02. There is no SMS for now, so no DLT yet; `docs/ops/dlt-registration.md` is kept for a later SMS fallback. The owner is starting Meta business verification; the checklist is in `docs/ops/whatsapp-otp-setup.md`. Phase 5 is replanned around WhatsApp OTP and can't go live until the Meta verification and the authentication template are approved. The designer needs to add a "no WhatsApp on this number" state to S01/S02.

## Owner decisions still open

- The legal text still says "MyHealth" in six places.
- The store link `https://jai38.github.io/MyHealth/` is a placeholder.
- The "Hosted in Mumbai" and "AES-256 / private S3" wording isn't confirmed.
- Keep or remove the Profile privacy badge.
- Phase 2 is still open, waiting on the budget alert email.

## UI session

- 2026-11-19: Brief task 5 is done on `phase-4b/simple-screens`, in draft PR [jai38/myhealthapp#3](https://github.com/jai38/myhealthapp/pull/3) (not merged).
  - **Setup:** React Native Testing Library 14 runs under Vitest as a second project, `render`, using a test-only `react-native` stand-in. `renderPreview(tab, state)` renders the real tabs from the dev-preview fixtures.
  - **Tests:** 13 tab render tests across all four tabs, and a one-notice check.
  - **Known failure:** Today `allDone-uploading` is marked `it.fails`, because upload progress and "All done" stack (audit item Today 1). Tasks 6 and 7 must flip it to passing.
  - **Gates:**
    - Before the change: typecheck passed; 550 tests in 48 files.
    - After the change: typecheck passes; 573 tests in 50 files.
  - **Not covered:** the "button label fits at the largest text size" check. The test renderer has no layout, so it needs the screenshot measurements.
  - Nothing is needed from the backend.
- 2026-11-19, D1 (WhatsApp OTP), UI inputs for the Phase 5 replan. The replan itself is the lead's.
  - **Package name:** the WhatsApp checklist had the wrong one. It is `com.ayoreni.app`, and `com.ayoreni.app.dev` for the dev build. Fixed in `docs/ops/whatsapp-otp-setup.md`.
  - **Signing-key hash: blocked.** CI-built APKs are signed with the public React Native debug key, so autofill needs a real release key first (owner).
  - **Autofill needs native code:** Android zero-tap and one-tap need a small native piece (a handshake with WhatsApp before the code request, and a receiver for the code). I know of no maintained Expo package for it, so plan a config plugin and a development build. iPhone gets copy code plus paste.
  - **Backend issues:** none opened yet. The UI will open `from-ui` issues for `POST /auth/otp/request` and `/verify` once the Phase 5 brief fixes their shapes, including a distinct "no WhatsApp on this number" error.

## Backend session

- 2026-11-19: First cloud session. On backend `main` (`b9a99e4`): `npm ci`, `prisma generate` and `typecheck` are clean; `npm test` gives 399 passed and 6 skipped (23 files passed, 1 skipped: the Docker suite). `docs/contracts/api.md` now has every route's body, query, response and error codes, taken from the code. Its "Known gaps" lists 7 bugs found while writing it, none fixed yet. The main ones: a cross-patient `confirm-upload` returns 403, not 404 (rule 4); two error paths return 500; report PDFs say "Test Patient". There are no open `from-ui` issues. No backend code changed.

## Gotchas

- On Windows, write files as UTF-8 and pass commit messages through a file, with no BOM.
- The app `.npmrc` has `legacy-peer-deps=true`, which Expo Router's peers need.
- Tests and `tsc` exclude `.claude/` worktrees.
- A desktop browser can't sign in to the production API, because CORS only allows GitHub Pages. Use the dev-only screen preview, or the dev API.
