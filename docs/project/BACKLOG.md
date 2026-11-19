# Master Project Backlog — My Health App

**Document Version:** 1.2.0  
**Last Updated:** 2026-10-12  

Statuses: `BACKLOG` | `READY` | `IN_PROGRESS` | `BLOCKED` | `IN_REVIEW` | `DONE`  
Priorities: `P0` (Critical/Blocker) | `P1` (High) | `P2` (Medium) | `P3` (Nice to have)  

---

## Phase 0: Discovery & Planning

### [DISC-001] Repository & Tooling Audit
- **Priority:** P0 | **Phase:** Phase 0 | **Dependencies:** None | **Status:** `DONE`
- **Description:** Physically inspect workspace directory `healthapp`, examine sub-repositories `myhealthapp` and `myhealthapp-backend`, verify git origins and available tooling (Node, npm, Docker, Git).
- **Acceptance Criteria:** `REPOSITORY_AUDIT.md` produced reflecting verified local environment findings.
- **Tests:** Verification via shell queries.

### [DISC-002] Tooling & Runtime Environment Review
- **Priority:** P0 | **Phase:** Phase 0 | **Dependencies:** DISC-001 | **Status:** `DONE`
- **Description:** Review package managers, Node 22 compatibility, Docker availability, and Windows OS constraints.
- **Acceptance Criteria:** Runtimes recorded with exact version numbers and compatibility assessed.
- **Tests:** Runtime version checks executed.

### [DISC-003] Architecture Alternatives & ADR Formulation
- **Priority:** P0 | **Phase:** Phase 0 | **Dependencies:** DISC-001 | **Status:** `DONE`
- **Description:** Formulate ADR-001 through ADR-007 covering mobile framework, backend hosting, database & ownership, auth, AI extraction, medication scheduling, and document storage.
- **Acceptance Criteria:** 7 complete ADRs created under `docs/architecture/adr/`.
- **Tests:** Architectural review against prompt requirements.

### [DISC-004] Relational Data Model & Ownership Design
- **Priority:** P0 | **Phase:** Phase 0 | **Dependencies:** DISC-003 | **Status:** `DONE`
- **Description:** Design complete relational schema (Users, Patient Profiles, Documents, Prescriptions, Prescription Items, Medical Events, Medications, Schedules, Dose Events, AI Jobs, Audit Logs) ensuring strict patient isolation and connection-pool safe RLS.
- **Acceptance Criteria:** Entity Relationship diagrams, table definitions, and constraints documented in ADR-003 and implementation plan.
- **Tests:** Relational integrity & foreign key constraint validation.

### [DISC-005] API & AI Contract Specification
- **Priority:** P0 | **Phase:** Phase 0 | **Dependencies:** DISC-004 | **Status:** `DONE`
- **Description:** Define versioned REST API endpoints and Zod schemas for AI extraction output and mobile consumption.
- **Acceptance Criteria:** Request/response schemas documented with zero tolerance for invented medical data.
- **Tests:** Schema validation tests.

### [DISC-006] Security, Privacy & PHI Compliance Review
- **Priority:** P0 | **Phase:** Phase 0 | **Dependencies:** DISC-003 | **Status:** `DONE`
- **Description:** Document security review covering tenant isolation, encryption at rest/in transit, zero secrets in mobile bundle, and compliance with DPDP & HIPAA principles.
- **Acceptance Criteria:** `SECURITY_REVIEW.md` authored.
- **Tests:** Threat model analysis.

### [DISC-007] UX Flows & Design System Proposal
- **Priority:** P1 | **Phase:** Phase 0 | **Dependencies:** DISC-003 | **Status:** `DONE`
- **Description:** Specify screen inventory, navigation structure, design tokens, and core flows (Prescription review, dose tracking, timeline, PDF export).
- **Acceptance Criteria:** UX specifications and design token architecture documented.
- **Tests:** Visual hierarchy and accessibility review against WCAG 2.2 AA.

