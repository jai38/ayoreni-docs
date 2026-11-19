# Open Questions & Key Architectural Decisions (Revised for Gate 1)

**Document Version:** 1.1.0  
**Last Updated:** 2026-10-12  
**Status:** Under Gate 1 Review  

---

## 1. Decisions Deliberately Deferred (With Recommended Defaults)

### Q1: Production Hosting Provider & Region Selection
- **Status:** **Deferred to Phase 5 / 6 Infrastructure Milestone.**
- **Evaluation Criteria:**
  1. Indian regional data residency (e.g. AWS `ap-south-1` Mumbai or GCP `asia-south1`) to satisfy the Digital Personal Data Protection (DPDP) Act.
  2. Backup and point-in-time recovery capabilities.
  3. Secret management integration (audited secret vaults).
  4. Operational overhead and monthly running cost.
- **Recommended Default:** Containerized deployment on **AWS ECS Fargate + RDS PostgreSQL + S3 (in Mumbai `ap-south-1`)** or managed Docker on **Render** (subject to regional residency verification).
- **Local Development Policy:** 100% self-contained local Docker Compose environment (Node.js, PostgreSQL, MinIO), ensuring development proceeds immediately without blocking on cloud vendor selection.

---

### Q2: Production AI Vision Provider Selection
- **Status:** **Deferred to AI Extraction Evaluation Benchmark Milestone (Phase 2).**
- **Evaluation Criteria:**
  1. Extraction accuracy on our custom Indian prescription benchmark (covering regional brand names, handwriting, medical shorthand, stamps, multipage discharge summaries).
  2. Contractually verified zero-training / data retention addendums on our active commercial account.
  3. Latency and cost per document.
- **Candidates:**
  - Anthropic Claude 3.5 Sonnet (strong clinical reasoning and layout parsing).
  - Google Gemini 1.5 Pro / Flash (large context window, fast vision processing, regional presence).
  - AWS Bedrock Claude 3.5 Sonnet (AWS enterprise security boundary and Mumbai region availability).
- **Recommended Default:** Maintain the provider-independent `PrescriptionExtractionProvider` interface and use the deterministic `MockPrescriptionExtractionProvider` for all local development and CI testing.

---

### Q3: Authentication Engine Selection
- **Status:** **Configurable behind `AuthProviderAdapter`.**
- **Evaluation Criteria:**
  1. Google Sign-In cryptographic token verification on mobile.
  2. Native email verification and password reset workflows.
  3. Session revocation capability (refresh token rotation and global logout).
  4. Data sovereignty (keeping user credentials in PostgreSQL).
- **Candidates:**
  - Embedded Node.js Argon2id + JWT + OIDC verifier (100% self-hosted in PostgreSQL).
  - Supabase Auth (Postgres-integrated, managed token lifecycle).
- **Recommended Default:** Start Phase 1 with the embedded Node.js Argon2id + JWT engine to maintain zero third-party dependencies during core development, leaving the adapter open to switch to managed Supabase Auth upon production deployment.

---

### Q4: Server-Side PDF Report Generation Engine
- **Status:** **Under Review for Phase 5.**
- **Options:**
  - Option A: **PDFKit** (Lightweight, pure Node.js, low memory footprint, deterministic layout).
  - Option B: **React-PDF / @react-pdf/renderer** (Declarative JSX-based layout matching mobile design tokens).
  - Option C: **Puppeteer / Headless Chromium** (HTML/CSS to PDF, highest visual polish, but requires heavier container resources).
- **Recommended Default:** **React-PDF / PDFKit** to keep container memory overhead small (<512MB) and startup times fast.
