# Ayoreni API contract

**Owner: the backend session.** Change this file only in the same PR as the code change. The app builds against it.

- **Base URL:** the production API Gateway URL plus `/api/v1`. The app reads it from `EXPO_PUBLIC_API_URL`. The dev API (`api-dev`) uses the same paths.
- **Auth:** `Authorization: Bearer <access token>` on every route except `/auth/*` (other than `/auth/me` and `DELETE /auth/account`), `/app-status` and `/healthz`.
- **Identity:** comes from the token only. Never send `patientProfileId`.
- **Another patient's resource:** returns 404, never 403.
- **Responses:** `{ "data": … }` on success, and an error body with a code on failure.

Draft taken from the route files on 19 November 2026. **Request and response shapes are still to be filled in by the backend session from the zod schemas,** route by route, as each one is next touched.

## Public

| Method | Path | Purpose |
|---|---|---|
| GET | `/healthz` | Liveness (no prefix) |
| GET | `/app-status` | `{ data: { minVersion, latestVersion, maintenance: { active, message? } } }`, with `Cache-Control: no-store` |

## Auth: `/auth`

| Method | Path | Purpose |
|---|---|---|
| POST | `/auth/signup` | Create an account |
| POST | `/auth/login` | Sign in and get access and refresh tokens |
| POST | `/auth/refresh` | Swap the refresh token for a new pair |
| POST | `/auth/logout` | Revoke the refresh token |
| POST | `/auth/request-password-reset` | Start a password reset |
| POST | `/auth/reset-password` | Finish a password reset |
| POST | `/auth/verify-email` | Verify an email (`EMAIL_VERIFICATION_REQUIRED` is false until Phase 5) |
| POST | `/auth/resend-verification` | Resend the verification email |
| GET | `/auth/me` | The current user |
| DELETE | `/auth/account` | Delete the account and purge its documents |

## Documents: `/documents`

| Method | Path | Purpose |
|---|---|---|
| POST | `/documents/upload-intent` | Get a signed upload slot |
| POST | `/documents/confirm-upload` | Confirm the upload; the server validates and hashes it |
| GET | `/documents/:documentId/signed-url` | A signed view URL (15 minutes at most) |
| GET | `/documents` | List the patient's documents |

## Prescriptions and extraction: `/prescriptions`

| Method | Path | Purpose |
|---|---|---|
| POST | `/prescriptions/extract-job` | Queue AI extraction for a document |
| GET | `/prescriptions/extract-job/:jobId` | Job status: QUEUED, PROCESSING, SUCCEEDED, FAILED or CANCELLED, with the draft payload |
| POST | `/prescriptions/extract-job/:jobId/process` | Process the job (non-queue mode) |
| POST | `/prescriptions/extract-job/:jobId/cancel` | Cancel the job |
| POST | `/prescriptions/:jobId/confirm` | The patient confirms the reviewed draft. This is the only path from AI draft to fact |
| GET | `/prescriptions/timeline` | Prescription timeline entries |
| GET | `/prescriptions/:prescriptionId` | One prescription |

## Medications: `/medications`

| Method | Path | Purpose |
|---|---|---|
| POST | `/medications` | Add a medicine by hand |
| GET | `/medications` | List medicines |
| GET | `/medications/doses` | Dose occurrences |
| POST, PATCH | `/medications/doses/:doseId/action` | Record Taken, Skipped or Snoozed. Only an explicit user action calls this |
| GET | `/medications/:id` | One medicine |
| GET | `/medications/:id/document-url` | Signed URL of its source document |
| POST | `/medications/:id/activate` | Make it active; reminders start |
| PATCH | `/medications/:id/schedule` | Change the schedule |
| POST | `/medications/:id/pause`, `/resume`, `/complete`, `/discontinue` | Course lifecycle |

## Timeline: `/timeline`

| Method | Path | Purpose |
|---|---|---|
| GET | `/timeline` | Events (search, year, category, paging) |
| GET | `/timeline/:id` | One event |

## Reports: `/reports`

| Method | Path | Purpose |
|---|---|---|
| POST | `/reports/export-data` | Data for the doctor summary (verified records only; AI drafts excluded) |
| POST | `/reports/generate` | Create a report |
| POST | `/reports/generate-pdf` | Create a PDF |
| GET | `/reports/:id/download` | Download a report |

## Known gaps (for later phases)

- There is no server endpoint for an as-needed (PRN) dose; the app saves those on the phone only (Phase 4 review, finding 2).
- A retried prescription confirm returns a conflict if the first response was lost (Phase 4 follow-up).
