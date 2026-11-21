# Ayoreni: shared rules for every Claude session

Ayoreni (repo names still say "myhealthapp") is a patient-first personal health record and medicine app for iOS and Android, India-first. Treat every change as sensitive health data.

This repo, `jai38/ayoreni-docs`, is the shared memory between sessions. Read these before you work:
1. `docs/STATUS.md`: where things stand, and what each session did last.
2. `docs/WORKING-AGREEMENT.md`: how the UI and backend sessions work together.
3. `docs/project/ROADMAP.md`: the plan. The current phase brief is in `docs/phases/`.
4. `docs/contracts/api.md`: the API both sides build against.
5. `docs/project/plan/plan.json`: the data behind the owner's plan map (https://claude.ai/artifact/BwYL4EwQyyhPkY2nkEWras). Update your own tasks' status there when you update `STATUS.md` (working agreement rule 9).

## Repos

| Repo | What | Stack | A push to `main` does |
|---|---|---|---|
| `jai38/myhealthapp` | Mobile app (UI session) | Expo SDK 57, React Native 0.86, React 19.2, TypeScript 6 strict, Expo Router | Deploys GitHub Pages and builds an Android APK |
| `jai38/myhealthapp-backend` (public) | API and AI worker (backend session) | Node 22, Express 4, TypeScript strict, Prisma 5, PostgreSQL | Runs CI only. AWS deploys are the manual `deploy-aws.yml` workflow |
| `jai38/MyHealth` | Landing page | Static HTML, CSS, JS | Deploys the landing page |
| `jai38/ayoreni-docs` (private) | Roadmap, briefs, reviews, contracts, status | Markdown | Nothing |

The backend repo is public: never put owner details, emails, account numbers or plans there. Those belong in this private repo.

## How work runs

- One phase at a time, with owner approval between phases. Each phase ships functionality and UI together.
- Opus writes briefs and reviews diffs. Sonnet takes migrations, medicine rules, auth, encryption, sync and permissions. Haiku does volume and copy work. After two failures, escalate.
- Tasks marked **Owner** in a brief are the owner's. Don't attempt them.
- Commit on phase branches (`phase-N/...`). Push and open PRs freely, but merge to `main` only when the owner says so.
- Use conventional commits, run typecheck and tests before calling work done, and touch nothing in `dist/`.

## Owner-only (never from a cloud session)

- AWS console, CLI and credentials; `terraform plan`/`apply`; `infra/envs/*.tfvars`.
- `.env` files and any key or token.
- Approving production deploys. The owner runs `deploy-aws.yml`, or tells a session to.

## Non-negotiable rules

**AI tools and secrets**
1. Never open or print `.env`, keys or AWS credentials. Never give a coding model production access or real patient data. Use synthetic data only (seed emails `@seed.ayoreni.test`; mock data such as "Asha Test" and "Testmed A 500 mg"), including for bug reproduction.
2. Tests use MinIO or memory storage, never real buckets.

**Ownership and security**
3. Identity comes only from the verified JWT, never from a client-supplied `patientProfileId`.
4. Cross-patient access returns 404, not 403.
5. Storage keys match `documents/<patientProfileId>/<uuid>` exactly.
6. Documents are private, reached only through signed URLs that last at most 15 minutes.
7. Uploads validate magic bytes, size, pages and dimensions, with a server-computed SHA-256. Fail closed.
8. No secrets in the mobile bundle. No health data in logs or audit records: IDs, action codes and timestamps only.
9. `validateProductionConfig()` stays intact and fails closed.

**Medical data**
10. AI output is a draft. Nothing becomes a fact until the patient confirms it.
11. Historical prescriptions default to `isActiveMedication=false` and never create reminders.
12. Keep `nameAsWritten` separate from `normalizedMedicineName`.
13. Flag uncertain or missing values; never guess them.
14. Notification delivery, dismissal or snooze never marks a dose Taken. Only an explicit user action does.
15. An unrecorded past dose is "Not confirmed", not Taken, and not presumed Missed. Never synthesize adherence.

**Code**
- TypeScript strict. Use Vitest, with tests beside existing suites. Touching a security or clinical rule requires a regression test.
- The backend uses ESM with `.js` import specifiers.
- Prefer maintained Expo packages over hand-written native code.
