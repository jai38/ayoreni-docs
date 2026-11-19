# Phase 4 Review: App shell and rebrand

**Reviewer:** Opus. **Date:** 6 November 2026. **Brief:** `docs/phases/04-brief.md`.
**Branches reviewed:** `myhealthapp` `phase-4/app-shell` at `a35f151`, diffed against `0ee4a84` (now also `origin/main`). `myhealthapp-backend` `phase-4/app-status` at `981a540`, one commit on top of `origin/main` (`b93cf5a`). `landing`: not touched by this phase and not reviewed.

## 1. Brief match

- [x] Every task in the brief's table has a result in its Results section, with two gaps in the record, not in the work:
  - Task 1 still says the merge is to do. `origin/main` is now `0ee4a84`, so it is done. The Results line is out of date.
  - Task 14, the owner phone check, has no written approval in Results. The owner approved it on the phone, as the review request says. Write that into Results.
  - Task 15 is this review.
- [x] Each "done when" holds, with evidence:
  - **Task 4:** `app.json:3` has `"name": "Ayoreni"`. The name under the icon needs a dev build to see, because Expo Go shows its own name. Not seen in this review.
  - **Task 5:** five tabs, with old paths redirecting. `app/(tabs)/_layout.tsx`, `app/(tabs)/timeline.tsx`, `app/(tabs)/reports.tsx` and `features/shell/tabs.ts`. Tests in `test/shell-tabs.test.ts`: "has the five tabs in order", "maps /timeline and /reports to the right Records segment", and "a reminder tap opens the medicine on the Medicines tab and carries no dose action".
  - **Task 6:** `features/auth/authGate.ts` and `app/_layout.tsx:69-107`. The signed-in tree is not rendered at all unless the gate is `signedIn`. Tests:
    - `test/auth-gate.test.ts`, "the auth gate never renders patient data after expiry" (5 cases).
    - `test/auth-refresh.test.ts`: 12 cases. They cover one shared refresh, a network failure keeping the session, a stale-token 401 not expiring a newer session, an old `expiresAt` while offline keeping the session, and the sign-out race.
  - **Task 7:** `app/(tabs)/index.tsx` and `medications.tsx` are 4 lines each. The two oversized modals are now 128 lines (`MedicationDetailModal.tsx`) and 365 lines (`ManualAddMedicationModal.tsx`). No file under `features/` or `app/` is over 400 lines.
  - **Task 8:** `services/network/connectivity.ts` and `features/system/ConnectivityBanner.tsx`. Tests in `test/connectivity.test.ts`: "shows only when offline and clears on reconnect", and "unknown, not offline, when reachability is null". Retry recovery has no test, because there are no render tests. The owner checked it with flight mode.
  - **Task 9:** tests on both sides.
    - App, `test/app-status.test.ts`: required, optional, maintenance, unreachable, non-200, malformed, timeout, and "server defaults (0.0.0) never block".
    - Backend, `test/app-status.test.ts`: defaults, configured versions, maintenance message, and invalid version strings rejected.
  - **Task 10:** `test/today-model.test.ts`, "CLINICAL: an unrecorded past dose is notConfirmed, never taken or missedByUser". `test/today-screen.test.ts`, "a not-confirmed dose uses the notConfirmed label and is never Taken or Missed".
  - **Task 11:** `components/` and `theme/` are deleted. No file in `app/`, `features/` or `services/` imports them. The only `../components/` imports are `design/`'s own components.
  - **Task 13:** recorded in Results.
- [x] All work is on `phase-4/*` branches. Neither branch is pushed (there is no `origin/phase-4/*`). Nothing in this phase merged to `main`. `main` only gained the Phase 3 merge, which task 1 asked for.
- [x] Scope. Beyond what the brief named, these files changed:
  - `.npmrc`, `tsconfig.json`, `vitest.config.ts` and `.gitignore`, all recorded in tasks 3 and 8.
  - Small changes to `services/documents/upload-service.ts` (the legacy file-system import), `services/api/client.ts` (refresh) and `services/notifications/notification-service.ts` (handler fields and log wording).
  - All of these are recorded in Results.
