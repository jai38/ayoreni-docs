# ADR-007: Document Storage, Privacy Lifecycle & Data Retention Policies

**Date:** 2026-10-12  
**Status:** accepted (Remediated & Verified in Phase 2)  
**Deciders:** Principal Architect & Security Engineer  

---

## Context

Patients upload sensitive medical documents (prescriptions, clinical notes, discharge summaries) and generate health summary reports in PDF format.
Requirements:
1. Documents must remain completely confidential and inaccessible to unauthorized actors.
2. Raw binary files must never be stored inside the relational database or exposed via public web links.
3. Upload and download workflows must be protected by server-side authorization checks.
4. Fail-closed security: missing uploads, absent storage objects, or corrupted files must never be accepted or fabricated.
5. Deep parser-based validation: prevent disguised executables, zip-bombs, corrupted files, and encrypted files.
6. Comprehensive privacy lifecycles: Define hard deletion, retention, backup retention constraints, audit trails, and third-party AI data handling under Indian DPDP and global health privacy principles.

---

## Decision

### 1. Private S3-Compatible Object Storage Abstraction & S3StorageProvider
- All document binaries and generated PDF exports are stored in private object storage (`S3StorageProvider` connecting to AWS S3, MinIO for local development, or Cloudflare R2).
- `MemoryStorageProvider` is retained strictly as an explicitly injected test double for isolated unit testing.
- Storage access is abstracted behind a `StorageProvider` interface (`createUploadIntent`, `verifyAndFinalizeUpload`, `getDownloadSignedUrl`, `deleteObject`, `checkObjectExists`).
- Bucket policies permanently block all public read/write access.
- File storage keys are randomized opaque UUIDs strictly scoped to the tenant (`documents/<patientProfileId>/<randomUUID>`), where the final path segment MUST be a valid UUID. Any key with non-UUID segments, path traversal, or prefix/extensions is strictly rejected.

### 2. Genuine Pre-Signed URL Upload, Durable Intent State Machine & Fail-Closed Finalization
1. **Upload Intent Persistence in PostgreSQL (`UploadIntentStatus` Lifecycle):**
   - Client calls `POST /api/v1/documents/upload-intent` with `{ filename, mimeType, fileSizeBytes }`.
   - Backend derives opaque storage key (`documents/<patientProfileId>/<uuid>`).
   - Backend persists a durable `UploadIntent` record in PostgreSQL (`PrismaUploadIntentRepository`) storing `storageKey`, `patientProfileId`, `originalFilename`, `mimeType`, `expectedSizeBytes`, `status: PENDING`, and `expiresAt` with strict **15-minute expiration** (900 seconds).
   - Server restart does not invalidate still-valid intents.
   - Backend signs an AWS S3 / MinIO `PutObjectCommand` with matching 15-minute expiration and returns pre-signed URL.
2. **Direct Binary Transmission:**
   - Client PUTs raw binary bytes directly to the pre-signed storage URL.
   - Heavy payloads bypass the application backend entirely, avoiding high memory overhead and JSON base64 encoding.
   - Application JSON request body limit is restricted to **1MB** to prevent payload inflation and DoS attacks.
3. **Server-Side Finalization, Concurrency Lock & Retry Semantics:**
   - Client calls `POST /api/v1/documents/confirm-upload` with `{ storageKey, mimeType, clientChecksum }`.
   - **Atomic Verification Lock (`acquireVerificationLock`):** Atomically transitions intent from `PENDING` -> `VERIFYING` with a 30-second lock lease (`verifying_started_at`). A concurrent request or expired lock cannot concurrently process the intent.
   - **Already-Finalized Mutual Exclusion:** If status is `VERIFIED`, returns HTTP 400 (`Upload intent has already been consumed. Duplicate confirmation rejected.`) preventing duplicate document records. If `REJECTED`, returns HTTP 400.
   - **Cross-Patient & Expiry Guard:** Rejects intent if expired, belonging to another patient profile, or for a mismatched storage key.
   - **Declared Size Enforcement (Pre-Download):** Backend issues an S3 `HeadObjectCommand` to inspect the actual stored object size before downloading any bytes into memory.
     - If object is not yet uploaded (`NotFound` / 404) or storage operation fails transiently, the verification lock is released back to `PENDING` (`releaseVerificationLock`), enabling safe client retry.
     - If stored object size does not match declared `expectedSizeBytes`, or is 0 bytes, or exceeds 15MB: intent is marked `REJECTED` (`markRejected`) with reason, failing closed.
   - **Bounded Streaming Download:** If sizes match, the object is downloaded using bounded streaming (`readStreamBounded`) that terminates the stream if bytes exceed the limit, preventing unbounded memory consumption.
   - **Real Worker-Thread Resource Limits & Thread Termination:**
     - PDF and image validation tasks run in an isolated worker thread (`src/modules/storage/parser-worker.cjs`) managed by `ParserConcurrencyManager` (`MAX_CONCURRENT_PARSERS = 4`, max queue depth = 20).
     - Execution is bounded by an explicit 5000ms timeout. Upon timeout, the worker thread is forcefully terminated (`worker.terminate()`), guaranteeing that runaway or malicious PDF/image processing cannot consume CPU or memory indefinitely.
     - For PDF: parsed with `pdf-lib`. Rejects malformed structures, rejects encrypted/password-protected files, enforces a maximum limit of **10 pages**.
     - For Images: parsed with `image-size`. Rejects malformed image structures, enforces maximum dimension ceiling of **8000x8000 pixels** and minimum resolution of 50x50 pixels.
   - **Checksum Verification & Terminal State Transition:**
     - Server computes SHA-256 hash across stored bytes and matches client-provided checksum.
     - On validation success, intent transitions permanently to `VERIFIED` (`markVerified`).
     - An immutable `Document` record is written to the database (Prisma / PostgreSQL) containing verified file size, SHA-256 checksum, page count, and dimensions.

