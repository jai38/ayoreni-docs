# Phase 4 Brief: App shell and rebrand

**Risk:** Medium. **Roadmap:** `docs/project/ROADMAP.md`, section 6 (Phase 4) and section 7 (W01, T1, Y1, Y2).
**Goal:** the app opens as Ayoreni into a five-tab shell built from the Phase 3 design system. Sign-in, medicines, reports and timeline still work, with clear error, offline and update screens, and the cost plan is re-priced from real usage.

**Status:** Complete, closed by the owner on 6 November 2026. Approved 5 November 2026, with every proposed default accepted. Both branches were pushed and merged to `main` on 6 November 2026: app `54b239c`, backend `0f28284` (adds the app-status settings to `infra/ecs.tf`, defaults 1.0.0, 1.0.0, false, message unset). The owner applied Terraform and the deploy workflow ran on 6 November 2026. Live `GET /api/v1/app-status` returns 1.0.0, 1.0.0 and maintenance off, with `no-store`.

## Decisions

The owner accepted all of these on 5 November 2026.

- **Expo SDK.** Upgrade from 52 to the latest stable SDK at the start of the phase, before any screen moves. The roadmap set this point for it, because it is cheaper before 87 screens exist. Opus checks the current version and breaking changes first (task 2). Expo Go on the owner's phone must then be updated to match.
- **App name.** The display name becomes **Ayoreni**. Bundle ID `com.myhealthapp.patient`, the slug, the URL scheme and the EAS project stay unchanged: nothing is in a store yet, and changing them now buys nothing while risking push and build setup. Rename them before the first store submission (Phase 31). **Update 7 Nov 2026 (owner decision):** renamed early, still before any store listing. Bundle ID and package are now `com.ayoreni.app`, and the scheme is `ayoreni`. Dev builds are a separate app, "Ayoreni Dev" (`com.ayoreni.app.dev`, scheme `ayoreni-dev`). The slug and EAS project are unchanged.
- **Tabs.** Today, Records, Medicines, Health Plan, Profile.

  | Today's screen | Goes to |
  |---|---|
  | `index.tsx` home | Today |
  | `timeline.tsx` | Records |
  | `reports.tsx` | Records |
  | `medications.tsx` | Medicines |
  | Account items | Profile |

  Health Plan shows a calm "coming soon" empty state until its phase.
- **Bridge, not redesign.** Existing screens are moved onto design-system components without changing what they do. New screen designs come in their own phases. The exceptions are T1 Today (first version), W01, Y1 and Y2.
- **Wave.** Used only on the white sign-in screen, behind the logo, at its current subtle strength. Off everywhere else.
- **Branch.** The owner merges `phase-3/design-system` into `main` first (task 1). Phase 4 then works on `phase-4/app-shell` from that `main`.

## Hard rules

1. No model opens `.env` or receives a key, cloud credential, production database access or real patient data.
2. Work goes on `phase-4/*` branches. No commits, pushes or merges unless the owner asks.
3. A backend push to `main` deploys to production. The backend change in task 9 merges only with the owner's go-ahead.
4. Any task that fails twice goes to the next model up, or to the owner.
5. All `CLAUDE.md` invariants hold. In particular:
   - Today never shows a past dose as Taken unless the user recorded it.
   - Notification taps never mark a dose Taken.
   - An unrecorded past dose shows as "Not confirmed" through `design/status.ts`.
6. Screens use only `design/` tokens and components. No new hard-coded colors, sizes or strings; every new string goes through `t()`.
7. Moving and splitting code must not change behavior. Each split task runs the full test suite before and after.

## State found on 5 November 2026

- **`myhealthapp`:**
  - `phase-3/design-system` is pushed at `0ee4a84` and not merged.
  - `main` is at `91126bc`.
  - 338 tests pass.
- **Old screens:**
  - `app/(tabs)/index.tsx` is 2,102 lines and `medications.tsx` is 1,324 lines.
  - There are four tabs.
  - `features/auth/AuthScreen.tsx` is the sign-in gate, called from `app/_layout.tsx`.
  - The old `components/` and `theme/` folders are still in use.