- [x] No `dist/` file changed. The backend change sits only in the app-status module, `app.ts`, `env.ts` and tests. The owner's uncommitted `infra/envs/v1.tfvars` was left alone and not opened.

## 2. Rules (`CLAUDE.md`, Non-negotiable rules)

- [x] **1-2 AI tools and secrets.**
  - No `.env` or key was opened in this review.
  - Test data is synthetic. Signed out, the guest demo medicines are synthetic.
  - The backend tests use the mock auth adapter and memory storage. The MinIO integration suite skipped without Docker.
- [x] **3 Identity from the JWT.** The app sends no `patientProfileId` in any request body or query. `app-status` is public and reads no identity.
- n/a **4 Cross-patient 404.** No owned-resource route changed. The route-isolation invariant now lists `GET /app-status` as public.
- n/a **5-7 Storage keys, signed URLs, upload validation.** Not changed. Only the import moved to `expo-file-system/legacy`.
- [x] **8 No secrets in the bundle; no health data in logs.**
  - No new secret. The API base URL fallback is a public URL.
  - `ae2920d` removed medicine names from two notification logs, and `test/invariants/no-medicine-names-in-logs.test.ts` now guards that.
  - The error boundary logs only the support code (`features/system/ErrorBoundary.tsx:31`).
  - The app-status handler logs nothing.
  - One older risk is left: finding 17.
- [x] **9 `validateProductionConfig()`.** Untouched: the diff has no lines in it. The new settings are checked by the env schema at startup, which fails closed (finding 11).
- [x] **10 AI output is a draft.**
  - The timeline now shows verification labels from `isVerified`, replacing a hard-coded "Patient Confirmed".
  - Bug 1 (`9617f0c`) adds medicines only after the server confirms.
- [x] **11 Historical prescriptions.** Only medicines the patient marked active are added locally (`features/today/prescription-confirmation.ts:23-28`). Reminders are built only from ACTIVE medicines with an unfinished course (`services/medications/reminder-sync.ts:34`).
- [ ] **12 `nameAsWritten` separate from `normalizedMedicineName`.** Partly. The local copy made after confirmation sets `normalizedMedicineName` to `nameAsWritten` when it is missing (finding 3). This is older code, moved unchanged, and the server copy replaces it on the next sync.
- [ ] **13 Flag uncertain values; never guess.** Partly.
  - Better than before: `UncertainMarker` now shows for UNCERTAIN medicines, and review times must be valid `HH:mm`.
  - Older guesses remain: findings 3, 4 and 16.
- [x] **14 Notifications never mark Taken.**
  - Taps only navigate: `app/_layout.tsx:57-61`, and the `MedicationsScreen.tsx:63` handler only opens the medicine.
  - Tests: `test/invariants/notifications-never-mark-taken.test.ts` and `notification-foreground-handler.test.ts`.
- [x] **15 No synthesized adherence.**
  - `todayModel.ts:75-78` handles PENDING: upcoming, then Due now for 60 minutes, then Not confirmed. It never becomes Taken or Missed.
  - A snoozed dose becomes Not confirmed 60 minutes after its reminder time (lines 69-72).
  - `pickNextDose` never picks a Not confirmed dose (lines 107-122).
  - Adherence (`53dce41`, `medication-store.ts:408-444`) counts recorded doses only. A day with nothing recorded shows `null`, displayed as "No data" (`WeeklyAdherenceCard.tsx:41`).
  - Taken requires an explicit tap.

Notes on invariants that changed:
- Rules 12 and 13 are not newly broken. The gaps are older behaviour that the split made visible, and they are listed as Medium findings.
- This phase added no new user-visible clinical claim. The older ones are findings 3 and 4.

## 3. Gate results