### [DISC-008] Testing & Deployment Strategy
- **Priority:** P0 | **Phase:** Phase 0 | **Dependencies:** DISC-001 | **Status:** `DONE`
- **Description:** Document testing pyramid (Unit, Integration, AI Eval with Indian benchmark, Mobile device, Security tests) and release gates.
- **Acceptance Criteria:** `TEST_STRATEGY.md` and `RELEASE_CHECKLIST.md` authored.
- **Tests:** Test strategy verification.

---

## Phase 1: Application Foundation

### [FOUND-001] Mobile Application Shell & Navigation
- **Priority:** P0 | **Phase:** Phase 1 | **Dependencies:** Gate 1 Approval | **Status:** `DONE`
- **Description:** Initialize Expo app with TypeScript strict mode and Expo Router file-based navigation in `myhealthapp/`.
- **Acceptance Criteria:** App compiles, displays splash screen, and routes between tabs (`home`, `timeline`, `medications`, `profile`) with zero warnings.
- **Tests:** `npm run typecheck` passes with zero errors; Expo config verified.

### [FOUND-002] Design Tokens & Reusable UI Components
- **Priority:** P1 | **Phase:** Phase 1 | **Dependencies:** FOUND-001 | **Status:** `DONE`
- **Description:** Build theme tokens (colors, typography, spacing, radius) and reusable atomic components (Button, Card, Input, Badge, ScreenLayout).
- **Acceptance Criteria:** Consistent calm health-tech theme adhering to WCAG 2.2 AA contrast standards (minimum 48dp touch targets).
- **Tests:** Component typecheck and structural verification.

### [FOUND-003] Backend Environment & Scaffold
- **Priority:** P0 | **Phase:** Phase 1 | **Dependencies:** Gate 1 Approval | **Status:** `DONE`
- **Description:** Initialize Node.js TypeScript modular monolith in `myhealthapp-backend/` with Docker Compose (Node, PostgreSQL, MinIO) and environment validation.
- **Acceptance Criteria:** Backend starts on port with healthcheck endpoint `/healthz` returning 200 OK.
- **Tests:** Vitest backend integration test passes (`test/health.test.ts`).

### [FOUND-004] Authentication Integration & Adapter Architecture
- **Priority:** P0 | **Phase:** Phase 1 | **Dependencies:** FOUND-003, FOUND-005 | **Status:** `DONE`
- **Description:** Implement `AuthProviderAdapter` with session verification middleware, server-side identity derivation, and comparison documentation in ADR-004. Zero placeholder UI buttons.
- **Acceptance Criteria:** Verified token resolution derives `patientProfileId` exclusively server-side. Cross-tenant access rejected with 404/401.
- **Tests:** Vitest authorization suite passes (`test/auth-middleware.test.ts`).

### [FOUND-005] Database Migrations & Initial Schema
- **Priority:** P0 | **Phase:** Phase 1 | **Dependencies:** FOUND-003 | **Status:** `DONE`
- **Description:** Configure PostgreSQL schema with Prisma ORM for `users`, `patient_profiles`, and `user_preferences`.
- **Acceptance Criteria:** Schema enforces unique constraints and cascade deletes between user and patient profile.
- **Tests:** Prisma schema validated; relational constraints confirmed.

### [FOUND-006] Type-Safe API Client
- **Priority:** P0 | **Phase:** Phase 1 | **Dependencies:** FOUND-001, FOUND-003 | **Status:** `DONE`
- **Description:** Implement mobile API client with automatic authorization header injection, Zod response validation, and unified error handling.
- **Acceptance Criteria:** Network errors, 401 unauth, and 422 schema errors produce typed user-friendly exceptions.
- **Tests:** Vitest API client suite passes (`test/api-client.test.ts`).

### [FOUND-007] CI Checks & Automated Testing Framework
- **Priority:** P1 | **Phase:** Phase 1 | **Dependencies:** FOUND-001, FOUND-003 | **Status:** `DONE`
- **Description:** Configure Vitest, ESLint, Prettier, and TypeScript compilation check scripts for both repositories.
- **Acceptance Criteria:** `npm test` and `npm run typecheck` execute cleanly across both sub-repos with 14 passing automated tests.
- **Tests:** 14/14 automated tests passing across client and server.