### 2.1 Production Configuration Fail-Closed Validation
- Application startup validates environment variables (`validateProductionConfig`).
- Refuses startup in production if `AUTH_PROVIDER === 'mock'`.
- Refuses startup in production if `JWT_SECRET` contains development defaults, insecure placeholders, or is <32 characters.
- Refuses startup in production if `DATABASE_URL` contains development credentials (`healthpassword`) or localhost addresses.
- Refuses startup in production if object storage credentials contain development defaults (`minioadmin`).
- Errors fail closed without logging secrets, connection strings, or patient identifiers.

### 3. Pre-Signed Download / Preview Flow
- Client calls `GET /api/v1/documents/:id/signed-url`.
- Backend verifies that the authenticated patient owns the document (`doc.patientProfileId === req.authContext.patientProfileId`).
- If unauthorized, returns **404 Not Found** (prevents cross-tenant document enumeration).
- Returns short-lived pre-signed GET URL with strict 15-minute expiration.

### 4. Encryption Controls
- **In Transit:** All communication over HTTPS enforcing TLS 1.3 (minimum TLS 1.2).
- **At Rest:** Server-Side Encryption with AES-256 (SSE-S3 or SSE-KMS) on all object storage buckets; volume-level AES-256 encryption on PostgreSQL datastores.

### 5. Data Retention, Deletion & Backup Limitations
1. **User Account Deletion & Right to Be Forgotten:**
   - Patient triggers account deletion -> System initiates an atomic purge transaction.
   - User credentials, patient profile, documents, prescriptions, timeline events, medications, schedules, and dose events are hard-deleted from the primary database.
   - Corresponding physical files in object storage are deleted via batch S3 `DeleteObjects` commands.
2. **Backup Deletion Limitations (Honest Compliance Posture):**
   - **Crucial Invariant:** Automated, encrypted database backups (daily snapshots and write-ahead transaction logs) are immutable.
   - Deleted patient records **cannot be immediately scrubbed from existing historical backups** without corrupting backup set integrity.
   - **Policy:** Backups operate on a rolling 30-day retention window. When an account is deleted, it is expunged immediately from the live operational database and object storage; historical backups naturally age out and are permanently destroyed as backup retention windows expire (within 30 days). If a disaster recovery restore is ever performed, a deletion tombstone log is re-applied during restoration to prevent restoring purged accounts.

### 6. Audit Logging Architecture
- An append-only, tamper-evident `audit_events` table records critical security and data lifecycle events:
  - `ACCOUNT_REGISTERED`, `LOGIN_SUCCESS`, `LOGIN_FAILED`, `DOCUMENT_UPLOADED`, `PRESCRIPTION_CONFIRMED`, `SCHEDULE_CREATED`, `REPORT_EXPORTED`, `ACCOUNT_PURGED`.
- **Sanitization Rule:** Audit logs capture only the actor UUID, target resource type, target resource UUID, action, IP address, and timestamp. **Zero medical data, medication names, dosages, or patient PII are ever copied into audit logs.**

---

## Consequences

### Positive
- Prevents IDOR, tenant spoofing, and public document enumeration.
- Eliminates 15MB base64 JSON payload vulnerabilities; application body limit constrained to 1MB.
- Deep parser validation prevents disguised executables, page-count overflows, and corrupt image attacks.
- Fail-closed verification guarantees that missing uploads never produce phantom documents.

### Negative
- Requires two-step upload handshake (intent -> direct PUT -> confirm).
- Requires pre-signed URL generation step before opening any document in the mobile previewer.
