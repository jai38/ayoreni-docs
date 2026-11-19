# Security, Privacy & PHI Compliance Review — My Health App

**Document Version:** 2.0.0  
**Last Updated:** 2026-10-12  
**Status:** Living Master Document (Updated for Phase 2 Verification)  

---

## 1. Compliance Frameworks & Regulatory Posture

My Health App handles sensitive patient medical data, including uploaded prescriptions, clinical diagnoses, and ongoing medication records. The system architecture is designed to comply with:
1. **India Digital Personal Data Protection (DPDP) Act, 2023:** Principles of notice, purpose limitation, verifiable consent, data minimization, and verified data erasure.
2. **HIPAA Security & Privacy Rules (International Healthcare Benchmark):** Strict access control, audit logs, encryption at rest/in transit, and contractual protections with third-party processors.

---

## 2. Inviolable Security & Medical Safety Invariants

1. **Anti-Spoofing Ownership Invariant:**
   - **Never trust a client-provided `patient_profile_id` as proof of ownership.**
   - All server routes resolve patient identity directly from the cryptographically verified JWT session claims (`req.authContext = { userId, patientProfileId }`).
   - Any query attempting to access or modify records enforces `WHERE patient_profile_id = req.authContext.patientProfileId`. Cross-user access returns `404 Not Found` to prevent ID enumeration.
2. **Private Document Storage & Magic Byte Validation:**
   - Public read/write access to object storage buckets is permanently blocked.
   - Upload and download operations are strictly gated behind short-lived pre-signed URLs (maximum 15-minute expiration) generated only after server-side ownership verification.
   - File uploads validate actual magic bytes (JPEG `FF D8 FF`, PNG `89 50 4E 47`, PDF `%PDF`) to reject disguised malicious binaries.
   - Server calculates and verifies SHA-256 checksums rather than trusting client-provided checksums alone.
3. **Medical Truth & Provenance Invariants:**
   - The original uploaded document is preserved immutably as evidentiary provenance.
   - Medicine names as written on the prescription are preserved separately from any normalized medicine name.
   - Ambiguous, uncertain, or missing fields are explicitly tagged (`confidence: 'UNCERTAIN'`, `date_is_uncertain: true`) and never silently guessed or hallucinated.
   - **Historical Prescription Inactivation Rule:** Historical prescriptions must not automatically become active medication regimens. During review, `is_active_medication` defaults to `false`.
   - **No Unreviewed AI Truth:** Unreviewed AI extractions remain strictly in draft state (`ai_extraction_jobs`) and never appear in the verified clinical timeline (`medical_events`).
4. **Zero Secrets in Mobile Client:**
   - The React Native mobile bundle contains zero database connection strings, S3 master secret keys, or third-party AI credentials.
5. **Decoupled Notification Delivery:**
   - Operating system notifications are ephemeral delivery alerts. A notification being delivered, displayed, dismissed, or snoozed must **never** automatically mark a dose as taken.

---

## 3. Data Classification & Protection Controls

| Classification Level | Examples | Storage Location | Protection Measures |
| :--- | :--- | :--- | :--- |
| **Level 3: Sensitive PHI** | Prescription images, diagnosed conditions, medication schedules, doctor notes, dose logs | Private S3 Bucket / PostgreSQL DB | AES-256 encryption at rest, Row-Level Security, short-lived signed URLs, zero stdout logging |
| **Level 2: Patient PII** | Full name, phone number, email, date of birth, emergency contact | PostgreSQL DB | Encrypted at rest, access-controlled by authenticated user session |
| **Level 1: System Data** | Session tokens, audit log events, app version, error codes | Redis / SecureStore / DB | Opaque UUIDs only, no medical context embedded |

---

## 4. Threat Modeling & Core Defense Controls

### 4.1 Tenant Isolation & IDOR Prevention
- **Threat:** User A attempts to view, edit, or delete User B's prescriptions or download User B's uploaded files.
- **Defense:**
  1. Primary Barrier: Application service layer forces tenant filtering on all database and storage operations.
  2. Storage Key Isolation: Opaque keys format: `documents/<patientProfileId>/<randomUUID>`. Any request attempting to confirm or access keys outside the authenticated patient path is rejected with `403 Forbidden`.
  3. API Barrier: Endpoints return `404 Not Found` when a resource exists but belongs to another patient, preventing resource existence enumeration.

### 4.2 Data Deletion, Retention & Backup Limitations
- **Account Deletion Cascade:**
  - Hard-delete transaction purges user credentials, patient profile, all relational records, and issues asynchronous deletion commands to object storage.
- **Backup Deletion Limitations (Honest Compliance Posture):**
  - Immutable database backups cannot be dynamically edited to excise deleted records without compromising cryptographic backup integrity.
  - **Policy:** Backups operate on a rolling 30-day retention window. Deletions apply immediately to live datastores and object storage; backup copies age out and are permanently destroyed as retention windows expire. A deletion tombstone log is maintained to re-purge any restored data during disaster recovery drills.

### 4.3 Third-Party AI Data Handling & Privacy
- **Threat:** Third-party vision model providers retaining patient health documents to train public AI models.
- **Defense:**
  1. Only model providers with contractually verified zero-training and non-retention agreements on our active commercial account will be used in production.
  2. The backend strips all unnecessary patient PII (user ID, phone, email) from API payloads before sending document images for extraction.
  3. Raw multimodal prompt payloads are never logged to routine application logs.
  4. Phase 2 development and testing used the deterministic `MockPrescriptionExtractionProvider` and synthetic benchmark dataset, sending zero patient data over external networks.

