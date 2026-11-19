# ADR-003: Relational Schema & Patient Profile Ownership

**Date:** 2026-10-12  
**Status:** proposed (Revised for Gate 1)  
**Deciders:** Principal Architect & Data Engineer  

---

## Context

My Health App manages highly sensitive, interconnected health data. The schema must fulfill these strict product requirements:
1. **Separation of Auth User and Patient Profile:** While the MVP serves one patient per account, the schema must cleanly isolate authentication identity (`users`) from clinical patient identity (`patient_profiles`) so future family member profiles can be added without table redesigns.
2. **Explicit Separation of Prescriptions and Active Medications:** Historical prescriptions must not automatically activate current medication schedules.
3. **Evidence-backed Provenance:** Timeline events must trace directly back to source documents or be marked patient-reported.
4. **Resilient Medication Lifecycle:** Differentiating the medicine entity, its prescribed instructions, its recurring reminder schedule, and individual dose outcome events.
5. **Strict Multi-Tenant Isolation & Anti-Spoofing:** Unambiguous ownership paths on all tables. Inviolable rule: **Never trust a client-provided `patient_profile_id` as proof of ownership.**
6. **Connection Pooling & RLS Interaction:** PostgreSQL Row-Level Security (RLS) rules must be designed to avoid connection pool state leaks (e.g., when using PgBouncer or Prisma connection pools).

---

## Decision

We adopt a fully normalized **PostgreSQL** relational schema managed through **Prisma ORM** migrations.

### Core Entity Relationship Design

```
users (auth identity)
  └── patient_profiles (clinical patient identity, 1:1 in MVP)
        ├── user_preferences (timezone, notification preferences)
        ├── documents (uploaded raw prescription images/PDFs)
        │     └── ai_extraction_jobs (processing state, provider, raw payload)
        ├── prescriptions (confirmed clinical prescription record)
        │     └── prescription_items (individual medicines on prescription)
        ├── medical_events (chronological timeline items)
        ├── medications (active/historical medication courses)
        │     └── medication_schedules (recurrence rules, times, start/end)
        │           └── dose_events (specific scheduled dose occurrences & outcomes)
        ├── report_exports (generated PDF metadata)
        └── audit_events (append-only security & data modification log)
```

### Table Definitions & Primary Keys

