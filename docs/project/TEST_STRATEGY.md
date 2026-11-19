# Test Strategy & Quality Assurance Framework — My Health App

**Document Version:** 2.0.0  
**Last Updated:** 2026-10-12  
**Status:** Living Master Document (Updated for Phase 2 Completion)  

---

## 1. Testing Philosophy

Health applications demand absolute data integrity and patient safety. An error in medication recurrence calculations, an accidental dose state mutation, or an authorization bypass in document storage can directly impact patient health and privacy.

We enforce a multi-layered testing pyramid where every critical business rule is validated by automated regression suites before code reaches a pull request.

```
          ▲
         / \     Physical Device Verification (Android / iOS)
        /   \    E2E Flows (Detox / Maestro / Manual Scripts)
       /─────\
      /       \   AI Extraction Indian Benchmark (32 Fixtures) & Mock Suite
     /         \  API Integration & Cross-Tenant IDOR Security Tests
    /───────────\
   /             \ Unit Tests: File Validator, Schedulers, Zod Schemas
  /───────────────\
```

---

## 2. Test Layers & Current Execution Status

| Test Level | Scope | Framework / Tool | Executed / Passing | Status |
| :--- | :--- | :--- | :---: | :---: |
| **Unit Tests (Mobile)** | `DiscreteOccurrenceScheduler`, `ApiClient`, `FileValidator`, `UploadService` | Vitest | 16 / 16 | **PASS** |
| **Backend Integration & Auth** | Auth middleware, ownership derivation, IDOR isolation, health probes | Supertest + Vitest | 6 / 6 | **PASS** |
| **Storage & Deep Parser Security**| Fail-closed validation, direct pre-signed upload, SHA-256 integrity, declared-size check, bounded streaming, 5s timeout, 15MB limit, `pdf-lib` parser (corrupt, encrypted, <=10 pages), `image-size` parser (corrupt headers, 50-8000px bounds), cross-user preview 404, restart survival, production fail-closed validation | Vitest + Supertest | 16 / 16 | **PASS** |
| **Extraction Pipeline & Concurrency**| Atomic job claiming (`claimJobForProcessing`), concurrency safe cancellation, late response protection, bounded retries (max 3), zombie job recovery, transactional confirmation, reinitialization survival | Vitest + Supertest | 11 / 11 | **PASS** |
| **AI Benchmark & Mock Contract** | Provenance verification, safety rule enforcement, deterministic test-double contract, harness scoring decoupled from AI accuracy | Custom BenchmarkEvaluator + Vitest | 5 / 5 (32 fixtures) | **PASS** |
| **Durable PostgreSQL & MinIO Integration** | Clean migration check, test patient/doc creation, job claim/complete, atomic rx confirmation, forced tx rollback verification, concurrent race condition, real MinIO direct PUT and verification | Vitest + Prisma + MinIO | 0 / 6 (6 skipped) | **NOT RUN** (Docker offline) |
| **Total Unit & Contract Tests** | Across both sub-repositories (excluding skipped live integration) | Vitest | **54 / 54** | **PASS** |

---

## 3. Specific Critical Test Suites

### 3.1 AI Extraction Evaluation Benchmark & Mock Contract Suite (`test/ai-benchmark.test.ts`)
- **Benchmark Honesty & Provenance Invariants:**
  - Validates that every one of the 32 fixtures in `modules/ai-extraction/benchmark/dataset.ts` possesses complete provenance metadata (`origin`, `license`, `provenance_notes`, `permitted_use`).
  - Confirms permitted use explicitly covers clinical extraction evaluation and training exclusion.
- **Mock Provider Contract Testing:**
  - `MockPrescriptionExtractionProvider` validated as a deterministic test double for CI and local development.
  - Verifies that mock fixture matching is classified strictly as a harness contract test and is **never** conflated with empirical AI accuracy.
- **Safety Violation Detection:**
  - Evaluator proves it catches hallucinated medications, transposed dosages, unflagged uncertain fields, and missing dates.
- **Live AI Integration Disabled:**
  - Third-party AI APIs remain disabled until enterprise DPDP/BAA privacy agreements are verified.

### 3.2 Storage Security, Fail-Closed Upload & Deep Parser Suite (`test/storage-and-upload.test.ts`)
- **Fail-Closed Private Storage:**
  - Upload intent creates tracked `UploadIntent` with 15-minute expiration.
  - Client must upload bytes directly to pre-signed URL.
  - Fabricated PDF fallback completely removed: confirming an upload without stored bytes fails closed with `400 Bad Request`.
  - Server recalculates SHA-256 checksum and rejects mismatched payloads with `400 Bad Request`.