### 4.4 Audit Trail & Log Sanitization
- Critical lifecycle events (`ACCOUNT_REGISTERED`, `LOGIN_SUCCESS`, `PRESCRIPTION_CONFIRMED`, `SCHEDULE_CREATED`, `REPORT_EXPORTED`, `ACCOUNT_PURGED`) write to an append-only `audit_events` table.
- **Sanitization Rule:** Logs record only actor UUID, target resource type, target resource UUID, action, IP address, and timestamp. **Zero patient names, drug names, diagnoses, or dosages are ever copied into audit logs.**

### 4.5 Production Fail-Closed Configuration Controls (`src/config/env.ts`)
- **Refuse Insecure Modes:** `validateProductionConfig` strictly refuses startup if:
  - `AUTH_PROVIDER === 'mock'` (mock auth restricted to isolated dev/test).
  - `JWT_SECRET` contains default dev tokens or is shorter than 32 characters.
  - `DATABASE_URL` references localhost or default dev credentials (`healthpassword`).
  - `STORAGE_ACCESS_KEY` or `STORAGE_SECRET_KEY` use default dev values (`minioadmin`).
- **Secret Sanitization:** Configuration error messages describe validation criteria generically without printing passwords, connection strings, or patient data.

### 4.6 Automated Security & Concurrency Test Suite Verification
- `test/storage-and-upload.test.ts` (16 passing):
  - Unsupported MIME types (.exe, .zip, .txt) rejected with 400.
  - Files exceeding 15MB limit rejected at intent and confirm stages.
  - Absent or zero-byte uploads fail closed with 400 (fabricated PDF fallback eliminated).
  - Declared size mismatch between upload intent and stored object rejected with 400.
  - Bounded streaming memory limits and 5s parser execution timeout enforced.
  - Spoofed file magic bytes rejected.
  - Deep parser validation rejects fake PDF headers, encrypted PDFs, and PDFs exceeding 10 pages (`pdf-lib`).
  - Deep parser validation rejects corrupt image headers and images outside 50x50 to 8000x8000 dimensions (`image-size`).
  - Server-calculated SHA-256 rejects client-mismatched or tampered files.
  - Cross-user upload confirmation rejected with `403 Forbidden`.
  - Cross-user download/preview rejected with `404 Not Found` to prevent ID enumeration.
  - Duplicate confirmation of same intent prevented via atomic consumption (mutual exclusion).
  - Upload intent survives simulated backend restart across service reinitialization.
  - Production config validation enforces fail-closed startup on mock auth and insecure credentials.
- `test/extraction-pipeline.test.ts` (11 passing):
  - Idempotent request handling on duplicate submissions.
  - Atomic job claiming (`claimJobForProcessing`) ensures mutual exclusion across concurrent workers.
  - Concurrency-safe job cancellation prevents late provider responses from overwriting terminal states.
  - Stale `PROCESSING` jobs recovered safely after worker interruption.
  - Unreviewed AI extractions never appear in verified timeline.
  - Bounded retries (max 3) preventing infinite extraction loops.
  - Atomic database transaction rolls back all clinical writes upon partial confirmation failure.
  - Unique constraint on `jobId` in `prescriptions` prevents duplicate prescriptions or timeline events across concurrent requests or restarts.
  - Cross-user job inspection rejected with `404 Not Found`.
- `test/upload-intent-lifecycle-and-parser-limits.test.ts` (17 passing):
  - Storage key UUID enforcement: accepts `documents/<patientProfileId>/<uuid>`, rejects non-UUID final segments, traversal, and malformed paths.
  - Upload intent state transitions: `PENDING` -> `VERIFYING` -> `VERIFIED` / `REJECTED`.
  - Concurrency lock lease: `acquireVerificationLock` prevents concurrent finalization races.
  - Safe client retry on transient failures: object not yet uploaded or transient S3 read error releases lock back to `PENDING`.
  - Permanent failure fails closed: invalid bytes, size mismatch, or expired lease transitions to `REJECTED`.
  - Duplicate confirmation protection: already-finalized intent cannot be finalized again.
  - Parser worker isolation & timeout: worker thread termination (`worker.terminate()`) halts runaway PDF/image parsers.
  - Parser concurrency bounds: max 4 workers, queuing excess tasks.
- `test/ai-benchmark.test.ts` (5 passing):
  - Provenance and permitted-use metadata verified on all 32 synthetic fixtures.
  - Deterministic test doubles verified without claiming empirical AI accuracy.
  - Detection of medical safety rule violations (hallucinations, dosage transposition, unflagged uncertainty).
- Mobile `test/prescription-review.test.ts` (4 passing):
  - AI extraction treated strictly as unverified draft.
  - Active medication defaults to false for historical prescriptions.
  - Separate preservation of `name_as_written` vs `normalized_medicine_name`.
  - Validation requires non-empty medicine names.
- `test/integration.postgres-minio.test.ts` (6 tests defined, reported as **NOT RUN / SKIPPED** when local container ports 5432 & 9000 are closed):
  - Full transactional integration suite ready for disposable container CI environment.



