# Production Readiness Assessment — My Health App

**Assessment date:** 2026-10-19
**Method:** Full read of both codebases, configs, CI/CD, live service probes, and test suites (all claims below verified in code unless marked otherwise).
**Supersedes:** the phase/gate status in `PROJECT_STATUS.md`, `IMPLEMENTATION_PLAN.md`, `BACKLOG.md`, `KNOWN_ISSUES.md` (all stale).

---

## 1. Where the product actually stands

The app is **feature-complete for the MVP scope** and is already running in a production-shaped environment:

| Layer | State | Evidence |
|---|---|---|
| Mobile typecheck | ✅ clean | `tsc --noEmit`, 0 errors |
| Mobile tests | ✅ 60/60 | `npm test` |
| Backend typecheck | ✅ clean | `tsc --noEmit`, 0 errors |
| Backend tests | ✅ 113 pass, 6 skipped | skip = Docker-only integration suite |
| Backend live | ✅ production | `GET /healthz` → `{"status":"ok","environment":"production"}` |
| Web app | ✅ live | `jai38.github.io/myhealthapp` |
| Landing | ✅ live | `jai38.github.io/MyHealth` |
| Android APK | ✅ available | GitHub pre-release `v0.1.0-preview` (2026-10-18) |
| CI/CD | ✅ green | Both repos: test → build → deploy hook → healthcheck |

**Feature inventory (built and wired end-to-end):** email/password auth, prescription upload (camera/gallery/PDF) with private pre-signed storage, Gemini AI extraction with Zod validation, mandatory review/confirm, verified timeline with search/filters, medications with recurrence and local offline notifications, dose tracking (taken/skip/snooze) with adherence, manual medication entry, PDF report generation + native share, account modal, legal consent gate, GitHub Pages web build.

**The remaining work is not feature work. It is launch hardening, compliance, and operational reliability.**

---

## 2. Launch tiers

Two realistic paths. Recommend **Tier A (closed beta) this week**, **Tier B (public PROD) after the P0 list**.

| Tier | Audience | Requirements |
|---|---|---|
| **A — Closed beta** | Known testers, synthetic/own data | Current build + G3, G5 fixed; explicit "beta, not medical advice" notice; testers informed data is on free-tier infra. |
| **B — Public production** | Real patients | All P0 items closed, P1 infra items addressed, compliance sign-off. |

---

## 3. P0 — blockers for public production

### G1. Account & data deletion does not exist
- **Evidence:** no `DELETE` route anywhere in `myhealthapp-backend/src` (`grep router.delete` → none); account UI has no delete action.
- **Why it blocks:** DPDP right-to-erasure; `RELEASE_CHECKLIST.md` §6; also a store-review (App Store / Play) expectation for health apps.
- **Fix:** `DELETE /api/v1/account` — authenticate, then in a transaction purge `User` (cascade removes profile/documents/intents/jobs/prescriptions/items/events/medications/doses), then delete all objects under `documents/<patientProfileId>/` from S3. Add tombstone/audit event (`ACCOUNT_PURGED`). Return 204. Add regression test proving storage objects are gone.
- **Estimate:** 1–1.5 days incl. tests.

### G2. Password reset, email verification, Google sign-in are stubs
- **Evidence:** `auth-adapter.ts:296-302` (`requestPasswordReset`/`resetPassword` → `// Stub`), `signInWithGoogle` throws `:269-271`; only `/auth/signup`, `/auth/login`, `/auth/me` routes exist; no UI for forgot-password.
- **Why it blocks:** account recovery is mandatory for real users; `RELEASE_CHECKLIST.md` §1.
- **Fix (minimum):** password reset via signed, single-use, 30-min token stored hashed in DB + transactional email provider (Resend/SES/Postmark). Email verification can be deferred if signup policy is explicit, but remove the dead `isVerified: true` shortcut or implement it. Google OAuth can ship post-launch (`OPEN_QUESTIONS.md` Q3) — but then remove/disable any implied support.
- **Estimate:** 2–3 days (reset + email provider). Google: +2 days.