- **Deep Parser-Based File Validation:**
  - `pdf-lib` inspects PDF byte stream: rejects fake PDF headers containing non-PDF payloads (`400 Bad Request`).
  - Rejects PDFs exceeding 10 pages (`400 Bad Request`).
  - Rejects password-protected or encrypted PDFs (`400 Bad Request`).
  - `image-size` inspects image streams: rejects truncated/corrupt JPEG/PNG images (`400 Bad Request`).
  - Rejects images outside dimension boundaries (minimum 50x50, maximum 8000x8000).
- **Tenant Isolation & Anti-Tampering:**
  - Cross-user upload confirmation (User A attempting to confirm User B's storage key) rejected with `403 Forbidden`.
  - Cross-user preview signed URL returns `404 Not Found` to prevent resource existence enumeration.

### 3.3 Extraction Lifecycle, Concurrency & Transactional Persistence (`test/extraction-pipeline.test.ts`)
- **Atomic Job Claiming & Concurrency Protection:**
  - Multiple concurrent worker invocations attempting `claimJobForProcessing()` on the same queued job allow exactly ONE worker to claim; subsequent worker claims return `null`.
  - Late provider responses arriving after a job has been CANCELLED or FAILED are prevented from overwriting terminal states.
- **Fault Recovery & Bounded Retries:**
  - Stale `PROCESSING` jobs left behind by interrupted or crashed workers are recovered via `recoverStaleProcessingJobs()`.
  - Bounded retries: max 3 attempts before final `FAILED` status; original document preserved immutably.
- **Durable Relational Records & Transactional Confirmation:**
  - Extraction jobs and clinical records persist in PostgreSQL via Prisma models (`Document`, `UploadIntent`, `ExtractionJob`, `Prescription`, `PrescriptionItem`, `TimelineEvent`).
  - Confirmation executes in a single database transaction: writes prescription, items (preserving exact written medicine name), and verified timeline event.
  - Unique constraint on `jobId` in `prescriptions` prevents duplicate confirmation across concurrent requests, server restarts, and instances.
  - Partial write failures roll back all clinical records.
  - Non-negotiable medical safety rules enforced: unreviewed AI extractions never appear on timeline; historical prescriptions default to `is_active_medication: false`.
  - Records and job states survive service reinitialization.

### 3.4 Live Datastore Integration Suite (`test/integration.postgres-minio.test.ts`)
- **Honest Test Reporting Policy:**
  - Socket probes or skipped tests are **never** counted as passing integration tests.
  - When PostgreSQL (port 5432) or MinIO (port 9000) are offline, tests are skipped via `describe.skipIf` and reported honestly as **NOT RUN / SKIPPED**.
- **Genuine Clinical Integration Scenarios (when containers are active):**
  1. *Clean Migrations & Patient/Doc Records:* Verifies clean schema migrations and durable creation of test user, patient profile, and document records.
  2. *Extraction Job Persistence Across Client Reinitialization:* Creates queued job, claims it atomically, completes it with structured clinical payload, and verifies payload across fresh Prisma client reinitialization.
  3. *Atomic Prescription Confirmation Transaction:* Atomically commits prescription, medicine items, timeline event, job confirmation, and document status in a single database transaction. Verifies records across fresh Prisma client.
  4. *Forced Transaction Failure & Rollback Proof:* Intentionally triggers a transaction error; queries database directly to prove zero partial clinical records remain.
  5. *Concurrency Race on Prescription Confirmation:* Races two simultaneous confirmation requests for the same job; verifies exactly one succeeds and exactly one prescription/timeline event exists.
  6. *Genuine MinIO Direct PUT, Verification & Cleanup:* Performs client PUT to presigned MinIO URL, executes server-side validation via `S3StorageProvider` and durable `PrismaUploadIntentRepository`, verifies intent consumption in PostgreSQL, and cleans up object.

---

## 4. Quality Gates for Merging & Deployment

1. **Local Pre-Commit:** `npm run typecheck && npm test` cleanly passes in both `myhealthapp/` and `myhealthapp-backend/` (54/54 unit/contract tests green, 0 compiler errors, integration honestly reported as NOT RUN when containers offline).
2. **CI Pipeline:** All unit, deep parser, concurrency, and persistence suites run against isolated test doubles with deterministic reproducibility.
3. **Pre-Release Gate:** Zero failed security tests, verified local notification triggers on physical Android/iOS devices.