- **Brand assets:** the icons, splash and logo are already done (Phase 3). Only the display name is left.
- **Usage log:** `docs/phases/usage-log.md` has no rows yet. Re-pricing (task 12) needs the owner's real usage figures.
- **Desktop testing:** a desktop browser at `localhost` cannot sign in, because the AWS API accepts browser requests only from `https://jai38.github.io`. Phone checks go through Expo Go.

## Tasks

| # | Task | Who | Done when |
|---|---|---|---|
| 1 | Confirm the decisions above. Merge `phase-3/design-system` into `main`. | **Owner** | Written yes or changes; `main` contains `0ee4a84` |
| 2 | Check the latest stable Expo SDK and its React Native version. List breaking changes that affect this app: router, notifications, file system, SVG, fonts and the new architecture. | **Opus** | Upgrade note in Results with a go or no-go |
| 3 | Upgrade the Expo SDK with `npx expo install --fix`. Fix type and test breakage. The owner updates Expo Go on the phone. | **Sonnet** | Typecheck and tests pass; app opens in the new Expo Go |
| 4 | Display name "Ayoreni" in `app.json`, and the `brand.name` string checked | Haiku | Name shows under the icon in a dev build |
| 5 | Routing shell: five tabs using the Phase 3 `TabBar`, a shared screen header, and old routes redirected to their new tab | **Sonnet** | Every old screen is reachable; deep links still resolve |
| 6 | W01 launch and auth gate: restoring session, session expired, then the sign-in screen | **Sonnet** | Each state renders; expired tokens never show stale patient data |
| 7 | Split `index.tsx` and `medications.tsx` into feature modules under `features/`, with no behavior change | Haiku, one file per task | Neither route file is over 300 lines; tests unchanged and passing |
| 8 | Y2 global error and no-connection screens, plus an offline banner, with retry. Use `@react-native-community/netinfo`. | **Sonnet** | A simulated network drop shows the banner; retry recovers |
| 9 | Y1 force update and maintenance. The backend adds `GET /api/v1/app-status` with `minVersion`, `latestVersion` and `maintenance`. The app blocks below `minVersion`, nudges below `latestVersion`, and shows maintenance. The app fails open if the endpoint is unreachable. | **Sonnet** (backend + app) | Tests for required, optional, maintenance and unreachable |
| 10 | T1 Today, first version: greeting, today's doses from existing medicine data, and the states in the table below. Status labels come only from `doseStatusDisplay`. | **Sonnet** data hook, Haiku UI | Clinical regression test: an unrecorded past dose renders "Not confirmed" |
| 11 | Re-skin the bridge screens: sign-in and register, Records (timeline and reports), Medicines, Profile. Swap the old `components/` and `theme/` for `design/`, then delete the old folders once unused. | Haiku, one screen per task | No imports from `components/` or `theme/` remain |
| 12 | Paste real Claude usage for Phases 1–4 into `usage-log.md` | **Owner** | Rows filled from the console, not estimated |
| 13 | Re-price roadmap section 4 from the logged usage | **Opus** | Section 4 updated with the new range and the stop rule |
| 14 | Phone check of old flows and new states. See Tests. | **Owner** | Written approval |
| 15 | Review against `TEMPLATE-review.md` | **Opus** | `04-review.md` written |

Tasks 4, 7 and 8 can run in parallel once task 3 lands. Task 5 must land before 6, 10 and 11.

## Screens and required states

| Screen | Loading | Empty | Error | No connection | Success |
|---|---|---|---|---|---|
| W01 launch and auth gate | Logo on white while the session restores | n/a | Session expired → sign-in, with a calm note | Cached session opens with the offline banner; no session → sign-in with a retry | Opens Today |
| T1 Today | Skeleton cards | New user: add first medicine. No medicines today: calm "nothing due" | Card-level error with retry | Last loaded data, offline banner and stamp | Due now, several due, all done; past unrecorded doses show "Not confirmed" |
| Y1 update and maintenance | n/a | n/a | n/a | Not shown: fails open | Required update (blocking), optional update (dismissible), maintenance |
| Y2 global error and offline | n/a | n/a | Friendly error with retry and a support code (no health data) | Banner on every tab, and the screen keeps working with what it has | Banner clears on reconnect |
| Health Plan tab | n/a | "Coming soon" empty state | n/a | n/a | n/a |
| Bridge screens | As today | As today | As today, restyled | Offline banner | As today, restyled |

