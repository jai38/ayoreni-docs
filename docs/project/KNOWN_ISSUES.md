# Known Issues & Environmental Considerations — My Health App

**Document Version:** 1.2.0  
**Last Updated:** 2026-10-12  

---

## 1. Active Gating Status

- **GATE-PHASE-3 (Phase 3 Medical Timeline Approval):** Phase 2 Prescription Pipeline remediation is complete with 75/75 automated unit and regression tests passing across mobile and backend. Execution is strictly halted at the Phase 2 completion review gate pending explicit user authorization to begin Phase 3 (Medical Timeline).

---

## 2. Environment Caveats & Platform Observations

- **ENV-001 (Windows 11 File Encoding):** Default PowerShell file creation in Windows can output UTF-16LE without explicit UTF-8 flags. All file writing tools and node scripts enforce explicit UTF-8 encoding.
- **ENV-002 (Local Datastores & Integration Suite Status):** Docker Compose configuration is defined in `myhealthapp-backend/docker-compose.yml` (PostgreSQL 16 and MinIO). In the active Windows 11 host environment, the Docker Desktop daemon was not running (`open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified`), and ports 5432 and 9000 are closed.
  - In accordance with our honest reporting policy, `test/integration.postgres-minio.test.ts` skips container-dependent tests via `describe.skipIf` and reports integration coverage as **NOT RUN (6 skipped)** rather than counting closed-port checks as passing tests.
  - To execute live container integration tests: start Docker Desktop and run `docker compose up -d` in `myhealthapp-backend/`, followed by `npx prisma migrate deploy` and `npm test`.
  - All 75 unit and regression test suites across mobile (20 tests) and backend (55 tests) run with 100% deterministic reproducibility using isolated test doubles.
- **ENV-003 (Platform Notification Caps):** Handled via `DiscreteOccurrenceScheduler` in `myhealthapp/services/notifications/actual-count-scheduler.ts` (enforcing iOS 60-slot ceiling and Android 100-slot ceiling).

---

## 3. Medical, Privacy & AI Operational Edge Cases

- **AI-001 (Production AI Provider Selection Deliberately Open):** In accordance with ADR-005, production AI provider selection remains open until commercial contractual privacy terms (BAA/DPA), training exclusion verification, and data residency under DPDP Act are evaluated against the 32-case Indian prescription benchmark. Zero real patient data was sent to third-party AI APIs during Phase 2.
- **AI-002 (Benchmark Provenance & Empirical Evaluation Separation):** All 32 benchmark fixtures carry explicit machine-readable provenance metadata. Test-double contract tests are strictly decoupled from empirical accuracy metrics. No unmeasured model scores are claimed.
- **STORAGE-001 (Fail-Closed Private Storage & Parser Bounds):**
  - Storage keys are validated strictly as `documents/<patientProfileId>/<uuid>`. Non-UUID final segments, path traversal, and format mismatches are rejected.
  - Upload intents enforce a strict 15-minute TTL with explicit database states (`PENDING`, `VERIFYING`, `VERIFIED`, `REJECTED`) and lock leasing. Transient storage errors allow safe client retry; permanent validation errors transition to `REJECTED`.
  - Absent or zero-byte uploads fail closed with 400 Bad Request; fabricated PDF fallbacks are strictly prohibited.
  - Parser resource limits: parsing runs in a bounded worker thread (`parser-worker.cjs`) with `MAX_CONCURRENT_PARSERS = 4` and is terminated forcefully via `worker.terminate()` upon 5000ms timeout. Enforces maximum 10 pages for PDFs (via `pdf-lib`) and 8000x8000 pixels for images (via `image-size`), with a 15MB file size ceiling.
- **REG-001 (Prescription Handwriting Variance & Medical Safety):** Severely degraded handwriting or torn physical prescriptions cannot be 100% reliably extracted by any vision model without human review. The UI mandates explicit patient review and confirmation before committing any clinical record to the medical timeline.
- **REG-002 (Historical Prescription Inactivation Rule):** Historical prescriptions never automatically activate recurring medication reminder alarms. Users must explicitly check the activation box during review if they are currently taking the medication.


