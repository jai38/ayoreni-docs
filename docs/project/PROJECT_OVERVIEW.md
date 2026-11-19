# Project Overview — My Health App

**Document Version:** 1.0.0  
**Last Updated:** 2026-10-12  
**Author:** Principal Software Architect & Engineering Lead  
**Status:** Living Master Document  

---

## 1. Product Vision & Mission

**My Health App** is a patient-first, security-hardened personal health management platform designed for mobile (iOS & Android). Its primary purpose is to provide patients with complete ownership, organization, and traceability over their medical history and ongoing medication regimens.

### Core Capabilities

1. **Personal Medical History & Timeline:**
   - Secure upload of historical and current medical documents (prescriptions, discharge summaries, diagnostic notes).
   - Server-side AI document extraction (using vision and multimodal language models) with structured schema validation.
   - **Mandatory patient review and explicit confirmation**: AI extractions are treated as draft suggestions until validated by the patient.
   - Chronological medical timeline organized by actual clinical event dates rather than upload dates.
2. **Medication Management & Adherence:**
   - Explicit confirmation of active medication regimens derived from prescriptions or patient entry.
   - Local, resilient medication reminders that operate offline without relying on network availability.
   - Traceable dose tracking (Taken, Skipped, Snoozed, Missed, Unresolved) with transparent adherence metrics.
3. **Doctor-Ready Health Reports:**
   - Selective export of medical history, prescriptions, and adherence data into a clean, professional, tamper-evident PDF report.
   - Native OS share sheet integration allowing patients to safely share reports with clinicians during consultations.

---

## 2. Inviolable Product Principles

- **Patient Ownership:** All patient data belongs strictly to the patient. No cross-patient data leakage, third-party tracking, or model training on patient records.
- **Evidence-Backed Records:** Every verified fact in the medical timeline must trace back to a source document or be explicitly flagged as patient-reported.
- **Separation of Prescriptions and Active Medications:** Historical prescriptions do *not* automatically activate recurring medication reminders. A doctor prescribing a course in 2023 does not mean the patient is taking it today.
- **No AI Hallucinations as Medical Truth:** The application must never present AI-inferred diagnoses, suggested cures, or speculative treatment plans as clinical facts. Missing data is acknowledged, never fabricated.
- **Offline Reminder Resilience:** Reminders must fire reliably on-device using local notification schedulers even when disconnected from the internet.
- **Privacy and Security by Design:** Zero privileged credentials in the mobile bundle; encrypted data at rest and in transit; private document storage with time-limited signed URLs; strict server-side authorization.

---

## 3. High-Level System Architecture

The application is structured as a client-server architecture with strict separation of concerns:

```
[ React Native / Expo Mobile App ]
     │ (HTTPS / TLS 1.3 + OIDC / JWT Auth)
     ▼
[ Modular Monolith Backend API (Node.js / TypeScript) ]
     ├── Auth & Profile Module
     ├── Document & Storage Service (Private S3 / Supabase Storage)
     ├── AI Extraction Pipeline (Claude 3.5 Sonnet / Vision Provider Adapter)
     ├── Medical Timeline & Prescription Engine
     ├── Medication & Recurrence Schedule Engine
     ├── Dose Tracking & Adherence Analytics
     ├── Doctor-Ready PDF Report Generator
     └── Audit Logging & Privacy/Deletion Engine
     │
     ▼
[ PostgreSQL Database (Relational Schema + Row-Level Security) ]
```

---

## 4. Key Architectural Boundaries

1. **Mobile Client (`myhealthapp/`):**
   - Built with React Native, TypeScript, Expo (Managed Workflow), and Expo Router.
   - Encapsulates UI, local notification scheduling, local cache (SQLite / MMKV), and secure token storage (`expo-secure-store`).
   - Does **not** store raw database credentials or AI provider API keys.
2. **Backend Service (`myhealthapp-backend/`):**
   - Modular monolith built with Node.js, Express/Fastify, and TypeScript.
   - Connects to PostgreSQL using Prisma ORM with version-controlled migrations.
   - Manages signed uploads to private object storage, calls external AI extraction providers via an abstracted provider interface (`PrescriptionExtractionProvider`), and enforces ownership checks on all endpoints.
3. **Storage & Documents:**
   - Raw prescriptions and PDF exports are stored exclusively in private cloud object storage buckets.
   - Files are accessed only via short-lived pre-signed URLs (e.g., 15-minute expiration) generated after authentication and authorization verification.

---

## 5. Phased Delivery Roadmap

- **Phase 0:** Discovery & Planning (Gate 1 — Current Step)
- **Phase 1:** Foundation (Auth, App Shell, Navigation, DB Schema & API Client)
- **Phase 2:** Prescription Pipeline (Upload, AI Extraction, Review & Confirmation)
- **Phase 3:** Medical Timeline (Chronological timeline, filters, source links)
- **Phase 4:** Medication Management (Recurrence engine, local notifications, dose tracking)
- **Phase 5:** PDF Reports & Privacy (Doctor export, deletion workflows, security audit)
- **Phase 6:** Release Preparation (Regression tests, Android/iOS builds, compliance checks)
