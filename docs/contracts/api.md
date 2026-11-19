# Ayoreni API contract

**Owner: the backend session.** Change this file only in the same PR as the code change. The app builds against it.

Filled in from the code on backend `main` (`b9a99e4`) on 19 November 2026. Every shape below comes from the zod schemas, the handlers and the repository mappers. Where the code does something odd, it says so under **Known gaps** at the end rather than describing what it should do.

> **Also pending: Phase 4b task 16,** in backend draft PR [jai38/myhealthapp-backend#9](https://github.com/jai38/myhealthapp-backend/pull/9), branch `phase-4b/dose-actions`, stacked on PR #4. It covers Missed, no future `actionTime`, `skipReason`, and `fullName` on `/auth/me`. Each of its changes is marked **(PR #9)**. It merges after PR #4.
>
> **Pending, not live yet (updated 19 November 2026):** backend draft PR [jai38/myhealthapp-backend#4](https://github.com/jai38/myhealthapp-backend/pull/4), branch `phase-4b/contract-gaps`, fixes the seven gaps found while writing this file. The routes below describe the behaviour **after** that PR. Each change is marked **(PR #4)**. Until the owner merges and deploys it, production still behaves the old way, recorded in "Known gaps". The app shouldn't rely on the new fields (`kind`, `fields`) until it is live (WORKING-AGREEMENT rule 5).

## Conventions

- **Base URL:** the production API Gateway URL plus `/api/v1`. The app reads it from `EXPO_PUBLIC_API_URL`. The dev API (`api-dev`) uses the same paths.
- **Auth:** `Authorization: Bearer <accessToken>` on every route except `/healthz`, `/app-status` and `/auth/*` (other than `GET /auth/me` and `DELETE /auth/account`).
- **Identity** comes from the token only. Never send `patientProfileId`; the server ignores it.
- **Another patient's resource** returns 404, never 403. **(PR #4)** removes the last exception, `confirm-upload`.
- **Success:** `{ "data": … }`. Some medication routes also add a top-level `"message"` string. `GET /timeline` is different: it returns `{ data, pagination }` at the top level (see below).
- **Errors:** `{ "error": { "code": string, "message": string, "details"?: … } }`. Show `message` only as a fallback; branch on `code` and the HTTP status.
- **Dates:** fields named `…Date` are `YYYY-MM-DD` strings. Fields named `…At` and `scheduledTime` are ISO 8601 timestamps (UTC). Optional fields that are empty are **left out** of the JSON, not sent as `null`.
- **IDs** are UUIDs.
- **Time zone:** routes that build dose schedules read the optional header `X-Timezone-Offset`, in minutes, from -840 to 840. Missing or out-of-range values mean 0 (UTC). Send the same sign as JavaScript's `-new Date().getTimezoneOffset()` (India is `330`). Routes that read it: `POST /prescriptions/:jobId/confirm`, `POST /medications`, `GET /medications/doses`, `POST /medications/:id/activate`, `PATCH /medications/:id/schedule`, `POST /medications/:id/resume`.
- **Request body limit:** 1 MB of JSON. File bytes never go through the API.

### Errors every protected route can return

| Status | `code` | When |
|---|---|---|
| 401 | `UNAUTHORIZED` | No `Bearer` header, or the token is invalid or expired. The app should refresh once, then sign out. |
| 400 | `INVALID_JSON` | **(PR #4)** The body isn't valid JSON. The message is fixed and never quotes the body. Before PR #4 this was a 500. |
| 413 | `PAYLOAD_TOO_LARGE` | **(PR #4)** The JSON body is over 1 MB. |
| 500 | `INTERNAL_ERROR` | Anything unhandled. In production `message` is always "An unexpected error occurred". |

### Validation errors **(PR #4)**

Every `400 VALIDATION_ERROR`, and the timeline's `400 INVALID_QUERY_PARAMETERS`, carries one shared list. Branch on `fields`, not on `details`:
```ts
{ error: { code: 'VALIDATION_ERROR', message: string,
           fields: Array<{ path: string, message: string }>,   // path is dot-joined, e.g. 'medicines.0.name_as_written'; '' for the whole body
           details?: … } }                                     // the older per-route shape, kept so nothing breaks (see each section)
```
`fields` can be empty when the failure isn't tied to one field. No route answers 422 any more.

An unknown path gets Express's default HTML 404, not JSON.

### Shared record shapes

These are referenced by name below. `?` marks a field that is left out when empty.

**Document**
```ts
{
  id, patientProfileId, storageKey,        // storageKey = documents/<patientProfileId>/<uuid>
  originalFilename, mimeType,              // 'image/jpeg' | 'image/png' | 'application/pdf'
  fileSizeBytes: number, sha256Checksum,   // server-computed SHA-256 (hex)
  status: 'UPLOADED' | 'EXTRACTING' | 'READY_FOR_REVIEW' | 'CONFIRMED' | 'FAILED',
  kind: 'UPLOAD' | 'REPORT',               // (PR #4) REPORT = a PDF the server generated; everything the patient uploaded is UPLOAD
  pageCount?: number, imageWidth?: number, imageHeight?: number,
  uploadedAt, updatedAt
}
```

**ExtractionJob**
```ts
{
  id, patientProfileId, documentId, idempotencyKey,
  status: 'QUEUED' | 'PROCESSING' | 'SUCCEEDED' | 'FAILED' | 'CANCELLED',
  retryCount: number, maxRetries: number, providerName: string,
  extractedPayload?: ExtractedPrescription,  // only once SUCCEEDED. An AI draft, never a fact.
  errorMessage?: string,                     // on FAILED; internal text, don't show it to patients
  startedAt?, completedAt?, confirmedAt?,    // confirmedAt set once the patient confirms
  createdAt, updatedAt
}
```

**ExtractedPrescription** (the AI draft, snake_case, validated by `ExtractedPrescriptionPayloadSchema`)
```ts
{
  prescription_date?: 'YYYY-MM-DD',
  date_is_uncertain: boolean,
  doctor_name?, clinic_hospital_name?, documented_diagnosis?,
  medicines: Array<{                       // at least 1
    name_as_written: string,               // exactly as on the paper
    normalized_medicine_name?: string,
    strength?, frequency_text?, duration_text?, instructions?,
    dosage_form: DosageForm,               // default 'TABLET'
    route: Route,                          // default 'ORAL'
    confidence: 'HIGH' | 'UNCERTAIN',
    source_page: number                    // 1-based
  }>,
  unparsed_notes?: string,
  validation_warnings: string[]
}
```
- `DosageForm`: `TABLET | CAPSULE | SYRUP | INJECTION | CREAM | DROPS | INHALER | POWDER | OINTMENT | OTHER`
- `Route`: `ORAL | TOPICAL | INTRAVENOUS | INTRAMUSCULAR | SUBCUTANEOUS | INHALATION | OPHTHALMIC | NASAL | OTIC | RECTAL | OTHER`

**Prescription**
```ts
{
  id, patientProfileId, documentId, jobId,
  prescriptionDate?: 'YYYY-MM-DD', dateIsUncertain: boolean,
  doctorName?, clinicHospitalName?, diagnosis?,
  reviewStatus: 'CONFIRMED', confirmedAt, confirmedByUserId,
  createdAt, updatedAt
}
```

**PrescriptionItem**
```ts
{
  id, prescriptionId, sourceDocumentId,
  nameAsWritten, normalizedMedicineName?,
  strength?, dosageForm, frequencyText?, durationText?, route, instructions?,
  isActiveMedication: boolean,
  createdAt, updatedAt
}
```

**TimelineEvent**
```ts
{
  id, patientProfileId,
  eventType: string,                 // today always 'PRESCRIPTION_RECORDED'
  eventDate: 'YYYY-MM-DD',
  dateIsUncertain?: boolean, datePrecision?: string,   // datePrecision defaults to 'EXACT'
  title, summary,                    // e.g. "Prescription from Dr X", "2 medications recorded"
  sourceDocumentId, sourcePrescriptionId?,
  isVerified: boolean,               // true: only patient-confirmed data becomes an event
  createdAt, updatedAt
}
```

**TimelineEventDetail** = TimelineEvent plus
```ts
{
  sourceDocument?: { id, originalFilename, mimeType, fileSizeBytes, pageCount?, uploadedAt },
  prescription?: {
    id, prescriptionDate?, dateIsUncertain, doctorName?, clinicHospitalName?, diagnosis?,
    reviewStatus?, confirmedAt?, confirmedByUserId?,
    items: PrescriptionItem[]
  }
}
```

**Medication**
```ts
{
  id, patientProfileId,
  prescriptionItemId?, prescriptionId?, sourceDocumentId?,   // empty for medicines added by hand
  nameAsWritten, normalizedMedicineName?,
  strength?, dosageForm: string, route: string, prescribedInstructions?,
  status: 'INACTIVE' | 'ACTIVE' | 'PAUSED' | 'COMPLETED' | 'DISCONTINUED',
  startDate?: 'YYYY-MM-DD', endDate?: 'YYYY-MM-DD',
  reminderTimes?: string[],          // 'HH:mm', 24-hour, local to the patient
  notes?, discontinuedReason?, discontinuedAt?, completedAt?,
  createdAt, updatedAt
}
```

**MedicationDetail** = Medication plus
```ts
{
  prescription?: { id, prescriptionDate?, doctorName?, clinicHospitalName? },
  sourceDocument?: { id, originalFilename, mimeType },
  doseLogs?: DoseLog[]               // newest 50, newest first
}
```

**DoseLog** (one scheduled dose)
```ts
{
  id, medicationId, patientProfileId,
  scheduledTime,                     // ISO timestamp, UTC
  status: 'PENDING' | 'TAKEN' | 'SKIPPED' | 'SNOOZED' | 'MISSED',
  actionTime?, snoozeMinutes?: number, notes?,
  skipReason?: SkipReason,           // (PR #9) only on SKIPPED, and only when the patient chose one
  createdAt, updatedAt
}
```
`PENDING` in the past means **Not confirmed**. Show it that way; don't show it as Taken or Missed. `MISSED` only ever comes from the patient's own "No, I missed it"; the server never sets it.

`SkipReason` **(PR #9)**: `FELT_UNWELL | DOCTOR_ADVISED | RAN_OUT | SIDE_EFFECTS | OTHER`.

**SignedDocumentUrl**
```ts
{ signedUrl: string, expiresInSeconds: 900, filename: string, mimeType: string }
```

---

## Public

### `GET /healthz` (no `/api/v1` prefix)
**200**
```ts
{ status: 'ok', service: 'myhealthapp-backend', timestamp, environment, deployment }
```
Not wrapped in `data`.

### `GET /app-status`
**200**, with `Cache-Control: no-store`
```ts
{ data: { minVersion: 'x.y.z', latestVersion: 'x.y.z', maintenance: { active: boolean, message?: string } } }
```
`message` is present only when `active` is true and a message is set.

---

## Auth: `/auth`

Auth routes report validation failures as **400 `VALIDATION_ERROR`**, with every zod message joined into `message`, no `details`, and **(PR #4)** `fields`.

**Session** (returned by signup, login and refresh)
```ts
{ userId, patientProfileId, accessToken, refreshToken, expiresAt: number }  // expiresAt = epoch ms of the access token
```
Access tokens last 15 minutes by default (`JWT_EXPIRES_IN`). Refresh tokens last 30 days and **rotate**: each refresh returns a new pair, and reusing an old refresh token revokes the whole session family.

### `POST /auth/signup`
Body
```ts
{ email: string (email, ≤254), password: string (8–72 chars), fullName?: string (≤200) }
```
**201**
```ts
{ data: {
    user: { email, id?, patientProfileId? },   // id and patientProfileId only when a session is issued
    session?: Session,                          // left out when verification is required
    requiresVerification: boolean
} }
```
The reply is the same whether or not the email is already registered. Errors: 400 `VALIDATION_ERROR`.

### `POST /auth/login`
Body: `{ email: string (email, ≤254), password: string (1–1024) }`
**200**: `{ data: { user: { id, email, patientProfileId }, session: Session } }`
Errors: 400 `VALIDATION_ERROR`; 401 `UNAUTHORIZED` (wrong email or password, with the same message either way); 403 `EMAIL_NOT_VERIFIED` (only when verification is required).

### `POST /auth/refresh`
Body: `{ refreshToken: string (1–512) }`
**200**: `{ data: { session: Session } }`
Errors: 400 `VALIDATION_ERROR`; 401 `UNAUTHORIZED` (the token is invalid, expired, revoked or reused).

### `POST /auth/logout`
Body: `{ refreshToken?: string }`. Revokes that token's family.
**200**: `{ data: { message: 'Signed out.' } }`. Always succeeds, even with no body or a bad token.

### `POST /auth/request-password-reset`
Body: `{ email: string (email, ≤254) }`
**200**: `{ data: { message } }`, the same whether or not the account exists. The reset link lasts 1 hour.
Errors: 400 `VALIDATION_ERROR`.

### `POST /auth/reset-password`
Body: `{ token: string (1–512), password: string (8–72) }`
**200**: `{ data: { message } }`. Ends every existing session for the user.
Errors: 400 `VALIDATION_ERROR`; 400 `INVALID_TOKEN` (invalid, expired or already used).

### `POST /auth/verify-email`
Body: `{ token: string (1–512) }`
**200**: `{ data: { message } }`
Errors: 400 `VALIDATION_ERROR`; 400 `INVALID_TOKEN`. Links last 24 hours. `EMAIL_VERIFICATION_REQUIRED` stays false until Phase 5.

### `POST /auth/resend-verification`
Body: `{ email: string (email, ≤254) }`
**200**: `{ data: { message } }`, the same whether or not the account exists or is already verified.
Errors: 400 `VALIDATION_ERROR`.

### `GET /auth/me` (token required)
**200**: `{ data: { userId, email, patientProfileId, fullName? } }`
**(PR #9)** `fullName` is the patient's own name from signup. It is left out when none was given; greet without a name then.

### `DELETE /auth/account` (token required)
No body. Deletes every stored document object and row first, then the user; the database cascades every other patient-owned row.
**200**: `{ data: { message } }`. If the storage purge fails, nothing is deleted and the call can be retried (500).

---

## Documents: `/documents`

There are three upload steps: ask for an intent, `PUT` the bytes straight to `uploadSignedUrl`, then confirm.

### `POST /documents/upload-intent`
Body (checked by hand, not zod)
```ts
{ filename: string, mimeType: 'image/jpeg' | 'image/png' | 'application/pdf', fileSizeBytes: number (1 … 15 MB) }
```
**200**: `{ data: { storageKey, uploadSignedUrl, expiresInSeconds: 900 } }`
Then `PUT` the raw bytes to `uploadSignedUrl` within 15 minutes, with no auth header.
Errors: 400 `BAD_REQUEST` (a field is missing, `fileSizeBytes` isn't a number, the MIME type isn't supported, the file is over 15 MB, or the size is 0 or less).

### `POST /documents/confirm-upload`
Body: `{ storageKey: string, mimeType?: string, clientChecksum?: string (SHA-256 hex) }`
The server reads the stored object and checks its magic bytes, real size (15 MB at most), PDF pages (10 at most) and image dimensions (8000 px at most). It then computes the SHA-256 and compares it with `clientChecksum` when one is sent. Any failure rejects the upload.
**201**: `{ data: Document }` (status `UPLOADED`). Confirming the same key twice in a race returns the existing record.
Errors:
- 400 `BAD_REQUEST`: `storageKey` is missing; the intent is not found, expired, already used or still being checked; the object is missing or empty; the MIME type or checksum doesn't match; the file fails validation. If the object isn't there yet, the intent is released and the call can be retried.
- 404 `NOT_FOUND`, message "Upload not found": **(PR #4)** the `storageKey` is malformed or belongs to another patient. The reply is the same whether or not the key exists. (Before PR #4 this was 403 `FORBIDDEN`.)

### `GET /documents/:documentId/signed-url`
**200**: `{ data: { signedUrl, expiresInSeconds: 900 } }`
Errors: 404 `NOT_FOUND` (missing, or another patient's).

### `GET /documents`
**200**: `{ data: Document[] }`, newest first. This **includes generated report PDFs** (status `CONFIRMED`; **(PR #4)** `kind: 'REPORT'`, so the app can tell them apart). It isn't paginated.

---

## Prescriptions and extraction: `/prescriptions`

The flow: `POST /extract-job`, poll `GET /extract-job/:jobId` until `SUCCEEDED`, show `extractedPayload` for review, then `POST /:jobId/confirm`. Confirming is the only path from an AI draft to a fact.

### `POST /prescriptions/extract-job`
Body: `{ documentId: string, idempotencyKey?: string, autoProcess?: boolean }`
- `idempotencyKey` defaults to `doc-<documentId>-<patientProfileId>`, so a second call for the same document returns the **existing job**, whatever state it is in.
- `autoProcess` defaults to true. In production (queue mode) the job goes to the worker and comes back `QUEUED` or `PROCESSING`. The worker scales from 0, so the first job after a quiet spell can take 2–3 minutes. In local or non-queue mode, extraction runs inside the request.
- The document's status moves to `EXTRACTING`, then to `READY_FOR_REVIEW` on success.

**202**: `{ data: ExtractionJob }`
Errors: 400 `BAD_REQUEST` (`documentId` is missing); 404 `NOT_FOUND` (the document is missing or another patient's).

### `GET /prescriptions/extract-job/:jobId`
**200**: `{ data: ExtractionJob }`. Poll this.
Errors: 404 `NOT_FOUND`.

### `POST /prescriptions/extract-job/:jobId/process`
Starts a `QUEUED` job by hand. If the job is already claimed or finished, it returns the job as it is.
**200**: `{ data: ExtractionJob }`
Errors: 404 `NOT_FOUND`.

### `POST /prescriptions/extract-job/:jobId/cancel`
**200**: `{ data: ExtractionJob }` (status `CANCELLED`). A worker result that arrives after cancelling doesn't overwrite it.
Errors: 404 `NOT_FOUND`; 400 `BAD_REQUEST`, message "Cannot cancel a completed or confirmed job" **(PR #4)**, when the job is `SUCCEEDED` or confirmed (before PR #4 this was a 500).

### `POST /prescriptions/:jobId/confirm`
Header: `X-Timezone-Offset` (optional). Body (`PatientPrescriptionReviewSchema`; the patient's reviewed and edited draft)
```ts
{
  documentId?: string,               // or document_id; defaults to the job's document
  prescription_date?: string,        // 'YYYY-MM-DD' or 'D/M/YYYY' / 'D-M-YYYY'; anything else is dropped
  date_is_uncertain?: boolean,       // default false
  doctor_name?, clinic_hospital_name?, documented_diagnosis?,
  medicines: Array<{                 // at least 1
    id?: string,                     // client UUID for the item, optional
    name_as_written: string (min 1),
    normalized_medicine_name?, strength?, frequency_text?, duration_text?, instructions?,
    dosage_form: DosageForm,         // required here
    route: Route,                    // required here
    is_active_medication?: boolean   // default false: historical prescriptions stay inactive
  }>
}
```
This runs in one transaction. It creates the Prescription, its items, a Medication for each item and a verified TimelineEvent, and it marks the job confirmed and the document `CONFIRMED`.
- Items with `is_active_medication: false` become `INACTIVE` medications, with **no reminders and no doses**.
- Items with `is_active_medication: true` become `ACTIVE`, with default reminders `08:00` and `20:00` and 7 days of `PENDING` doses. The app should let the patient pick the times, then call `PATCH /medications/:id/schedule`.
- The timeline event's date is `prescription_date`, or today (UTC) when none is given.

**201**
```ts
{ data: { prescription: Prescription, items: PrescriptionItem[], timelineEvent: TimelineEvent } }
```
Errors:
- 400 `VALIDATION_ERROR`, with `fields` **(PR #4)** and `details` = zod `format()` output (nested by field).
- 400 `BAD_REQUEST`: the job is already confirmed, or isn't `SUCCEEDED`.
- 404 `NOT_FOUND`: the job or document is missing or another patient's.

### `GET /prescriptions/timeline`
**200**: `{ data: TimelineEvent[] }`, newest first, not paginated. This is the older list; prefer `GET /timeline`.

### `GET /prescriptions/:prescriptionId`
**200**: `{ data: { prescription: Prescription, items: PrescriptionItem[] } }`
Errors: 404 `NOT_FOUND`.

---

## Medications: `/medications`

Every `:id` and `:doseId` must be a UUID, or the call gets **400 `INVALID_ID`**. Validation failures are **400 `VALIDATION_ERROR`**, with `fields` **(PR #4)** and `details` = zod `format()` output. State-rule failures are **400 `INVALID_ACTION`**. A missing medicine or one belonging to another patient is **404 `NOT_FOUND`**. Successful mutations also return a top-level `message`.

### `POST /medications` (add by hand)
Header: `X-Timezone-Offset`. Body
```ts
{
  nameAsWritten: string (min 1),
  dosageForm?: string,               // default 'TABLET'; free text here, not the enum
  strength?: string, route?: string, // route default 'ORAL'
  prescribedInstructions?: string,
  reminderTimes?: string[],          // 'HH:mm'
  startDate?: 'YYYY-MM-DD',          // default today (UTC)
  endDate?: 'YYYY-MM-DD',
  notes?: string (≤500)
}
```
With `reminderTimes`, the medicine is `ACTIVE` and 7 days of doses are made; without them it is `INACTIVE`.
**201**: `{ data: Medication, message }`
Errors: 400 `VALIDATION_ERROR`; 400 `BAD_REQUEST` (the name is blank after trimming).

### `GET /medications`
Query: `status?` = `INACTIVE | ACTIVE | PAUSED | COMPLETED | DISCONTINUED`
**200**: `{ data: Medication[] }`, newest first.
Errors: 400 `INVALID_STATUS`.

### `GET /medications/doses`
Query: `from?`, `to?` (anything `new Date()` parses; ISO is best). Header: `X-Timezone-Offset`.
The default window runs from today 00:00 UTC to 7 days later at 23:59:59 UTC. The window can be **30 days at most**. The call also tops up `PENDING` doses for active medicines inside the window.
**200**: `{ data: DoseLog[] }`, sorted by `scheduledTime`, oldest first.
Errors: 400 `INVALID_DATE` (a date doesn't parse, or **(PR #4)** the window is over 30 days; before PR #4 the second case was a 500).

### `POST /medications/doses/:doseId/action`
Only an explicit tap by the patient calls this. Never call it from a notification being delivered, dismissed or snoozed.
Body
```ts
{ action: 'TAKEN' | 'SKIPPED' | 'SNOOZED' | 'MISSED',   // MISSED: (PR #9)
  notes?: string (≤500),
  snoozeMinutes?: int 5–120,          // SNOOZED only; default 15
  actionTime?: ISO datetime,          // default now. (PR #9) must not be in the future (2 minutes allowed for clock drift)
  skipReason?: SkipReason }           // (PR #9) SKIPPED only; optional
```
Each call replaces the dose's earlier action: fields it doesn't set (snooze, skip reason, notes) are cleared.
**200**: `{ data: DoseLog, message }`
Errors:
- 400 `INVALID_ID`.
- 400 `VALIDATION_ERROR`: also **(PR #9)** for an `actionTime` in the future (`fields[0].path` = `actionTime`), an unknown `skipReason`, or a `skipReason` with any action but `SKIPPED` (`path` = `skipReason`).
- 400 `INVALID_ACTION` **(PR #9)**: `MISSED` on a dose that isn't due yet (`scheduledTime` in the future).
- 404 `NOT_FOUND`.

### `PATCH /medications/doses/:doseId/action` (correct a mistake)
Body: `{ action: 'TAKEN' | 'SKIPPED' | 'MISSED' | 'PENDING', notes?: string (≤500), skipReason?: SkipReason }`. `MISSED` and `skipReason` are new in **(PR #9)**; `skipReason` is for `SKIPPED` only.
- `TAKEN`, `SKIPPED` and `MISSED` set `actionTime` to now.
- `PENDING` is **Undo**. **(PR #9)** It clears `actionTime`, `skipReason`, `snoozeMinutes` and `notes`, so the dose is Not confirmed again, with no trace of Taken or Missed. Before PR #9, `actionTime` became now.

**200**: `{ data: DoseLog, message }`
Errors: as for POST (there is no `actionTime` to check here).

### `GET /medications/:id`
**200**: `{ data: MedicationDetail }`
Errors: 400 `INVALID_ID`; 404 `NOT_FOUND`.

### `GET /medications/:id/document-url`
**200**: `{ data: SignedDocumentUrl }`
Errors: 400 `INVALID_ID`; 404 `NOT_FOUND` (no medicine, no source document, or the document is unavailable).

### `POST /medications/:id/activate`
Header: `X-Timezone-Offset`. Body: `{ reminderTimes: string[] ('HH:mm', min 1), startDate?: 'YYYY-MM-DD', endDate?: 'YYYY-MM-DD' }`
Sets the status to `ACTIVE` and makes 7 days of doses.
**200**: `{ data: Medication, message }`
Errors: 400 `INVALID_ID` or `VALIDATION_ERROR`; 400 `INVALID_ACTION` (the medicine is `COMPLETED` or `DISCONTINUED`); 404 `NOT_FOUND`.

### `PATCH /medications/:id/schedule`
Header: `X-Timezone-Offset`. Body: as for activate.
Cancels future `PENDING` doses, then rebuilds 7 days of them if the medicine is `ACTIVE`. It doesn't change the status.
**200**: `{ data: Medication, message }`
Errors: as for activate.

### `POST /medications/:id/pause`
No body. The medicine must be `ACTIVE`. Cancels future `PENDING` doses.
**200**: `{ data: Medication, message }`. Errors: 400 `INVALID_ID` or `INVALID_ACTION`; 404 `NOT_FOUND`.

### `POST /medications/:id/resume`
Header: `X-Timezone-Offset`. No body. The medicine must be `PAUSED` and have reminder times. Rebuilds 7 days of doses.
**200**: `{ data: Medication, message }`. Errors: as for pause.

### `POST /medications/:id/complete`
No body. Fails if the medicine is already `COMPLETED` or `DISCONTINUED`. Sets `completedAt` and cancels future `PENDING` doses.
**200**: `{ data: Medication, message }`. Errors: as for pause.

### `POST /medications/:id/discontinue`
Body: `{ reason?: string (≤500) }`. An invalid body is ignored, not rejected. Fails if the medicine has already ended. Sets `discontinuedAt` and `discontinuedReason`, and cancels future `PENDING` doses.
**200**: `{ data: Medication, message }`. Errors: as for pause.

---

## Timeline: `/timeline`

This list shows only **verified** events.

### `GET /timeline`
Query (all optional, all strings)
| Param | Rule |
|---|---|
| `search` | Case-insensitive match on the title, summary, doctor, clinic, diagnosis and medicine names (as written or normalised) |
| `year` | 1900–2100 |
| `category` | `ALL`, `PRESCRIPTION` (also matches `PRESCRIPTION_RECORDED`), or an exact `eventType` |
| `limit` | 1–50, default 20 |
| `offset` | 0 or more, default 0 |

**200** (note: `pagination` sits beside `data`)
```ts
{ data: TimelineEventDetail[], pagination: { totalCount, limit, offset, hasMore: boolean } }
```
Sorted by `eventDate`, newest first, then by `createdAt`, newest first.
Errors: 400 `INVALID_QUERY_PARAMETERS`, with `fields` **(PR #4)** and `details` = zod `format()` output.

### `GET /timeline/:id`
**200**: `{ data: TimelineEventDetail }`
Errors: 400 `INVALID_ID`; 404 `EVENT_NOT_FOUND`.

### `GET /timeline/:id/document-url`
**200**: `{ data: SignedDocumentUrl }`
Errors: 400 `INVALID_ID`; 404 `DOCUMENT_NOT_FOUND` (no event, no source document, or the document is unavailable).

---

## Reports: `/reports`

The three POST routes take the same body (`exportConfigSchema`):
```ts
{
  title?: string (≤100),             // default 'Medical Summary Report'
  startDate?: string, endDate?: string,   // 'YYYY-MM-DD' or ISO; start must not be after end
  includeTimeline?: boolean,         // default true
  includeMedications?: boolean,      // default true
  selectedEventIds?: uuid[], selectedMedicationIds?: uuid[],
  allowEmpty?: boolean               // allow a report with 0 records
}
```
Only verified records are included: timeline events with `isVerified` (up to 500), and medicines, which exist only through a confirm or a manual add. AI drafts are never included.

**ExportData**
```ts
{
  patient: { patientProfileId, fullName },   // (PR #4) the patient's own name; '' when none was given. Before PR #4 always 'Test Patient'
  reportTitle, generatedAt,
  dateRange: { startDate: string | null, endDate: string | null, label },
  summary: { timelineEventsCount, medicationsCount, totalRecords },
  timelineEvents: TimelineEventDetail[],      // oldest first
  medications: MedicationDetail[]             // ACTIVE first, then by name
}
```

### `POST /reports/export-data`
**200**: `{ data: ExportData }`
Errors: 400 `VALIDATION_ERROR`: schema failures, with `fields` **(PR #4)** and `details` = the zod `errors` array; or a bad date or range, without `details` and with `fields` naming `startDate` or `endDate` **(PR #4)**.

**(PR #4)** The PDF prints the same name, or "Name not recorded" when there is none, and its footer says Ayoreni. The PDF fonts are Latin-only: a name in another script prints "Name on file (script not supported in PDF)".

### `POST /reports/generate`
Builds the PDF and stores it as a patient document (so it shows up in `GET /documents`).
**201**
```ts
{ data: {
    reportId,                          // = the stored Document id
    title, filename, mimeType: 'application/pdf', fileSizeBytes, pageCount, generatedAt,
    summary: { timelineEventsCount, medicationsCount, totalRecords },
    downloadUrl,                       // signed, 15 minutes
    pdfBase64
} }
```
Errors: 400 `VALIDATION_ERROR` (schema, or **(PR #4)** a bad date range; before PR #4 a bad range gave `EMPTY_REPORT_ERROR`); 400 `EMPTY_REPORT_ERROR` (no records and `allowEmpty` not set).

### `POST /reports/generate-pdf`
**200**: the raw PDF bytes, `Content-Type: application/pdf`, `Content-Disposition: inline; filename="medical-report.pdf"`. Nothing is stored.
Errors: 400 `VALIDATION_ERROR` (schema, or a bad date range); 400 `EMPTY_REPORT_ERROR`.

### `GET /reports/:id/download`
**200**: the stored bytes, `Content-Type: application/pdf`, `Content-Disposition: attachment; filename="<originalFilename>"`.
Errors: 400 `INVALID_ID`; 404 `NOT_FOUND` (missing, another patient's, or **(PR #4)** not a generated report: an uploaded PDF or photo is 404 here; use `GET /documents/:documentId/signed-url` for those).

---

## Not for the app

These routes exist in `src/app.ts` but are scaffolding. Don't build against them.
- `GET /patient/profile` returns `{ data: { patientProfileId, userId, email, fullName: 'Test Patient', createdAt } }`, with a hard-coded name and timestamp. Use `GET /auth/me`.
- `GET /patient/:patientProfileId/records` is a fixture for the cross-tenant test. It returns `{ data: { patientProfileId, records: [] } }`, or 404 for another patient's ID.

## Known gaps (for later phases)

- There is no server endpoint for an as-needed (PRN) dose; the app saves those on the phone only (Phase 4 review, finding 2).
- A retried prescription confirm, after the first response was lost, returns **400 `BAD_REQUEST`** ("already been confirmed"), not the created records (Phase 4 follow-up). A true race on the unique constraint can still surface as 500.
- Found while writing this contract on 19 November 2026. **All seven are fixed in draft PR [jai38/myhealthapp-backend#4](https://github.com/jai38/myhealthapp-backend/pull/4)**, which isn't merged or deployed yet. Until it is, production still does this:
  1. `POST /documents/confirm-upload` with another patient's `storageKey` returns **403 `FORBIDDEN`**. Rule 4 says it should be 404.
  2. `POST /prescriptions/extract-job/:jobId/cancel` on a `SUCCEEDED` or confirmed job returns **500**. The route checks for a different error message than the repository throws.
  3. `GET /medications/doses` with a window over 30 days returns **500**. The service sets 400, but the route doesn't map it.
  4. Report PDFs and `export-data` print the patient name as **"Test Patient"** for every user, and the PDF footer says "My Health App".
  5. `GET /reports/:id/download` streams **any** of the patient's documents (photos too) labelled as `application/pdf`, not only reports.
  6. Validation error shapes differ: auth gives 400 with no `details`; most others give 400 with zod `format()` `details`; reports give the zod `errors` array; the global handler gives 422.
  7. A malformed JSON body gives 500, not 400.
- Still open after PR #4:
  - The PDF draws text with a Latin-only font, so Devanagari text in medicine names, doctors or notes becomes blank. Only the patient-name line says so.
  - The verify-email message still says "My Health App" (`src/modules/auth/email-service.ts`).
  - `GET /patient/profile` (scaffolding, "Not for the app") still returns "Test Patient".
