# ADR-002: Backend Architecture & Hosting Strategy

**Date:** 2026-10-12  
**Status:** proposed (Revised for Gate 1)  
**Deciders:** Principal Architect & Backend Engineering Lead  

---

## Context

The backend service for My Health App must handle:
1. Patient profile and authentication management.
2. Signed upload and private storage of medical documents.
3. Asynchronous AI document extraction orchestration.
4. Relational medical event timeline, prescription tracking, and medication schedules.
5. Dose tracking and adherence calculation.
6. Doctor-ready PDF report generation.
7. Audit trails, privacy controls, and data erasure workflows.

We must select a coherent architecture and local development environment while establishing formal evaluation criteria for production hosting.

---

## Decision

### 1. Backend Architecture: TypeScript Modular Monolith
We adopt a **Modular Monolith** built with **Node.js, Express/Fastify, and TypeScript**, backed by **PostgreSQL** (via **Prisma ORM**) and **S3-compatible Private Object Storage**.

The codebase resides in `myhealthapp-backend/` organized into bounded domain modules:
- `modules/auth/`: Identity verification, session handling, token issuance, password reset.
- `modules/patients/`: Patient profiles and settings.
- `modules/documents/`: Private file storage, pre-signed upload/download URLs, checksum verification.
- `modules/ai-extraction/`: Provider abstraction (`PrescriptionExtractionProvider`), job queue, Zod validation.
- `modules/prescriptions/`: Prescription review, correction, confirmation, and item models.
- `modules/timeline/`: Chronological medical event query engine.
- `modules/medications/`: Regimens, recurrence engine, and dose event synchronization.
- `modules/reports/`: Doctor PDF report generation and native sharing.
- `modules/audit/`: Append-only compliance audit trail.

### 2. Local Development Environment: Docker Compose
For local development, testing, and CI, the environment is 100% self-contained and reproducible via Docker Compose:
- **`app`**: Node.js modular service running with hot reload.
- **`postgres`**: Local PostgreSQL 16 container with persistent volumes.
- **`minio`**: Local S3-compatible private object storage container with automated bucket initialization.
- **Zero Cloud Dependencies Required for Local Dev:** Developers can develop, test, and run full E2E flows offline without cloud accounts or API keys.

### 3. Production Hosting: Deferred Decision with Explicit Evaluation Matrix
We deliberately **defer the final production hosting selection** until the infrastructure evaluation phase. The production hosting candidate matrix is evaluated against the following criteria:

| Evaluation Criterion | Evaluation Requirement |
| :--- | :--- |
| **Data Residency & Compliance** | Data must reside in Indian data centers (e.g., AWS `ap-south-1` Mumbai, GCP `asia-south1`, or Azure Central India) to comply with Indian healthcare and DPDP provisions. |
| **Cost & Predictability** | Baseline monthly running cost for compute, database, storage, egress, and backups. |
| **Backup & Point-in-Time Recovery** | Automated daily snapshots, transaction log archiving, and tested restore drills. |
| **Secrets Management** | Native, audited secret storage (e.g. AWS Secrets Manager, Doppler, or HashiCorp Vault) preventing secrets in environment text files. |
| **Operational Simplicity** | Containerized deployment (Docker) with automated health checks, zero-downtime rolling deploys, and minimal maintenance overhead. |

**Candidate Options for Production:**
- **Option 1: AWS ECS Fargate + RDS PostgreSQL + S3 (in Mumbai `ap-south-1`):** Enterprise standard, robust compliance, native IAM, automated RDS snapshots.
- **Option 2: Managed PaaS (e.g., Render / Fly.io / Railway) + Cloudflare R2:** Rapid deployment, lower initial operational burden, but must verify Indian regional data residency guarantees.
- **Option 3: DigitalOcean / Hetzner Dedicated VPC + Managed Postgres:** Highly cost-effective, but requires self-managed secrets and compliance configuration.

---

## Consequences

### Positive
- The application remains completely containerized and cloud-agnostic; no vendor lock-in.
- Developers have a fast, free, fully working local environment via Docker Compose.
- Final production hosting decision is driven by verified regional compliance, cost, and operational facts rather than premature assumptions.

### Negative
- Production infrastructure deployment scripts (Terraform / CloudFormation / manifests) are deferred until the hosting provider decision gate.