| Gate | Command | Result | Notes |
|---|---|---|---|
| Mobile typecheck | `npm run typecheck` in `myhealthapp` | Pass, exit 0, 0 errors | At `a35f151` |
| Mobile tests | `npx vitest run` in `myhealthapp` | 500 passing in 44 files, 0 skipped, exit 0 | Matches Results |
| Backend typecheck | `npm run typecheck` in `myhealthapp-backend` | Pass, exit 0, 0 errors | At `981a540` |
| Backend tests | `npm test` in `myhealthapp-backend` | 350 passing, 6 skipped, exit 0 | 22 files passed. 1 file skipped: the Docker integration suite, because no containers were running |
| Backend build | `npm run build` in `myhealthapp-backend` | Pass, exit 0 | `dist/` is gitignored; the working tree is unchanged apart from the owner's `v1.tfvars` |
| CI | GitHub Actions | Not run | Neither `phase-4/*` branch is pushed, and workflows run on `main` |
| Migrations | n/a | n/a | No schema change in either repo |
| Owner demo | Task 14 phone check | Done (owner approved on the phone) | Not yet written in Results |

## 4. Findings

| # | Severity | Where | Finding | Owner or model to fix | Status |
|---|---|---|---|---|---|
| 1 | Medium | `features/medications/DaySchedule.tsx:136-158` | The Medicines day schedule still shows Taken and Skip for doses of paused or stopped medicines. `d0fee5c` fixed only the detail card (`DoseHistoryCard.tsx:61`), although its message and `dose-display.ts:24` say the list matches. The row also offers Taken for any day picked in the weekly card, including future days, and the pending count includes paused medicines' doses. The patient can record a dose for a medicine they stopped, or a dose that has not happened yet. Fix: gate the row on `canActOnDose` and on the dose not being in the future. Add a test. | Sonnet | Fixed, `54b239c`: Taken and Skip hidden for paused, stopped and future doses. Test: `test/dose-review-fixes.test.ts` |
| 2 | Medium | `features/today/useDoseActions.ts:11-27`; `services/medications/medication-store.ts:390-392, 454` | Two older behaviours (moved unchanged), the same kind of problem as Bug 1. First, an as-needed dose is saved only on the phone (`addDoseLog`) and never sent to the server, yet the notice says "Logged dose … taken". The next server sync replaces the phone's dose list and drops it. Second, a Taken, Skip or Snooze that the server fails to save keeps the phone's copy with no message, and the next sync reverts it. The patient believes a record exists that does not. | Sonnet, in the medicines or reminders phase | Partly fixed, `54b239c`: a failed Taken, Skip or Snooze is undone and shown as an error. As-needed doses still save only on the phone (no server endpoint), and the notice now says so. Carried to the medicines or reminders phase |
| 3 | Medium | `features/today/utils.ts:34, 38, 40, 42, 43, 47` | After the server confirms, the phone builds its own copy of each medicine and guesses missing values: reminder `08:00`, `normalizedMedicineName = nameAsWritten` (rule 12), `TABLET`, `ORAL`, "Take as prescribed", and `patient-local`. This copy is shown, and can schedule an 08:00 reminder, until the next server sync. Fix: re-fetch the confirmed medicines from the server instead of building them on the phone. | Sonnet | Partly fixed, `54b239c`: the phone copy takes the server's values, matched by the name as written, and guesses nothing when there is no match. Re-fetching from the server is still the cleaner fix. Test: `test/prescription-local-copy.test.ts` |
| 4 | Medium | `services/notifications/notification-service.ts:159-181` | When a medicine has no instructions, the 15-minute advance reminder says "Prepare to take with water." That is a clinical instruction the prescription never gave, and it is wrong for inhalers, drops, injections and sublingual tablets (rule 13). The titles and bodies are also hard-coded English with emoji, outside `t()`, and they put the medicine name on the lock screen. | Haiku (drop the default line now); Sonnet (lock-screen privacy, reminders phase) | Fixed, `54b239c`: default line removed, medicine name off the lock screen, text through `t()`. Test: `test/notification-service.test.ts` |
| 5 | Medium | `design/strings/en.ts:315, 322` | Profile says "End-to-End Encrypted Storage" and "protected with end-to-end encryption". This is inaccurate: data is encrypted at rest and over TLS, and the server can read documents for extraction. | Owner decides the wording; Haiku swaps the string | Fixed in copy, `54b239c`: replaced with "encrypted at rest and in transit" wording |
| 6 | Medium | `design/strings/en.ts:262, 266, 321`; `features/account/AccountInfoSections.tsx:15-16, 58` | "HIPAA Aligned" on Profile, and "HIPAA & DPDP Aligned" plus HIPAA, DPDP and GDPR claims in the privacy text. No assessment backs these claims. | Owner | Fixed in copy, `54b239c`: HIPAA, DPDP and GDPR claims removed. Open, owner decision: keep or remove the Profile badge (key still named `hipaaBadge`) |
| 7 | Low | `design/strings/en.ts:195, 221, 289, 467` | "Secure cloud, hosted in Mumbai", "stored in private S3 buckets", AES-256, and "Private S3 Storage" on the report export. These are plausibly true since the AWS cutover (the API is in `ap-south-1`), but the owner has not confirmed them. "Private S3 Storage" is internal jargon on a patient screen. | Owner | Open, owner decision pending |
| 8 | Low | `design/strings/en.ts:244, 248, 251, 257, 266, 280` | The terms and privacy text still say "MyHealth", six times. Legal wording needs the owner and a legal check. | Owner | Open, owner decision pending |
| 9 | Low | `CLAUDE.md:15` | Still says "Expo SDK 52, React Native 0.76". It is now SDK 57, React Native 0.86.3, React 19.2 and TypeScript 6. Not edited in this review. | Owner or Opus | Open, owner to ask for the edit |
| 10 | Low | `services/app-status/storeLinks.ts:3` | The Update button opens `https://jai38.github.io/MyHealth/`, which uses the old name. Until store listing, a required update has nowhere real to send people, so avoid raising `APP_MIN_VERSION` before then. | Owner to confirm | Open, owner to confirm |
| 11 | Low | `myhealthapp-backend/src/config/env.ts:44-47, 50-55` | **Checked and true:** an invalid `APP_MIN_VERSION` or `APP_LATEST_VERSION` fails the env schema, and `process.exit(1)` stops the container. That covers `1.2`, `v1.2.3`, `1.2.3-beta` and an empty string, all tested. The error output carries Zod messages only, never values. The defaults (`0.0.0`, maintenance off) never block anyone: the app test "server defaults (0.0.0) never block" covers it. Two small gaps remain. First, `APP_MAINTENANCE` treats anything except `true` as off, so `1`, `yes` or `on` silently mean no maintenance. That fails open, which is safe for patients, but a typo goes unnoticed. Second, nothing checks that latest ≥ min. Owner note: leave settings unset rather than blank, because a blank version stops the container. | Sonnet (strict enum `true`/`false`, plus a min ≤ latest check) | Fixed, backend `7a5a629`: `APP_MAINTENANCE` accepts only `true` or `false`, and min above latest fails startup. Test: `test/app-status.test.ts` |
| 12 | Low | `features/medications/useMedicationsData.ts:37-38, 50, 86-97`; `features/today/useDoseActions.ts:27, 32, 49-61`; `features/today/usePrescriptionUpload.ts:31, 39, 44, 71-77, 104`; `features/today/prescription-confirmation.ts:2` | Notices and alerts are still hard-coded English, some with emoji (✓, ⏰), bypassing `t()`: "marked as taken!", "Action Failed", "Prescription confirmed!", "AI Clinical Parser". `handleRecordDoseAction` is copied in two hooks. In `useMedicationsData`, the load errors are swallowed (`.catch(() => [])`), so its `error` state never shows. | Haiku | Fixed, `54b239c`: notices moved into strings, one shared `runDoseAction`, load errors now shown |
| 13 | Low | `services/auth/auth-service.ts:168, 202, 208`; `services/api/client.ts:68-81`; `services/documents/file-validator.ts:32, 40`; `services/timeline/timeline-service.ts:143`; `features/today/utils.ts:3-15` | More user-visible English outside `t()`. Server error text is shown raw in `AuthModal.tsx:96`. `formatTimeDifference` is unused. | Haiku | Fixed, `54b239c`: text moved into strings; server error text no longer shown on sign-in |
| 14 | Low | `features/account/AccountContent.tsx:107` | The Profile version is hard-coded `'1.0.0'`. The update check reads `Constants.expoConfig.version`, so the two can disagree after a version bump. | Haiku | Fixed, `54b239c`: version read from `Constants.expoConfig` |
| 15 | Low | `features/records/adherenceDots.ts:15` | The timeline dot turns "partial" (caution) when one dose was taken and another is Not confirmed. A day with nothing taken and every dose Not confirmed stays neutral. So an unrecorded dose colours the dot only when another dose was taken, against the file's own comment. Not a rule-15 breach, because no miss is shown, but it is inconsistent. | Opus decides in the adherence phase | Fixed, `54b239c`: unrecorded doses never colour a day. Test: `test/adherence-counts.test.ts` |
| 16 | Low | `features/today/todayModel.ts:55, 59` | A dose with a missing or invalid `scheduledTime` becomes time 0 and shows "Not confirmed" with a 1970 time. A Taken dose with no `actionTime` and no `updatedAt` shows the scheduled time as the taken time. Both should show "time unknown" (rule 13). | Sonnet | Fixed, `54b239c`: missing times show "Time unknown". Test: `test/dose-review-fixes.test.ts` |
| 17 | Low | `services/documents/upload-service.ts:96, 221` | `console.warn` prints the raw file-system error, which can contain the file path or name. A patient's file name can hold health data (rule 8). Older code. | Haiku | Fixed, `54b239c`: logs the error type only |
| 18 | Low | `test/` (whole app) | There are still no component render tests. Several fixes (`f7311a8`, `d0fee5c`, task 10 wiring) are checked by reading source text. Carried from Phase 3, finding 1. | Opus decides in the next brief (React Native Testing Library) | Open, next brief |
| 19 | Info | `features/today/prescription-confirmation.ts:12-30` | Bug 1 (`9617f0c`): medicines are added only after the server confirms, and the draft stays open on failure. Test: `test/prescription-confirmation.test.ts`. | Sonnet | Fixed |
| 20 | Info | `services/medications/reminder-sync.ts:24-43` | Bug 2 (`1e6a296`): one payload builder, used by `useTodayData.ts:89` and `useMedicationsData.ts:48`. The server sends `endDate` as `YYYY-MM-DD`, so the end-of-day check is valid. | Sonnet | Fixed |
| 21 | Info | `services/medications/medication-store.ts:408-444` | Adherence fix (`53dce41`): recorded doses only, with a `null` percentage shown as "No data". Test: `test/adherence-counts.test.ts`. | Sonnet | Fixed |
| 22 | Info | `design/strings/en.ts:38-44` | The wording holds: "Not confirmed" for unrecorded past doses, and "Marked missed" only for an explicit user mark. No other "Missed" string exists. | n/a | Holds |
| 23 | Info | `services/auth/auth-service.ts:23-70` | Tokens are still in AsyncStorage, and in `localStorage` on web. This is known (roadmap line 53) and is Phase 5's "secure token storage". | Phase 5 brief | Open, Phase 5 |
| 24 | Info | `docs/phases/04-brief.md` Results | Task 1 still reads "merge to do", and task 14's approval is not written down. | Opus | Fixed at close-out: Results updated |

