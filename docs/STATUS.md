# Ayoreni status

Each session reads this first and updates **only its own section** at the end of a piece of work: one or two lines, dated, with links to PRs and issues. The owner edits "Now" and "Owner decisions".

## Now (owner)

- **Current phase:** 4b, simple and modern screens. Brief: `docs/phases/04b-brief.md`.
  - Tasks 1–3 are done.
  - **The UI design is final** in the owner's work-account canvas: https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9. That canvas is the source of truth for every screen. The older mockups in the owner's personal account are superseded.
  - **Next:** build the approved screens (brief tasks 5–13), with render tests.
- **Next phase:** 5, accounts and sign-in (roadmap section 6).
- **Moved to cloud sessions on 19 November 2026.** There is one UI session and one backend session; see `WORKING-AGREEMENT.md`.

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

## Owner decisions still open

- The legal text still says "MyHealth" in six places.
- The store link `https://jai38.github.io/MyHealth/` is a placeholder.
- The "Hosted in Mumbai" and "AES-256 / private S3" wording isn't confirmed.
- Keep or remove the Profile privacy badge.
- Phase 2 is still open, waiting on the budget alert email.

## UI session

- 2026-11-19: (not started in the cloud yet). Start from the branch `phase-4b/simple-screens`, which has the dev-only screen preview (`/dev/preview/<tab>?state=`) and `scripts/screens.mjs` for before/after screenshots.

## Backend session

- 2026-11-19: (not started in the cloud yet). `main` includes the dev environment and seed. There are no open `from-ui` issues.

## Gotchas

- On Windows, write files as UTF-8 and pass commit messages through a file, with no BOM.
- The app `.npmrc` has `legacy-peer-deps=true`, which Expo Router's peers need.
- Tests and `tsc` exclude `.claude/` worktrees.
- A desktop browser can't sign in to the production API, because CORS only allows GitHub Pages. Use the dev-only screen preview, or the dev API.