### G3. Auth tokens stored insecurely on device
- **Evidence:** `services/auth/auth-service.ts` uses `AsyncStorage` for `accessToken`/session; `expo-secure-store` is not in `package.json`; docs claim SecureStore.
- **Why it blocks:** on compromised/rooted devices tokens are readable; health app security posture.
- **Fix:** add `expo-secure-store`; store token/session there on native, keep `localStorage` only for web; migrate/clear old AsyncStorage keys on first run. Add a test for the storage adapter.
- **Estimate:** 0.5–1 day.

### G4. Abuse protection & CORS too open
- **Evidence:** `app.ts:77-83` — CORS `origin: true` when `CLIENT_URL=*`; Render sets `CLIENT_URL=*`. No rate-limit middleware in `package.json` or `app.ts`. No helmet rate limiting. Auth endpoints unprotected.
- **Fix:** `express-rate-limit` (global + strict login/signup/reset buckets), set production `CLIENT_URL` to the actual origins (Pages URL + future app scheme), slow down bcrypt on repeated failures, consider account lockout/backoff. Also add request body size limits already present (1 MB — keep).
- **Estimate:** 0.5 day.

### G5. Dose history is synthesized as "TAKEN"
- **Evidence:** `services/medications/medication-store.ts:132-176` — `generateTrailingWeekDoses()` sets **every past occurrence to `status: 'TAKEN'` with an actionTime**. It is used for guest demo (`init`, line 227) **and** for real users via `addMedication()` (line 389-396, which calls `generateWeekDoses` → trailing week). The dashboard/medications screens compute adherence from this store (`getAdherenceForDate`, line 361).
- **Impact:** a user who adds a medication today sees 6 days of fabricated "taken" doses and inflated adherence. This contradicts the product's own invariant (known issues §3, ADR-006: unrecorded doses are not taken).
- **Fix:** never synthesize `TAKEN`. Use `MISSED`/`UNCONFIRMED` for past occurrences (or no historical rows at all), and derive historical adherence from server `DoseLogs`. Keep demo fixtures strictly behind guest mode with a visible "demo data" label. Add unit tests.
- **Estimate:** 1 day incl. tests.

### G6. AI provider privacy terms are not verified
- **Evidence:** Gemini 2.5 Flash is the production extraction provider (`app.ts:172-178`, `GEMINI_API_KEY`); `KNOWN_ISSUES.md` AI-001 still says production provider selection is deliberately open pending BAA/DPA/zero-training verification.
- **Why it blocks:** sending real prescriptions to a provider without contractual non-retention/no-training terms violates the stated privacy posture.
- **Fix:** obtain and record the provider's data-processing terms (paid tier if needed), document in `DECISION_LOG.md` + ADR-005, and gate `AI_EXTRACTION_PROVIDER=gemini` behind an explicit `AI_PROVIDER_APPROVED=true` flag in production. Until then, beta uses synthetic/own documents only.
- **Estimate:** procurement/legal, not engineering.

### G7. Committed storage credential in `render.yaml`
- **Evidence:** `render.yaml:34-35` — `STORAGE_ACCESS_KEY` value hardcoded in git (secret key is `sync: false`, correctly).
- **Fix:** rotate keys if the access key is treated as sensitive; set `sync: false` for both and enter in Render dashboard. Audit git history for any real secret (the `.env` files are gitignored — good).
- **Estimate:** 15 minutes.

### G8. Legal / DPDP artefacts
- **Evidence:** consent gate in `features/legal/LegalTermsModal.tsx`; landing page has terms modal text. No published privacy policy URL, data-fiduciary identity, grievance officer, retention schedule, or sub-processor list (Supabase, Render, Google).
- **Fix:** publish `/privacy` and `/terms` (landing repo is ideal), link them in app + store listings, name the data fiduciary and grievance contact, list sub-processors and regions, define retention. Have counsel/DPDP review.
- **Estimate:** mostly non-engineering.

---

## 4. P1 — reliability & launch quality