**Counts:** Medium 6, Low 12, Info 6. Nothing is High.

**Fix status, 6 November 2026:** fixes are in app `54b239c` on `phase-4/app-shell` and backend `7a5a629` on `phase-4/app-status`. App gates: typecheck pass, 525 tests in 46 files. Backend gates: typecheck pass, 359 passing, 6 skipped (Docker suite). The fixes have not been seen on a phone, and there are no render tests.

## 5. Verdict

**Pass with follow-ups.** No clinical or security rule is breached by new code in this phase:
- Today, adherence, notification taps, the auth gate and app-status all hold their rules, with tests.
- The Medium items are older behaviour the split made visible (findings 1-4), or copy that waits on owner decisions (5-6).

- **Blocking items:** none.
- **Follow-ups carried to the next phase:**
  - **Before any real patient uses the app:** findings 1, 2, 3 and 4 (Sonnet), and the owner's wording decisions on 5, 6, 7 and 8.
  - **Before raising `APP_MIN_VERSION`:** findings 10 and 11.
  - **Haiku string and cleanup tasks:** 12, 13, 14 and 17.
  - **Smaller items:**
    - 9: update `CLAUDE.md`.
    - 15: the adherence phase decides.
    - 16: Sonnet.
    - 18: the next brief.
    - 23: Phase 5.
    - 24: Opus updates Results.
  - **Carried from Results:**
    - The prescription confirm retry conflict (backend, owner-gated).
    - `endDate` from `duration_text`.
    - The SDK 58 move once its Expo Go ships.
    - The splash re-crop.
    - `npm audit` triage.
    - Shipping the base64 sample files in release builds.
    - The refresh having no reconnect trigger.
    - The optional card and offline banner doubling the top inset.
  - **Owner, to ship task 9:**
    - Merge `phase-4/app-status`, which deploys.
    - Add the four settings to the ECS task definition.
    - Leave the maintenance message unset rather than blank.