### [FOUND-008] Sub-repository Documentation
- **Priority:** P2 | **Phase:** Phase 1 | **Dependencies:** FOUND-001, FOUND-003 | **Status:** `DONE`
- **Description:** Document setup, environment variables, Docker commands, and run scripts in `myhealthapp/README.md` and `myhealthapp-backend/README.md`.
- **Acceptance Criteria:** Developer onboarding guides verified for clean checkout.
- **Tests:** Manual documentation walkthrough.

---

## Phase 2: Prescription Pipeline & AI Evaluation (Remediated & Verified)

### [RX-001] Document Capture & Picker
- **Priority:** P0 | **Phase:** Phase 2 | **Dependencies:** FOUND-001 | **Status:** `DONE`
- **Description:** Mobile image capture via camera, photo gallery selection, and document picker for multi-page PDFs with size/MIME verification.
- **Acceptance Criteria:** Allowed formats (JPEG, PNG, PDF up to 15MB) accepted; invalid types rejected with clear UI feedback.
- **Tests:** File picker validation unit tests (5 passing).

### [RX-002] Secure Document Upload Flow, Durable Intents & Fail-Closed Storage
- **Priority:** P0 | **Phase:** Phase 2 | **Dependencies:** RX-001, FOUND-005 | **Status:** `DONE`
- **Description:** Pre-signed S3/MinIO upload flow. Removed fabricated PDF fallback; absent or missing upload bytes fail closed. Client performs direct binary PUT to pre-signed URL; server validates actual stored object (declared size match, MIME, SHA-256 checksum, intent expiry, ownership). S3StorageProvider uses durable PostgreSQL `UploadIntent` persistence (`PrismaUploadIntentRepository`) with restart survival and atomic mutual exclusion. Enforces declared size check via `HeadObjectCommand` pre-stream, bounded streaming, 5s parser execution timeout, and production fail-closed config validation.
- **Acceptance Criteria:** `S3StorageProvider` operates against MinIO/S3 with durable PostgreSQL intents; `MemoryStorageProvider` retained strictly as test double; cross-user upload intents rejected; deep parser validation (`pdf-lib` rejects encrypted/corrupt PDFs and enforces <=10 pages, `image-size` enforces 50px-8000px dimensions); declared size mismatch fails closed; production refuses mock auth and insecure secrets.
- **Tests:** Storage & upload test suite (`test/storage-and-upload.test.ts`, 16 passing).

### [RX-003] Extraction Job Lifecycle & Concurrency Control
- **Priority:** P0 | **Phase:** Phase 2 | **Dependencies:** RX-002 | **Status:** `DONE`
- **Description:** Asynchronous processing pipeline with atomic job claiming (`claimJobForProcessing`). Concurrency safe transitions; late responses cannot overwrite cancelled/failed states; zombie job recovery for interrupted workers; patient-scoped idempotency keys.
- **Acceptance Criteria:** Multiple concurrent workers cannot claim the same job; cancelled jobs cannot be updated to SUCCEEDED; bounded retries (max 3); jobs left in PROCESSING recovered safely.
- **Tests:** Extraction pipeline tests (`test/extraction-pipeline.test.ts`, 11 passing).

### [RX-004] AI Provider Abstraction & Isolated Mock Contract
- **Priority:** P0 | **Phase:** Phase 2 | **Dependencies:** RX-003 | **Status:** `DONE`
- **Description:** Implement `PrescriptionExtractionProvider` interface decoupling AI vendors. Implement deterministic `MockPrescriptionExtractionProvider` for local dev and CI. Gated live third-party AI integration until contractual BAA/DPA terms are satisfied.
- **Acceptance Criteria:** Mock contract tests explicitly separated from AI model accuracy evaluation; no real patient data transmitted to third parties; zero unmeasured model scores reported.
- **Tests:** Benchmark contract tests (`test/ai-benchmark.test.ts`, 5 passing).