| # | Item | Detail / fix |
|---|---|---|
| P1-1 | Free-tier hosting | Render free spins down → 30–60 s cold start; Supabase free has connection/storage caps. For public prod, upgrade Render to a paid instance (≥512 MB) and Supabase to a paid plan with PITR backups, or move DB to the same region as Render (currently Singapore backend + `ap-south-1` DB — acceptable for India, verify latency). |
| P1-2 | JWT lifecycle | `auth-adapter.ts:222-241` hardcodes 7d access / 30d refresh; `JWT_EXPIRES_IN`/`REFRESH_TOKEN_EXPIRES_IN` env vars ignored. No `/auth/refresh` route; client never refreshes and just logs out on 401 (`api/client.ts:63-68`). Implement rotation + refresh route; shorten access to 15–60 min. |
| P1-3 | Hardcoded profile stub | `app.ts:108-122` returns `fullName: 'Test Patient'`. Wire to `PatientProfile` or remove the endpoint (client uses auth session). |
| P1-4 | Signup verification | `auth-adapter.ts:166` sets `isVerified: true`. Either implement verification or drop the column semantics and document open signup. |
| P1-5 | Observability | No error tracking, structured logging, metrics, or uptime alerts. Add Sentry (mobile + API), JSON logs with request IDs, `/healthz` deep check (DB connectivity), uptime monitor. Ensure PHI is never logged. |
| P1-6 | Integration tests in CI | Postgres+MinIO suite (6 tests) is skipped locally. Add a CI service-container job or a staging smoke test so migrations/transactions are exercised before deploy. |
| P1-7 | Local env points at PROD DB | `myhealthapp-backend/.env` uses the Supabase production `DATABASE_URL` with `NODE_ENV=development`. A local mistake can corrupt real data. Use Docker Postgres locally; reserve Supabase URL for deploy secrets only. |
| P1-8 | Landing waitlist is fake | `landing/index.html` handler only toggles a success message; no email is captured. Wire to a form service or relabel as a demo. |
| P1-9 | Store readiness | iOS build has only been produced as a simulator preview (`eas.json`), no App Store archive, privacy manifest, or Play listing. `RELEASE_CHECKLIST.md` §7. |
| P1-10 | Docs drift | Phase docs are stale; update `PROJECT_STATUS.md`/`BACKLOG.md` or mark them archived to stop future agents being misled. |

---

## 5. P2 — post-launch

- Offline queue for dose actions: currently optimistic local write + fire-and-forget POST (`medication-store.ts:339-350`); failed syncs are silently dropped. Add a durable retry queue with idempotency.
- `syncFromServer` only overwrites when the server list is non-empty (`medication-store.ts:376-384`) — stale local meds can persist after server-side deletion. Reconcile by ID.
- Timeline pagination is limited to 500 events in report export; evaluate for heavy users.
- Audit events table is specified in docs but not in `schema.prisma`; confirm whether audit logging is required by DPDP/your counsel.
- Accessibility audit (WCAG 2.2 AA), dynamic type, screen-reader pass.
- `npm audit` clean-up; pin/upgrade Expo SDK and Prisma on a schedule.

---

## 6. Recommended sequence to PROD

**Week 1 — unblock closed beta**
1. G5 (fabricated adherence) — clinical integrity.
2. G3 (secure token storage).
3. G7 (secrets hygiene) + P1-6 (integration tests in CI) + P1-7 (local env).
4. Add a visible "Beta — not medical advice, demo/synthetic data only" banner and gate real-patient data.

**Week 2 — unblock public launch**
5. G4 (rate limits/CORS) + P1-2 (JWT/refresh) → auth hardening.
6. G1 (account deletion) + G2 (password reset minimum).
7. G8 (published legal pages, consent links) — parallel, non-engineering.
8. G6 (AI provider terms) — parallel, procurement.

**Week 3 — operational readiness**
9. P1-1 (paid hosting/DB), P1-5 (Sentry + uptime + deep health), P1-3/P1-4 cleanups.
10. Full `RELEASE_CHECKLIST.md` pass on real devices (Android + iOS), store builds.

**Definition of done for public PROD**
- All P0 closed with regression tests.
- CI green including Postgres/MinIO integration suite.
- Working: signup → upload → AI draft → edit → confirm → timeline → activate med → reminder fires offline → dose taken/skipped → adherence accurate → PDF export/share → password reset → account delete (DB + storage verified empty).
- Legal pages published and linked; AI provider terms on file; uptime + error alerts firing.