- **Future change cost.** The owner wants later copy and layout cuts, such as showing less text on a page, to be cheap, small, screen-level changes that reuse `design/` components and existing strings.
  - **This mostly holds now.**
    - Screens are thin compositions of `design/` parts under `features/`, and most are under 300 lines.
    - Screen text is in `design/strings/en.ts`.
    - Labels come through `t()`, and the dose pills come from `doseStatusDisplay`.
    - So hiding a section or shortening a line is usually a one-file screen edit, or a one-line string edit.
  - **Where it is not true yet:**
    - **Text that lives in logic, not strings:**
      - `useMedicationsData.ts` and `useDoseActions.ts` (notices and alerts).
      - `usePrescriptionUpload.ts` (progress, error and confirmation text).
      - `prescription-confirmation.ts:2`.
      - `notification-service.ts:159-181` (reminder text).
      - `auth-service.ts` and `api/client.ts` (error text shown to users).
      - `file-validator.ts`, and `timeline-service.ts:143`.
      - Changing any of this copy means editing hooks and services, and their tests, rather than a string.
    - **Server text shown as-is:** `AuthModal.tsx:96` and `usePrescriptionUpload.ts:75`. The screen cannot shorten these.
    - **The legal and privacy text:** long paragraphs in `en.ts:240-295`, rendered by `LegalTermsModal.tsx` (309 lines). Cutting it is cheap as strings, but each change needs owner and legal sign-off.
    - **Duplicated code:** the dose-action handler is copied in two hooks, so a notice change has to be made twice.
    - **Larger screens, where a layout cut touches more code:**
      - `useMedicationDetail.ts` (400 lines).
      - `ManualAddMedicationModal.tsx` and `DocumentPreview.tsx` (365 each).
      - `AuthScreen.tsx` and `AuthModal.tsx` (over 320 each).
    - **No render tests (finding 18):** layout cuts cannot be checked automatically. They still need an eye check on the phone.
  - **Recommendation:** fold findings 12 and 13 into one Haiku task that moves every user-visible string into `en.ts` and merges the two dose-action hooks. After that, copy cuts stay at the string or screen level.
- **Not verified in this review:**
  - CI, because the branches are not pushed.
  - The backend integration suite, because Docker was not run.
  - Any device or browser render of the signed-in screens. I relied on the owner's phone approval.
  - The display name under the icon, which needs a dev build.
  - The app-status flow against a real stack.
  - TalkBack, VoiceOver and the largest text size beyond the owner's check.