### [RX-005] Indian Prescription Evaluation Benchmark Dataset & Provenance
- **Priority:** P0 | **Phase:** Phase 2 | **Dependencies:** RX-004 | **Status:** `DONE`
- **Description:** Maintain 32 synthetic/de-identified evaluation cases covering cursive handwriting, Indian brand names, abbreviations, and discharge summaries, with explicit machine-readable provenance metadata on each fixture.
- **Acceptance Criteria:** Document provenance, license, permitted use, and clinical archetypes recorded for all 32 fixtures. Mock matching never claimed as evidence of clinical AI accuracy.
- **Tests:** Benchmark provenance and safety rule validation tests (`test/ai-benchmark.test.ts`).

### [RX-006] Structured Extraction & Zod Domain Validation
- **Priority:** P0 | **Phase:** Phase 2 | **Dependencies:** RX-004 | **Status:** `DONE`
- **Description:** Strict validation of extraction payload: prescription date, clinic/doctor, medicines, dosage, frequency, duration, instructions. Preserves exact written medicine name separately from normalized interpretation.
- **Acceptance Criteria:** Raw extraction rejected if violating schema; uncertain or missing fields explicitly marked with warning badges; defaults never silently substituted.
- **Tests:** Schema validation test suite against valid and malformed AI outputs.

### [RX-007] Original Document Preview Component
- **Priority:** P1 | **Phase:** Phase 2 | **Dependencies:** RX-001 | **Status:** `DONE`
- **Description:** In-app document viewer supporting pinch-to-zoom, pan, expand/collapse view, and multi-page PDF navigation.
- **Acceptance Criteria:** Smooth document exploration alongside extracted fields; secure pre-signed preview URLs with 15-minute expiration; cross-user preview returns 404.
- **Tests:** UI render and interaction tests.

### [RX-008] Prescription Review & Correction UI
- **Priority:** P0 | **Phase:** Phase 2 | **Dependencies:** RX-006, RX-007 | **Status:** `DONE`
- **Description:** Interactive review screen displaying extracted values, flagging low-confidence/uncertain items, and allowing full patient editing.
- **Acceptance Criteria:** Patient can edit any field, add missing medicines, or discard invalid entries; historical prescriptions do NOT automatically activate regimens (`is_active_medication` defaults to false).
- **Tests:** Form interaction and editing tests.

### [RX-009] Confirmation & Durable Clinical Records Ingestion
- **Priority:** P0 | **Phase:** Phase 2 | **Dependencies:** RX-008 | **Status:** `DONE`
- **Description:** Endpoint `POST /api/v1/prescriptions/:jobId/confirm` executing inside an atomic database transaction. Persists `Document`, `UploadIntent`, `ExtractionJob`, `Prescription`, `PrescriptionItem`, and `TimelineEvent` models in PostgreSQL via Prisma.
- **Acceptance Criteria:** Unique constraint on `jobId` in `prescriptions` prevents duplicate confirmation across concurrent requests, server restarts, and instances; partial failure rolls back all clinical writes; original document remains immutable; survives service reinitialization.
- **Tests:** Database transaction and reinitialization tests (`test/extraction-pipeline.test.ts`).

### [RX-010] Error Recovery, Bounded Retries & Live Integration Suite
- **Priority:** P1 | **Phase:** Phase 2 | **Dependencies:** RX-003 | **Status:** `DONE`
- **Description:** Safe retry mechanics for failed AI extractions without creating duplicate document records. Live integration test suite (`test/integration.postgres-minio.test.ts`) covering durable clinical transactions (clean migrations, patient/document creation, job claim/complete, atomic prescription confirmation, forced failure rollback proof, concurrency race mutual exclusion, and MinIO direct upload).
- **Acceptance Criteria:** Idempotent retry operations; bounded max 3 retries before FAILED status; source document never lost; integration tests honestly reported as NOT RUN / SKIPPED when local container ports 5432/9000 are closed.
- **Tests:** Network timeout, provider error, and integration test suite (`test/integration.postgres-minio.test.ts`).

