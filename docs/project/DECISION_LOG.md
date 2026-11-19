# Architecture Decision Log — My Health App

**Document Version:** 1.1.0  
**Last Updated:** 2026-10-12  

This log maintains an immutable chronological record of all architectural and product decisions. Each major decision is backed by an Architecture Decision Record (ADR) under [`docs/architecture/adr/`](file:///c:/Users/malan/Documents/GitHub/healthapp/docs/architecture/adr/).

---

## Decision Index

| ADR ID | Decision Title | Status | Date | Primary Driver |
| :--- | :--- | :---: | :---: | :--- |
| [ADR-001](file:///c:/Users/malan/Documents/GitHub/healthapp/docs/architecture/adr/ADR-001-mobile-framework.md) | Mobile Framework: React Native with Expo & Expo Router | Proposed (Gate 1) | 2026-10-12 | Cross-platform parity, native notification scheduling, TypeScript strictness |
| [ADR-002](file:///c:/Users/malan/Documents/GitHub/healthapp/docs/architecture/adr/ADR-002-backend-hosting.md) | Backend Architecture & Production Hosting Strategy | Proposed (Revised) | 2026-10-12 | Modular monolith, Docker Compose local dev, deferred production host with regional DPDP compliance matrix |
| [ADR-003](file:///c:/Users/malan/Documents/GitHub/healthapp/docs/architecture/adr/ADR-003-database-and-ownership.md) | Relational Database Schema, Anti-Spoofing Ownership & RLS Pooling Strategy | Proposed (Revised) | 2026-10-12 | Unambiguous ownership, client-ID anti-spoofing, connection pool safe RLS, family profile readiness |
| [ADR-004](file:///c:/Users/malan/Documents/GitHub/healthapp/docs/architecture/adr/ADR-004-authentication.md) | Authentication & Identity Management | Proposed (Revised) | 2026-10-12 | Configurable provider adapter, complete auth lifecycle, server-side identity derivation, automated IDOR tests |
| [ADR-005](file:///c:/Users/malan/Documents/GitHub/healthapp/docs/architecture/adr/ADR-005-ai-provider-and-extraction.md) | AI Prescription Extraction Architecture | Proposed (Revised) | 2026-10-12 | Provider-agnostic interface, Indian prescription benchmark, zero hardcoded models, mock provider for dev/CI |
| [ADR-006](file:///c:/Users/malan/Documents/GitHub/healthapp/docs/architecture/adr/ADR-006-medication-scheduling.md) | Resilient Medication Scheduling, Platform-Aware Limits & Offline Reconciliation | Proposed (Revised) | 2026-10-12 | Platform-aware dynamic horizon (iOS 64 limit), independent DB truth, notification dismissal invariant, offline sync |
| [ADR-007](file:///c:/Users/malan/Documents/GitHub/healthapp/docs/architecture/adr/ADR-007-document-storage-and-privacy.md) | Document Storage, Signed URLs, Privacy Lifecycle & Backup Retention Limits | Proposed (Revised) | 2026-10-12 | Private object storage, 15-min signed URLs, backup deletion limitations, audit logging, third-party AI data minimization |

---

## Log Entries

### Entry 2026-10-12: Phase 0 Discovery Formulation
- Proposed ADR-001 through ADR-007 as part of Gate 1 architectural review.
- Established strict policy: No code implementation until user provides explicit sign-off on Gate 1.

### Entry 2026-10-12: Gate 1 Conditional Review Revisions
- Revised ADR-002: Kept local Docker Compose; established evaluation matrix for production hosting (deferring choice to evaluate Indian regional residency e.g. AWS Mumbai, backup/restore, cost).
- Revised ADR-003: Added anti-spoofing invariant (never trust client-supplied patient ID) and transaction-scoped RLS pattern for connection pooling.
- Revised ADR-004: Created configurable `AuthProviderAdapter`, specified complete auth lifecycle (email verification, password reset, Google sign-in), and server-side authorization enforcement.
- Revised ADR-005: Removed hardcoded Claude 3.5 Sonnet dependency; established candidate evaluation criteria; prohibited claiming zero-retention without verified account contract; specified 30+ document Indian prescription benchmark; added `MockPrescriptionExtractionProvider`.
- Revised ADR-006: Replaced static 14-day assumption with dynamic platform-aware scheduling (iOS 64 limit, Android exact alarms/battery optimization); decoupled DB truth from OS notification queue; established invariant that notification delivery/dismissal/snooze never marks a dose taken; defined offline sync conflict resolution.
- Revised ADR-007: Added explicit backup deletion limitation disclosure (30-day rolling aging out + restore tombstones) and third-party AI data minimization rules.

### Entry 2026-10-12: Phase 3 Medical Timeline Architecture & Implementation
- Dedicated Timeline Module: Created authenticated `/api/v1/timeline` backend routes and service decoupled from raw ingestion drafts.
- Clinical Data Integrity: Guaranteed unreviewed AI drafts never enter verified timeline. Only explicit patient confirmations create timeline events.
- Date Uncertainty & Timezone Safety: Stored `dateIsUncertain` and `datePrecision` (`EXACT`, `MONTH_ONLY`, `YEAR_ONLY`). Timezone-safe string parsing prevents day shift.
- Secure Source Access: Short-lived pre-signed download URLs generated via `/api/v1/timeline/:id/document-url` with IDOR prevention and strict ownership checks.
- Mobile Experience: Built chronological timeline feed grouped by month/year with search, category/year filtering, pagination, empty states, and full detail modal (`TimelineDetailModal.tsx`).