1. **`users`**: `id` (UUID, PK), `email` (unique), `password_hash` (nullable if OAuth), `auth_provider`, `is_verified` (BOOLEAN), `created_at`, `updated_at`.
2. **`patient_profiles`**: `id` (UUID, PK), `user_id` (UUID, FK -> users.id), `full_name`, `date_of_birth`, `gender`, `blood_group`, `is_primary_account_holder` (boolean), `created_at`, `updated_at`.
3. **`documents`**: `id` (UUID, PK), `patient_profile_id` (UUID, FK), `storage_key` (UUID string), `original_filename`, `mime_type`, `file_size_bytes`, `sha256_checksum`, `processing_status` (UPLOADED, EXTRACTING, READY_FOR_REVIEW, CONFIRMED, FAILED), `created_at`, `updated_at`.
4. **`ai_extraction_jobs`**: `id` (UUID, PK), `document_id` (UUID, FK), `provider_name`, `model_version`, `status` (PENDING, PROCESSING, SUCCESS, FAILED), `extracted_payload` (JSONB), `error_message`, `attempt_count`, `created_at`, `completed_at`.
5. **`prescriptions`**: `id` (UUID, PK), `patient_profile_id` (UUID, FK), `source_document_id` (UUID, FK), `prescription_date` (DATE, nullable), `date_is_uncertain` (BOOLEAN), `doctor_name`, `clinic_hospital_name`, `documented_diagnosis`, `review_status` (DRAFT, CONFIRMED, DISCARDED), `confirmed_at`, `created_at`.
6. **`prescription_items`**: `id` (UUID, PK), `prescription_id` (UUID, FK), `medicine_name_as_written`, `normalized_medicine_name`, `strength`, `dosage_form` (TABLET, SYRUP, INJECTION, etc.), `frequency_text`, `duration_text`, `route`, `instructions`, `created_at`.
7. **`medical_events`**: `id` (UUID, PK), `patient_profile_id` (UUID, FK), `event_type` (PRESCRIPTION, VISIT, LAB_RESULT, PATIENT_REPORTED), `event_date` (DATE, nullable), `date_precision` (EXACT, MONTH_YEAR, YEAR_ONLY, UNSPECIFIED), `title`, `description`, `source_document_id` (UUID, FK, nullable), `source_prescription_id` (UUID, FK, nullable), `created_at`.
8. **`medications`**: `id` (UUID, PK), `patient_profile_id` (UUID, FK), `source_prescription_item_id` (UUID, FK, nullable), `medicine_name`, `dosage`, `is_active` (BOOLEAN), `started_at` (DATE), `stopped_at` (DATE, nullable), `instructions`, `created_at`.
9. **`medication_schedules`**: `id` (UUID, PK), `medication_id` (UUID, FK), `patient_profile_id` (UUID, FK), `recurrence_type` (DAILY, MULTIPLE_DAILY, SPECIFIC_WEEKDAYS, EVERY_N_DAYS), `recurrence_config` (JSONB, e.g. weekdays array or interval integer), `dose_times` (TIME[]), `start_date` (DATE), `end_date` (DATE, nullable), `is_active` (BOOLEAN), `created_at`.
10. **`dose_events`**: `id` (UUID, PK), `medication_schedule_id` (UUID, FK), `patient_profile_id` (UUID, FK), `scheduled_for` (TIMESTAMP WITH TIME ZONE), `status` (PENDING, TAKEN, SKIPPED, SNOOZED, MISSED), `recorded_at` (TIMESTAMP WITH TIME ZONE, nullable), `skip_reason` (TEXT, nullable), `snooze_count` (INTEGER DEFAULT 0), `client_uuid` (UUID for offline deduplication, unique), `created_at`, `updated_at`.
11. **`report_exports`**: `id` (UUID, PK), `patient_profile_id` (UUID, FK), `storage_key`, `date_range_start`, `date_range_end`, `included_sections` (JSONB), `file_size_bytes`, `created_at`.
12. **`audit_events`**: `id` (UUID, PK), `patient_profile_id` (UUID, FK), `actor_user_id` (UUID, FK), `action` (VARCHAR), `resource_type`, `resource_id`, `ip_address`, `user_agent`, `created_at`.

---

## Multi-Tenant Authorization & Row-Level Security (RLS) Strategy

1. **Application Layer as Authoritative Barrier:**
   - Every API request derives the authorized `patient_profile_id` from the cryptographically verified JWT session context.
   - All repository/service layer queries enforce explicit `WHERE patient_profile_id = session.patientProfileId` filters.
   - Client-provided `patient_profile_id` values in request bodies or query params are strictly ignored.
2. **Database-Layer Defense-in-Depth (PostgreSQL RLS with Connection Pooling):**
   - In pooled database environments (e.g. PgBouncer or Prisma connection pool), session variables set via `SET app.current_patient_id` can leak across pooled connections if not isolated.
   - **Enforcement Pattern:** When RLS is active, queries execute within transaction blocks using local variables:
     ```sql
     SET LOCAL app.current_patient_id = '<authorized_patient_uuid>';
     ```
   - RLS policy:
     ```sql
     CREATE POLICY "patient_profile_isolation" ON prescriptions
       FOR ALL USING (patient_profile_id = NULLIF(current_setting('app.current_patient_id', true), '')::uuid);
     ```

---

## Consequences

### Positive
- Strict foreign key constraints and cascade rules ensure zero orphaned records.
- All records map unambiguously to `patient_profile_id`, making tenant isolation checks straightforward and performant.
- Future family profile support is baked into the relational model without requiring schema migrations.
- Historical prescriptions cannot accidentally activate current medications.

### Negative
- Multi-table schema requires coordinated transactions for prescription confirmation and schedule creation.
- Offline synchronization must map client-generated UUIDs idempotently.
