# ADR-006: Medication Scheduling, Notification Limits & Offline Resilience

**Date:** 2026-10-12  
**Status:** proposed (Revised for Phase 1)  
**Deciders:** Principal Architect & Mobile Engineering Lead  

---

## Context

Medication reminders must help patients take medicines on time while operating within native operating system constraints on iOS and Android:
1. **OS Notification Limits:**
   - **iOS:** Strict hard ceiling of **64 local notifications** scheduled simultaneously across the entire application. Exceeding 64 causes iOS to silently drop or truncate notifications.
   - **Android:** While Android does not have the exact 64-notification ceiling, system limits and OEM vendor power-saving frameworks constrain alarm scheduling.
2. **Deficiencies of Static Approximations:**
   - Approximating notification count by dividing by "daily doses" fails on complex schedules (e.g. alternating days, cyclical every 3 weeks, varying multi-dose times, finite courses ending mid-week).
   - The scheduler must count **actual upcoming discrete notification timestamps** and fill the queue safely up to the platform ceiling.
3. **Realistic OS Delivery Expectations:**
   - OS notification delivery must be treated as **best-effort**. Exact alarms (`SCHEDULE_EXACT_ALARM`) and battery optimization exemptions must **not** be treated as unconditional requirements; the app must function gracefully if permissions are denied or exact alarms are disabled.
4. **Decoupling Clinical Truth from Delivery Alerts:**
   - Notification delivery, display, dismissal, or snoozing must **never** automatically mark a dose as taken.
5. **Robust Replenishment Lifecycle:**
   - Safe replenishment triggers on schedule changes, app launches, timezone transitions, and device restarts.

---

## Decision

### 1. Actual-Count Notification Scheduler (`DiscreteOccurrenceScheduler`)
Instead of a daily-dose approximation or a static 14-day assumption, the mobile client implements an **Actual-Count Scheduling Engine**:

1. **Platform Capacity Thresholds:**
   - **iOS Cap:** 60 scheduled notifications (leaving a 4-slot buffer for snoozes and immediate system alerts out of the 64 maximum).
   - **Android Cap:** 100 scheduled notifications.
2. **Discrete Projection Algorithm:**
   - Collect all active medication schedules for the patient.
   - Use the pure `RecurrenceCalculator` to project discrete, chronological upcoming dose occurrence timestamps $[t_1, t_2, t_3, \dots, t_N]$ starting from `now`.
   - Take the top $K$ upcoming occurrences where $K = \min(N, \text{PlatformCap})$.
   - This guarantees that whether a patient has 1 medication once weekly or 6 medications 4 times daily, the scheduler always schedules the exact next $K$ occurrences without ever exceeding platform limits.
3. **Safe Replenishment Triggers:**
   The notification queue is audited and replenished upon:
   - Schedule modification (creation, edit, pause, cancellation).
   - App foreground transition (`AppState === 'active'`).
   - Device reboot (`BOOT_COMPLETED` on Android).
   - Timezone change or system clock alteration event.
   - Recording an action on a scheduled dose (cancelling the active alert and scheduling the next available occurrence).

### 2. Best-Effort Delivery & Graceful Degradation
- The app explicitly treats OS notification delivery as **best-effort**, not an infallible medical alarm.
- **Permission Flow:**
  - Standard user-friendly notification permission request.
  - If permissions are granted, standard local notifications are scheduled.
  - If exact alarms (`SCHEDULE_EXACT_ALARM`) or battery optimization exemptions are unavailable or declined on Android, the app gracefully falls back to inexact windowed alarms (`setWindow` / standard notification scheduling) and clearly informs the user that reminders may arrive within a 5-15 minute delivery window.
  - Under no circumstances is exact alarm capability treated as an unconditional blocker for using the app.

### 3. Decoupling Notification State from Clinical Adherence
- **Inviolable Invariant:** Notification delivery, display, dismissal, or snoozing **never automatically marks a dose as taken**.
- Only an affirmative, explicit user action (tapping "Take" in the notification action or tapping "Mark Taken" in the app) transitions a dose event to `TAKEN`.
- Dismissed notifications remain `PENDING` until their confirmation cutoff window expires (e.g. 4 hours past scheduled time), at which point they become `MISSED / UNCONFIRMED`.
- Snooze reschedules the notification by 15-30 minutes and increments `snooze_count` without marking the dose taken.

### 4. Independent Persistence & Offline Conflict Resolution
- Clinical schedules and dose events are stored in the relational database (PostgreSQL backend, local SQLite mirror on mobile).
- Offline dose events use client-generated UUIDs (`client_uuid`) and sync idempotently to `/api/medications/doses/sync`.
- Conflict resolution ensures affirmative client actions (`TAKEN`) take precedence over automated server expirations (`MISSED`).

---

## Consequences

### Positive
- Completely prevents iOS 64-notification overflow bugs under any combination of complex schedules.
- Gracefully functions across all Android devices without requiring invasive or restricted system permissions.
- Absolute integrity of adherence data: missed reminders or dismissed banners cannot corrupt clinical records.

### Negative
- High-frequency regimens on iOS (e.g. 15 doses/day) cover fewer days in advance (~4 days) and rely on the patient opening the app or background replenishment tasks.
