# Release Checklist — My Health App MVP

**Document Version:** 1.0.0  
**Target Release:** MVP v1.0.0  

All items in this checklist must be satisfied before Gate 5 (Production Release Approval) can be granted.

---

## 1. Authentication & Session Management
- [ ] Email/password registration, login, and password reset verified with real SMTP/Auth provider.
- [ ] Google OAuth sign-in verified on both Android and iOS devices.
- [ ] Session tokens expire as configured; silent refresh works without user disruption.
- [ ] Logout invalidates both client-side secure storage and server-side refresh tokens.
- [ ] No placeholder or non-functional auth buttons exist.

## 2. Prescription Pipeline & AI Processing
- [ ] Camera capture, image picker, and PDF picker function across multiple Android/iOS versions.
- [ ] Documents upload exclusively to private object storage buckets with randomized UUID keys.
- [ ] Extraction responses strictly adhere to Zod domain schemas; malformed outputs trigger safe retries.
- [ ] Low-confidence and ambiguous fields are visibly flagged with warning badges for patient review.
- [ ] Mandatory patient review and explicit confirmation step blocks draft records from timeline activation.
- [ ] Original prescription image is preserved and always viewable alongside extracted records.

## 3. Medical Timeline Integrity
- [ ] Records display in correct chronological order based on medical event date (not upload date).
- [ ] Approximate and unknown dates are rendered with clear "Unspecified" indicators, never fabricated.
- [ ] Tapping any timeline event links directly to the verified prescription and source document.
- [ ] Historical prescriptions are cleanly separated from active medications.

## 4. Medication Scheduling & Offline Reminders
- [ ] Recurrence engine passes 100% of unit tests (Daily, Multiple daily, Weekdays, Every N days).
- [ ] Local device notifications trigger on time while device is in Airplane Mode / disconnected from network.
- [ ] Dose action buttons (Taken, Skipped, Snooze) function from lock screen notifications and in-app dashboard.
- [ ] Snoozing never counts as a taken dose; unrecorded doses become "Missed / Unconfirmed".
- [ ] Offline dose records queue locally and synchronize idempotently upon network restoration.
- [ ] Adherence percentage matches the approved clinical formula.

## 5. Doctor-Ready PDF Reports & Native Sharing
- [ ] Patient can selectively filter records (date range, specific conditions/medications) to include.
- [ ] Generated PDF displays clean, professional styling with zero overlapping text or data truncation.
- [ ] Clear disclaimer appears on every page noting that the report reflects patient-uploaded records.
- [ ] Native OS share sheet opens cleanly allowing AirDrop, Email, WhatsApp, or Print.
- [ ] Generated PDF is stored securely or cleaned up after sharing; no public web links.

## 6. Security, Privacy & Compliance
- [ ] Cross-tenant access tests (IDOR verification) pass 100% across all API endpoints.
- [ ] Client application bundles contain zero API keys, database secrets, or master tokens.
- [ ] All server logs are sanitized: zero patient names, phone numbers, or health details in stdout.
- [ ] Pre-signed storage URLs have maximum 15-minute TTLs.
- [ ] Full account and data deletion routine purges all database records and storage files permanently.
- [ ] Privacy policy and patient consent disclosures meet Digital Personal Data Protection (DPDP) and HIPAA principles.

## 7. Build Quality & Performance
- [ ] Clean Android Release AAB / APK generated with Proguard / R8 minification.
- [ ] Clean iOS build archive validated with required entitlements and privacy manifests.
- [ ] Automated test suite (Unit, Integration, E2E) achieves green status across client and backend.
- [ ] `npm audit` reports zero critical or high vulnerabilities.
- [ ] Accessibility audit verifies WCAG 2.2 Level AA compliance (contrast, dynamic font scaling).