## Tests

- **Gates:** `npm run typecheck` and `npm test` in `myhealthapp`, plus the backend for task 9. Counts recorded in Results.
- **New regression tests:**
  - An unrecorded past dose on Today is "Not confirmed", never Taken or Missed.
  - The auth gate never renders patient data after expiry.
  - The app-status client: required, optional, maintenance, and fails open when unreachable.
  - The offline banner shows and clears.
  - Old route paths redirect to their new tabs.
- **Owner phone check (Expo Go, new SDK):**
  1. Fresh open shows the splash, then sign-in.
  2. Sign in. Today loads.
  3. Add a medicine and confirm no past dose shows as Taken.
  4. Upload a report and see it in Records.
  5. Turn on airplane mode: the banner shows, then clears when you reconnect.
  6. Try the largest text size.

## Review

Opus reviews the diffs and gate results once at the end, using `TEMPLATE-review.md`.

## Results

- Task 1:
  - The owner accepted all defaults on 5 November 2026.
  - The merge of `phase-3/design-system` into `main` is done: `origin/main` is `0ee4a84` (checked in the review).
- Task 2, Expo upgrade note (Opus, 5 November 2026). **Verdict: go, in steps, to SDK 57 now and SDK 58 later.**
  - **Where things stand:**
    - The latest stable is SDK 58: React Native 0.88, released 15 October 2026 ([changelog](https://expo.dev/changelog/sdk-58)).
    - The store Expo Go supports only one SDK at a time.
    - Expo Go for SDK 58 is not in the stores yet ("within a few weeks"), so the store build still runs SDK 57.
    - Testing SDK 58 today would need a development build, which means an EAS or Android Studio build and is costly for this phase.
  - **Plan:**
    - Upgrade one SDK at a time, 52 → 53 → 54 → 55 → 56 → 57.
    - At each step run `npx expo install expo@^N --fix`, `npx expo-doctor@latest`, typecheck and tests.
    - Commit each step on its own, so a failure points at a single SDK.
    - Move to 58 as a small follow-up once the SDK 58 Expo Go is in the Play Store.
    - The owner installs the SDK 57 Expo Go from the Play Store (replacing the SDK 52 build) for the phone check.
  - **What helps:**
    - The project uses Continuous Native Generation: there are no `android/` or `ios/` folders, so nothing native has to be migrated by hand.
    - `newArchEnabled` is already true, and the legacy architecture is going away.
    - Local Node is 22.20, which meets SDK 58's minimum of 22.13 or later.
  - **Known breakage to handle:**
    - `expo-file-system`:
      - Its API changed in SDK 54.
      - `app/(tabs)/reports.tsx` and `features/reports/ReportExportModal.tsx` import the old API. Switch them to `expo-file-system/legacy` first, then the new `File` API in a later phase.
      - From SDK 58, `File.write()` is async.
    - React 19 and Expo Router major versions: route typing and layout APIs change, and SDK 58 removes much of the forked React Navigation API. `app/_layout.tsx` and `app/(tabs)/_layout.tsx` are the main touch points.
    - Android edge-to-edge became the default, then mandatory. Check that the safe areas on `Screen` and `TabBar` still hold.
    - Notifications:
      - The foreground behaviour changes in SDK 58.
      - `services/notifications/notification-service.ts` sets a handler. Keep it explicit, and add a regression test that delivery never marks a dose Taken (rule 14).
      - Remote push stopped working in Expo Go on Android from SDK 53. Local reminders are unaffected. Push testing needs a development build (Phase 5 already needs one for passkeys).
    - The `react-native-svg` and `@expo-google-fonts/inter` versions move with `--fix`. Recheck the logo mask and the gallery after the upgrade.
  - **Later benefit (noted for the reminders phase):** SDK 58 adds an Android `delivery: 'alarmClock'` option for scheduled notifications, which battery savers don't defer. That suits medicine reminders.
- Task 3, Expo SDK 52 → 57 (Sonnet, 5–6 November 2026; resumed after a laptop restart):
  - **Commits** on `phase-4/app-shell`, one per SDK: `f16a4ba` 53, `2eb9658` 54, `cf3d420` 55, `afa3ef6` 56, `b7523e0` 57, and the test commit `d28fd9e`.
  - **Now on:** React Native 0.86.3, React 19.2, Expo Router 57 and TypeScript 6. expo-doctor is clean at SDK 57.
  - **Gates, rechecked by Opus:** typecheck exit 0; tests 339 passing in 29 files.
  - **Changes beyond the version bumps:**
    - A committed `.npmrc` with `legacy-peer-deps=true`. Expo Router's optional peers otherwise fail `npm ci`.
    - Four files moved to the `expo-file-system/legacy` import: reports, report export, the document picker and upload.
    - The splash moved into the `expo-splash-screen` plugin.
    - `newArchEnabled` removed; it is always on now.
    - `absoluteFillObject` replaced with `absoluteFill`.
    - The removed `StatusBar` `backgroundColor` prop dropped.
    - `@expo/vector-icons` and `expo-splash-screen` became direct dependencies.
    - The `tsconfig` `baseUrl` removed.
    - The notification handler adds `shouldShowBanner` and `shouldShowList`.
  - **New test:** `test/invariants/notification-foreground-handler.test.ts` proves the foreground handler never marks a dose Taken.
  - **Follow-ups:**
    - The splash plugin draws the image at a fixed width (200 dp by default), and our `splash.png` has wide padding around the mark, so the mark may look small. Re-crop to the mark only when the splash is first seen in a build.
    - `npm audit` warnings are not yet triaged.
    - CI Java 17 is believed sufficient; confirm on the first `main` CI Android build after merge.
  - **Phone check needed (SDK 57 Expo Go):**
    - Document upload and the picker.
    - PDF export and share.
    - Android edge-to-edge safe areas and the status bar.
    - Foreground reminder banners.
    - Tapping a notification opens the medicine.
    - The logo and gallery.
- Task 4 (Opus, 6 November 2026), `92dde4a`:
  - The display name is now "Ayoreni".
  - The Account version line and the shared-report title now say Ayoreni.
- Task 7 (Haiku), `480e8d9` and `86f9ca5`:
  - `app/(tabs)/index.tsx` went from 2,102 lines to 4. It is now `features/today/`: 19 files, the largest 309 lines.
  - `medications.tsx` went from 1,324 lines to 4. It is now `features/medications/`: 9 new files.
  - Opus checked that each original line appears verbatim. Typecheck passes, and tests were unchanged at each commit.
  - Still oversized: `MedicationDetailModal.tsx` (1,357 lines) and `ManualAddMedicationModal.tsx` (510 lines).
- Task 8 (Sonnet), `36fc049`, merged as `314990d`:
  - NetInfo connectivity: unknown reachability is treated as unknown, not offline.
  - A calm offline banner with Retry, on the signed-in app.
  - A design-system global error screen with an opaque support code `AY-…`.
  - The error boundary no longer stores or logs the error message. Retry re-mounts the tree.
  - Not covered yet: the sign-in screen has no banner. W01's offline state is part of task 6.
- `78ce5a7`: Vitest and tsc now ignore agent worktrees under `.claude/`, which were double-counting tests.
- Gates after the merge (Opus): typecheck exit 0; 348 tests passing in 30 files.
- **Existing bugs found by the split (not fixed; behavior was kept as is):**
  1. **Clinical, and should be fixed this phase:** when the server fails to confirm a prescription, the app swallows the error and still says "Prescription confirmed!", then adds the medicines locally. The patient believes a record is saved that the server may not have.
  2. **Clinical, and should be fixed this phase:** reminder scheduling differs between Home and Medicines. Home syncs active medicines with an end date. Medicines syncs every medicine with `isActive` and no end date. Whichever screen loads last reschedules the alarms from different data.
  3. "Next dose" is picked differently on Home (due, then snoozed, then overdue, then upcoming) and on Medicines (the first actionable dose). Today (task 10) should own one rule.
  4. A full-screen early return makes the review `Modal` on Home dead code.
  5. `allDoses` state is set on both screens but never read.
- Bug fixes (Sonnet, reviewed by Opus):
  - **Bug 1, `9617f0c`:**
    - `confirmPrescriptionDraft` now adds medicines locally only after the server confirms.
    - On failure the review stays open with a calm inline error, and nothing is added or scheduled.
    - Test: `test/prescription-confirmation.test.ts`.
  - **Bug 2, `1e6a296`:**
    - A shared `buildReminderSyncPayload` (`services/medications/reminder-sync.ts`), used by both Home and Medicines.
    - It includes only ACTIVE medicines with an unfinished course.
    - Opus checked that `syncAllMedicationReminders` cancels every `med-*` and `snooze-*` alarm before rescheduling, so leaving out inactive medicines still clears their alarms.
    - Test: `test/reminder-sync.test.ts`.
  - **Gates (Opus):** typecheck exit 0; 353 tests passing in 32 files.
  - **Follow-ups:**
    - Duplicates are prevented by a database unique constraint on `jobId`. But a confirm whose response was lost will make the retry fail with a conflict, even though the record exists. Fix: the server returns the existing confirmation for the same patient and job, or the client treats that conflict as success after re-fetching. Backend change, owner-gated deploy.
    - `buildMedicationRecord` never sets `endDate` from `duration_text`, so a locally added medicine has no end date until the next server sync.
- **Task 5 (Sonnet, reviewed by Opus), `be0286c`:**
  - Five tabs: Today, Records, Medicines, Health Plan, Profile. The design-system `TabBar` is connected to Expo Router through `features/shell/RouterTabBar.tsx`, and every tab uses the shared `ScreenHeader` (design `TopBar`). The old emoji icons and `theme/` tab styling are gone.
  - Records: a Timeline / Reports switch over the old screens, which were moved unchanged into `features/records/`. `/timeline` and `/reports` redirect to `/records?view=…` and are hidden from the bar.
  - Medicines keeps the `medications` route, so notification taps (`medicineOpenTarget`) still open the medicine and record nothing.
  - Health Plan: a "coming soon" empty state. Profile: `AccountContent` extracted from the account sheet, which still opens from Today.
  - The header drops its top inset while the offline banner shows, because the banner already covers the status bar.
  - Test: `test/shell-tabs.test.ts` (tab order, title keys, every old path maps to a tab).
  - **Gates (Opus):** typecheck exit 0; 364 tests passing in 33 files. The web bundle exports.
  - **Not seen yet:** the shell has not been viewed in a browser or on a phone (local Docker was down). Check it in task 14.
  - **Follow-ups:**
    - Records still shows the old screens' own titles under the new header and switch. Task 11 removes them.
    - Switching segments resets the other view's filters and scroll.
    - Task 10 can point Today's avatar at the Profile tab instead of the sheet.
- **Task 6 (Sonnet, reviewed by Opus), `9e09af3`, `17926be`, `2135522`:**
  - A pure gate (`features/auth/authGate.ts`): restoring, signed out, expired, signed in. The layout renders from it, and `theme/` is gone from `app/_layout.tsx`.
  - Restoring shows the Ayoreni mark on white. Expired shows the sign-in screen with "Your session ended. Please sign in again." Signing out on purpose shows no note.
  - On expiry the session and the medication store are cleared synchronously before any re-render, and the signed-in tree unmounts. No other module-level patient caches exist.
  - Signed out and offline: the sign-in screen shows the offline banner, and an offline sign-in gives a calm message.
  - **Sent back once (Opus):** the first version signed patients out by the phone's clock after the 15-minute access token, including offline. Fixed in `17926be`:
    - The app now refreshes through `POST /auth/refresh`, one refresh at a time, retrying the failed request once.
    - The session expires only when the server rejects the refresh or there is no refresh token. A network failure keeps the session.
    - A 401 for an old token cannot expire a newer session.
    - Signing out on purpose cancels medicine reminders and calls server logout (best effort). Expiry keeps reminders.
  - Opus fixed a race where a sign-out during a refresh could bring the session back, with a regression test.
  - **Gates (Opus):** typecheck exit 0; 389 tests passing in 35 files.
  - **Not verified:** not run against the real backend or on a phone. Check in task 14: sign in, wait 20 minutes, reopen; and open in flight mode.
  - **Follow-ups:**
    - After an offline start with an old token, the refresh waits for the next 401; there is no reconnect trigger.
    - Signed out, the medication store loads guest demo medicines (synthetic, behind the unmounted tree). Revisit.
    - AuthScreen gained the wave and a white background; the rest is re-skinned in task 11.
- **Task 10 (Sonnet model, Haiku UI, reviewed by Opus), `2860b2b`, `a2e775c`, `d3f9170`:**
  - One dose model (`features/today/todayModel.ts`), used by Today and Medicines:
    - PENDING is upcoming before its time, then **Due now** for `DUE_WINDOW_MINUTES` (60), then **Not confirmed**. It is never Taken or Missed.
    - A snoozed dose becomes Not confirmed 60 minutes after its reminder time.
    - Taken shows the user's recorded `actionTime` (falling back to the record's `updatedAt`).
    - Paused or stopped medicines are left out.
  - One next-dose rule, `pickNextDose`: due now, then snoozed, then upcoming. A Not confirmed dose is never "next". The old OVERDUE state and `getSpotlightDose` are gone.
  - `useToday()` gives the screen state (loading, new user, nothing due, due, all done, error), grouped doses, greeting, `isOffline` and `lastUpdatedAt`, and refreshes the clock every minute.
  - Today rebuilt on `design/` only:
    - The greeting; the avatar opens the Profile tab.
    - A next-dose card with Taken, Snooze 15 min and Skip.
    - Due now, then "Earlier today" (Not confirmed, with Taken / Skip), later, and done.
    - As-needed doses, quick actions, and a card-level error with retry.
    - Status pills come only from `doseStatusDisplay`.
    - Fourteen old Today files were deleted.
  - Removed from Today: the date picker, adherence numbers and recent activity. The day schedule stays in Medicines; records are in Records.
  - Clinical regression test: an unrecorded past dose is `notConfirmed` and renders `status.notConfirmed`.
  - Opus replaced a test that passed on a code comment with a real check of the wiring.
  - **Gates (Opus):** typecheck exit 0; 414 tests passing in 37 files.
  - **Not seen yet:** not viewed in a browser or on a phone (local Docker was down). Check it in task 14.
  - **Owner decisions:** is the 60-minute "Due now" window right? Should any of the removed Today sections come back?
  - **Follow-ups:**
    - `useDoseActions` and `usePrescriptionUpload` still have hard-coded English messages (task 11).
    - The prescription picker, manual add and review screens are not on `design/` yet (task 11).
    - `a2e775c` carries a Haiku co-author line instead of Opus.
- **Task 11 (Haiku, one area per agent, reviewed by Opus):** every screen moved onto `design/`, with no behavior change. Typecheck and tests were re-run after each area.
  - Unit 1 auth `23153d2`: sign-in, register, legal terms on design/. 414 tests. Not fixed: AuthModal passes untrimmed email; trust copy says S3, AES-256 and "hosted in Mumbai" (true only if prod is on AWS now; owner to confirm wording).
  - Unit 2 profile `f3ab81c`: account items as iOS-style lists, split into three components. Copy claims to check: "End-to-End Encrypted Storage" (it is AES-256 at rest and TLS, not end-to-end), "HIPAA Aligned", "never sell"; version hard-coded "1.0.0"; sign-out alert shows raw error message.
  - Unit 3 timeline `5817706`: timeline list, filters, date strip and detail on design/. Verification labels now follow `isVerified` (old hard-coded "Patient Confirmed" claimed confirmation for every entry; Opus accepts this as a rule-10 fix). Filters stay fixed above the list. Not fixed: date strip cells look tappable but do nothing; `formatMedicalEventDate` has hard-coded English.
  - Unit 4 reports `aafdcdc`: reports view and export modal on design/, split into five components; report content unchanged. Not fixed: export modal header ignores the top safe area; quick export and preview use different report titles; "Private S3 Storage" wording; CLAUDE.md still says SDK 52.
  - Unit 5 medicines list `e582e83`: Medicines tab, next dose, day schedule, weekly card and list on design/; dose pills now from the shared rule (unrecorded past doses show Not confirmed). Old styles files deleted. **Clinical bug found (rule 15):** `getAdherenceForDate` divides taken by every scheduled dose, so unrecorded doses lower the percentage like misses (weekly bars, Medicines, timeline date strip dots). Fix after task 11 (Sonnet). Also: "Daily" shown for every medicine; Active tab count excludes paused; `useMedicationsData` notices hard-coded English with emoji.
  - Unit 6 add medicine `c63e9a1`: manual add on design/, custom times via `TimeField` ("HH:mm" kept). Start date stays a free-text field (DateField would change what is saved). **Bug to fix with the adherence fix:** reminder scheduling failures are swallowed, yet the success message says reminders are active.
  - Unit 7 medicine detail `1f951a1`: the 1,357-line modal split into `features/medications/detail/` (hook plus six cards, all under 400 lines); handlers moved unchanged (agent diffed them). Dose pills from the shared rule. **Bug for the Sonnet fix:** Take/Skip/Snooze still show for doses of a paused or stopped medicine.
  - Unit 8 prescriptions `f3eb832`: review screen split from 1,317 lines into `features/prescriptions/review/`. The picker and preview moved to `features/documents/`. UNCERTAIN medicines now show `UncertainMarker` (the field was stored but never shown, against rule 13). Pure review logic has tests.
  - `4b2699a` (Opus): error boundary and system screens moved to `features/system/`; `components/` and `theme/` deleted. No imports from them remain.
  - `8172dd1` (Opus): the Inter fonts were only loaded by the dev gallery, so every design screen fell back to a serif font on web (seen in the browser). The root layout now loads them.
  - **Bug fixes found by the re-skin (Sonnet, reviewed by Opus):**
    - `53dce41`: adherence counts only recorded doses. Unrecorded doses show as "not confirmed", and a day with nothing recorded shows "No data", never 0% (rule 15).
    - `f7311a8`: says reminders are active only when every time was scheduled.
    - `d0fee5c`: no dose actions on paused or stopped medicines.
    - `cfd7d55`: review reminder times must be valid `HH:mm`.
    - `68019db`: picker test files only in dev builds.
  - `ae2920d` (Opus): two notification logs printed the medicine name; they now log the ID only (rule 8), with an invariant test.
  - `a64d2b1` (Opus): the sign-in, register and sign-out buttons say Ayoreni.
  - **Gates (Opus):** typecheck exit 0; 469 tests passing in 43 files. The web bundle exports. Sign-in seen in the browser; the signed-in screens were not (no local backend).
  - **Owner decisions:**
    - Wording of the security and privacy claims: "End-to-End Encrypted", "HIPAA Aligned", "private S3", "hosted in Mumbai".
    - The legal text still says "MyHealth".
    - CLAUDE.md still says Expo SDK 52 (now 57).
  - **Follow-ups:**
    - Some tests for fixes 2, 3 and 5 read source files, because the repo has no component-render tests. Consider adding React Native Testing Library.
    - `useMedicationsData` and `useDoseActions` notices are still hard-coded English with emoji.
    - The base64 sample files still ship in the release bundle; only the buttons are gated.
    - The commits by Haiku agents carry the Opus co-author line, as instructed.
- **Task 9 (Sonnet, reviewed by Opus):**
  - **Backend `981a540`** on the new local branch `phase-4/app-status`, cut from `main`. Not pushed or merged.
    - Public `GET /api/v1/app-status` with `Cache-Control: no-store`, returning `{ data: { minVersion, latestVersion, maintenance: { active, message? } } }`.
    - Settings `APP_MIN_VERSION`, `APP_LATEST_VERSION` (strict `x.y.z`, default `0.0.0`), `APP_MAINTENANCE` (default false) and `APP_MAINTENANCE_MESSAGE`. The defaults never block anyone.
    - Route-isolation test updated.
    - Gates: typecheck clean; 350 passing, 6 skipped (the Docker integration suite).
    - The owner's uncommitted `infra/envs/v1.tfvars` change was left alone. The backend working copy is now on `phase-4/app-status`.
  - **App `a35f151`:**
    - The pure decision is: maintenance, then required update, then optional update. It fails open on unreachable, malformed or timed-out (4 s) responses.
    - Checked at launch and on returning to the foreground, at most every 5 minutes, without delaying launch.
    - Required update and maintenance are blocking full screens. The optional update card is dismissible per version.
    - Store links point to the landing page until store listing. The owner should confirm `https://jai38.github.io/MyHealth/`.
    - Gates (Opus): typecheck clean; 500 tests passing in 44 files.
  - **Owner, to ship:**
    - Merge the backend branch, which deploys.
    - Add the four settings to the ECS task definition: `APP_MIN_VERSION=1.0.0`, `APP_LATEST_VERSION=1.0.0`, `APP_MAINTENANCE=false`, `APP_MAINTENANCE_MESSAGE` unset.
    - An invalid version makes the container fail to start.
  - **Phone check (task 14):** set maintenance on, then a higher minimum, then a higher latest version on a test stack, and reopen the app each time.
  - **Not verified:** screens not rendered; the optional card and offline banner may double the top inset.
- **Task 13 (Opus):** re-priced roadmap section 4 on Pro plan limits (owner decision, 5 Nov 2026).
  - The build has no per-token bill: the cost is the subscription fee, and the limit is the weekly cap.
  - First reading: 5-hour window 51%, weekly cap 21%, covering the end of Phase 3 and most of Phase 4.
  - Projection: 15–40% of a weekly cap per phase.
  - The new stop rule replaces the $200 hard stop: stop and re-plan when one phase needs more than a full weekly cap, or two weeks in a row hit the cap.
  - `usage-log.md` gained a plan-limit readings table, filled after each phase.
  - Task 12 is satisfied by that reading, because Pro shows no per-token figures.
- **Task 14 (Owner):** the owner checked the old flows and new states on the phone (SDK 57 Expo Go) and approved them for v1 on 6 November 2026.
- **Task 15 (Opus):** `docs/phases/04-review.md` written on 6 November 2026. Verdict: pass with follow-ups. No High findings and no blocking items.
- **Review fixes, 6 November 2026:**
  - **App `54b239c`:**
    - Dose lists hide Taken and Skip for paused, stopped and future doses.
    - A failed save is undone and shown as an error.
    - Doses with no valid time show "Time unknown".
    - Reminders no longer add "Prepare to take with water.", and the medicine name stays off the lock screen.
    - Confirmed medicines take the server's values instead of guessed ones.
    - Unrecorded doses never colour a timeline day.
    - The end-to-end, HIPAA, DPDP and GDPR claims are replaced with accurate wording, and user-visible notices moved into strings.
  - **Backend `7a5a629`:** startup fails on an `APP_MAINTENANCE` value other than `true` or `false`, or a minimum version above the latest.
  - **Gates:**
    - App: typecheck pass; 525 tests passing in 46 files.
    - Backend: typecheck pass; 359 passing, 6 skipped (the Docker integration suite).
  - **Each finding's status is in the review's findings table.**
- **Phase 4 complete (owner, 6 November 2026).**
  - **Not verified:**
    - The review fixes on a phone.
    - CI, because neither branch is pushed.
    - The app-status flow against a real stack.
  - **Owner decisions still open:**
    - "MyHealth" six times in the legal text.
    - The store link `https://jai38.github.io/MyHealth/`.
    - "Hosted in Mumbai" and "AES-256 in private S3 buckets".
    - Whether to keep the Profile privacy badge.
  - **Carried to later phases:**
    - Render tests (finding 18).
    - As-needed doses saved only on the phone (finding 2).
    - The phone copy of a confirmed medicine, which matches server medicines by the name as written (finding 3).
    - The follow-ups in the review verdict.
  - **Owner, to ship:**
    - Push and merge both branches. An app `main` push deploys GitHub Pages and an EAS APK, and a backend `main` push deploys to production.
    - Add the four app-status settings to the ECS task definition.
