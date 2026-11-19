# Phase 4b Brief: Simple, modern screens

**Risk:** Medium. **Roadmap:** `docs/project/ROADMAP.md`, section 5 (design direction) and section 6 (Phase 4b).
**Goal:** a new user opens Ayoreni and, without help, sees what to take now and how to add a medicine. Every tab fits the phone cleanly: nothing overlaps or clips, there's little text, and it feels like a current app, not a 2010 one.

**Status:** Approved by the owner on 6 November 2026, with Health Plan hidden until Phase 18. Tasks 1–4 are done. **The design is final** (19 November 2026) in the owner's work-account canvas, "AYORENI · Calm Glass UI v4": https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9. It is the only design source for this build; the earlier mockups (`S8f1FKrbV875uhqYMSRS8G` and the 26-screen "App v2" set) are superseded. The build tasks below (5–22) replace the old tasks 5–15. Several are blocked on the owner decisions in "Build decisions (19 November 2026)".

## Why this phase exists

The owner used the Phase 4 build on the phone on 6 November 2026. The components are right, but the screens are overwhelming and feel dated. They follow the designer's text-heavy concept too closely. What the owner saw:

- **Today:**
  - The "Starting the AI reader" notice overlaps the "All done for today" banner.
  - The page scrolls even when there's nothing to do.
  - When scrolled, only the greeting moves up, and nothing stays at the top.
- **Records:** too many buttons (All, Milestone, All years and more) before any content.
- **Medicines:**
  - A long scroll: next dose, weekly card, day schedule and the list all stacked.
  - The "Prescribed" count sits alone on its own row, while the other counts share a line.
- **Profile:**
  - A long scroll of paragraphs.
  - Button text overflows its button.
  - Too many things to do.
- **First use:** it isn't obvious how to add a medicine or how to use the app.

The owner is fine with leaving the designer's concept behind. Phase 5 adds sign-in, passkey and recovery screens on top of this shell, so the screens get fixed now, before more are built the same way.

## Decisions (approved 6 November 2026; see the build decisions for what the final design changes)

1. **Leave the designer's layout behind, keep the brand.** Keep the Phase 3 tokens, colors, Inter, icons, logo and components. Change what each screen holds and how it's arranged. Section 5's content rules still apply: calm status wording, no pill pictures, and white backgrounds.
2. **Mockups before code.** Opus designs every changed screen as a clickable phone-size page. The owner opens it on the phone and approves or changes it before any screen is built (tasks 3 and 4).
3. **Rules for every screen.** These replace the "Layout" line in roadmap section 5.
   - **One main action per screen**, in the same place on every tab.
   - **Today fits one screen** in its normal states (nothing due, one or two due, all done) on a 360 × 800 phone at default text size. It may scroll only when many doses are listed.
   - **A large title that stays at the top** and shrinks into a small bar as you scroll, iOS-style, on every tab.
   - **One notice at a time.** Upload progress, "saved", errors and "all done" share one slot that takes turns, so they can never stack or overlap.
   - **Short text.** A button label is one to three words and never wraps or clips. A card has one line of heading and at most two lines of detail. Paragraphs live one tap away, never on a tab.
   - **Details one tap away.** Filters live behind a Filter button. History, the weekly view and legal text go one level down.
   - **Checked at the hard sizes.** Nothing overlaps or clips at 360 × 800, on a large phone, and at the largest system text size.
4. **Adding is always one tap.** A single **Add** button on Today and Medicines opens a small sheet with *Scan a prescription* and *Add by hand*. A new user's Today shows these two choices and three short tips, and nothing else.
5. **Tabs.** Today, Records, Medicines and Profile. **Health Plan is hidden until its phase (18),** because a tab that only says "coming soon" adds noise (owner decision, 6 November 2026).
6. **Records is one list.** The timeline becomes the Records screen, with search and one Filter button. Reports becomes a row near the top that opens its own page. The Timeline/Reports switch and the year and category chips go away from the main view.
7. **No new features and no backend change.** Data, medicine rules, reminders and upload flow stay exactly as they are. This phase changes layout, navigation depth and wording only.
8. **Branch.** `phase-4b/simple-screens` in `myhealthapp`, cut from `main` (`54b239c`).

## Build decisions (19 November 2026)

The final canvas holds seven screens, S01–S07, in two flows. It differs from this brief's earlier plan and from the roadmap in ways the owner must confirm before the affected tasks start. The lead session's recommendation follows each one.

| # | The final design… | Conflicts with | Recommendation | Blocks |
|---|---|---|---|---|
| D1 | Signs in with a **mobile number and SMS OTP** (S01, S02) | Roadmap §5 and Phase 5: email, password and passkey; SES email only, no SMS provider | Owner decides the sign-in method. If mobile OTP: Phase 5 is rewritten, and an SMS provider plus India DLT sender registration are needed (Owner). **Build S01–S04 in Phases 5 and 6**, where their backend lands, not in 4b: a sign-in screen with no endpoint can't ship (working agreement, rule 5) | 17–20 |
| D2 | Uses a **new visual system**: Plus Jakarta Sans, a new palette, glass cards with mesh backgrounds, and dark mode | Roadmap §5 tokens, Inter, white backgrounds; brief decision 1 ("keep the brand") | Accept, since the owner called the design final, and update roadmap §5 on approval. Keep Noto Sans Devanagari for Hindi (Plus Jakarta Sans has no Devanagari). Fix the contrast failures listed under task 6 | 6 onward |
| D3 | Puts **appointments, readings, supplements, search and a notification bell** on Today and in the Add choices | Brief decision 7 (no new features, no backend change); these features belong to Phases 11, 14 and 18 | Build Today and the Add sheet with what exists now: medicines and records. **Hide** the appointment card, Add Appointment, Add Reading, search and the bell until their phases, as Health Plan was hidden. Show the Supplement tag only once a medicine can carry that type (D-gap 6) | 10, 11, 12 |
| D4 | **Doesn't design Records, Medications or Profile** (only their tab icons; "Profile has 9 sections" is mentioned) | Brief tasks 9 and 10, and STATUS ("source of truth for every screen") | Restyle the current three tabs onto the v4 tokens and the 4b layout rules in this phase; design them properly in the canvas before their own phases. The alternative is to design them now (Opus, about 3–5% of a weekly cap) and build them here | 14, 15, 16 |
| D5 | Marks **S07 Dose update "In review"**, not locked | "Design is final" | Owner locks S07 in the canvas before task 13 starts | 13 |
| D6 | Lets **Today scroll** (S06 is 1,700 pt tall: three dose groups, an appointment, Quick Actions) | Brief decision 3 ("Today fits one screen") | Drop the one-screen rule for Today; keep "nothing overlaps or clips" and the single notice slot | 10 |
| D7 | Shows a **"2 of 6 taken" ring with a percentage** on Today | Brief clinical choice "no percentages"; `CLAUDE.md` rule 15 | Keep the count, which comes only from recorded Taken doses; drop the % label or confirm it | 10 |

What the design gets right clinically, and the build must keep: "Not confirmed" for an unrecorded past dose ("Tap to update"), Missed only when the user says "No, I missed it", Taken from an explicit action only, an Undo after Mark as Taken, the note "This only updates your log", and the allergy states "No known allergies" and "Unknown" kept distinct from an empty list.

## Hard rules

1. No model opens `.env` or receives a key, cloud credential, production database access or real patient data. Mockups and screenshots use synthetic data only.
2. Work goes on `phase-4b/*` branches. No commits, pushes or merges unless the owner asks. A push to app `main` deploys GitHub Pages and builds an APK.
3. Any task that fails twice goes to the next model up, or to the owner.
4. All `CLAUDE.md` invariants hold. In particular:
   - No dose shows as Taken unless the user recorded it.
   - Notification taps never mark a dose Taken.
   - An unrecorded past dose shows "Not confirmed" through `design/status.ts`, never "Missed".
   - Paused, stopped and future doses offer no Taken or Skip.
   - Moving a dose list to a new place keeps these rules and their tests.
5. Screens use only `design/` tokens and components (the v4 tokens from task 6). A new shared piece goes into `design/components/` with a gallery entry. Every string goes through `t()`.
6. The owner's open wording decisions (legal text "MyHealth", the store link, the "Mumbai/AES-256/S3" claims, the privacy badge) are not decided here. Moving that text one level down doesn't change its wording.
7. Fixtures and screenshots use the `CLAUDE.md` synthetic set, not the names drawn in the canvas.
8. A feature hidden under D3 is not built half-way: no tile, button or empty card for it.

## State found on 6 November 2026

- **`myhealthapp`:** `main` is at `54b239c`, the Phase 4 close-out, deployed with the pre-release `v0.3.0-phase4-preview`. 525 tests pass. There are no component render tests (Phase 4 review, finding 18).
- **Screen files:**
  - Today: `features/today/HomeScreen.tsx`. It renders up to 13 blocks in one scroll: greeting, offline stamp, notices, processing, five state cards, four dose sections, as-needed doses and quick actions.
  - Records: `features/records/` (the Timeline/Reports switch, `TimelineFilters.tsx`, `TimelineDateStrip.tsx` and `ReportsHero.tsx`).
  - Medicines: `features/medications/MedicationsScreen.tsx`, with the next dose, weekly card, day schedule and list in one scroll.
  - Profile: `features/account/AccountContent.tsx` and its sections.
  - The shared header is `features/shell/ScreenHeader.tsx`. It's a fixed `TopBar` that doesn't shrink on scroll and doesn't hold the title in place on Today.
- **Desktop testing:** a desktop browser can't sign in to the AWS API, because CORS only allows GitHub Pages. Screens behind sign-in have so far been seen only on the phone. Task 2 fixes this for screenshots.
- **Backend:** no change needed. The Gemini 3.5 Flash-Lite extraction was approved by the owner on the phone on 6 November 2026.

## Tasks

Tasks 1–4 are done (see Results). The old tasks 5–15 are replaced by these. **Who** is UI (the UI session, `myhealthapp`), Backend (the backend session, `myhealthapp-backend`) or Owner. The model column follows `CLAUDE.md`.

| # | Task | Who | Model | Done when |
|---|---|---|---|---|
| 5 | Decide D1–D7 above, and lock S07 in the canvas | **Owner** | n/a | Written answer to each |
| 6 | **v4 tokens and theme** in `design/`: the colour tokens (light and dark), Plus Jakarta Sans type scale, radii, elevation and mesh background from the canvas's Main board; a light/dark theme that follows the system. Fix the contrast failures first: text-tertiary `#8A97A8` on `#F7FAFC` is 2.8:1 and on white 3.0:1; teal text `#028265` on teal tint is 4.35:1; action blue `#0071FF` on white is 4.37:1 for 13–15 pt links. Pick shades that pass 4.5:1 and record them in the canvas | **UI** | Sonnet | Gallery shows every token in both themes; the Phase 3 contrast and touch-size checks pass |
| 7 | **Render tests** with React Native Testing Library and Vitest (Phase 4 finding 18), with checks that fail if two notices render at once or a button label is wider than its button at the largest text size | **UI** | Sonnet | Setup lands with one passing render test per tab |
| 8 | **Shared pieces** in `design/components/`, each with a gallery entry in both themes: glass card, hero card, section header with count, medicine row (icon tile, name, detail line, status chip, sub-line, chevron), status chip (icon plus word for Taken, Due now, Upcoming, Not confirmed, Skipped, Marked missed), Quick Action tile, bottom sheet with grab handle, radio option row, segmented control, time wheel, toast with Undo, pill tab bar (Today, Records, Medications, Profile), large title that shrinks on scroll, single notice slot | **UI** | Sonnet | Gallery shows each in every state; render tests pass |
| 9 | **Dev preview fixtures** for every state in the table below, in both themes, with synthetic data only: rename the canvas's people and medicines to the `CLAUDE.md` set ("Asha Test", "Testmed A 500 mg", "Dr. Test Clinician") | **UI** | Haiku | `/dev/preview` renders every state |
| 10 | **S06 Today, returning user**, built on `todayModel.ts` and `doseStatusDisplay` unchanged: greeting by time of day with first name, hero card (taken count, next dose, "N needs confirming"), Morning / Afternoon / Evening groups with counts, Mark as Taken on a due dose, toast with Undo, Quick Actions (Upload Record, Add Medication only, per D3). Uploading, saved and error share the notice slot | **UI** | Sonnet | Clinical tests unchanged and passing; both S06 states match the canvas in both themes |
| 11 | **S05 Welcome, first-time Today**: welcome hero with Get Started, Quick Actions, "Complete your health profile" card (states: incomplete, complete). The card shows only once S04 exists (Phase 6, per D1); until then it's hidden | **UI** | Haiku | The three S05 states match the canvas |
| 12 | **Add sheet** ("What would you like to add first?"): Health Record (upload) and Medication or Supplement (add by hand), Maybe Later. Reading and Appointment hidden per D3. Opens from Get Started, the Quick Action tiles, and Medications. Offline: upload disabled with a reason; add by hand still works | **UI** | Haiku | Sheet opens from each entry point; offline state shown |
| 13 | **S07 Dose update sheet**: Did you take this dose? Yes (as scheduled, just now, or another time on a wheel), No I skipped it (reason chips, optional), No I missed it; Save disabled until a choice; the "only updates your log" note. Opens from a Not confirmed or due dose only: paused, stopped and future doses never open it (`canActOnDose` gate). Blocked by task 5 (D5) and task 21 | **UI** | Sonnet | The four S07 states match; dose-action and clinical tests pass |
| 14 | **Records** restyled onto v4 (or rebuilt to a new design, per D4): one list, search, a Filter button, Reports as its own page. Old `/timeline` and `/reports` links still resolve | **UI** | Haiku | Render test passes; no clipping at the largest text size |
| 15 | **Medications** restyled onto v4 (per D4): active list first, counts on one line, schedule and weekly view one tap down, dose actions behind `canActOnDose` and the future-dose gate. The tab label becomes "Medications" | **UI** | Sonnet wiring, Haiku layout | Dose-action tests unchanged and passing |
| 16 | **Profile** restyled onto v4 (per D4): name card plus grouped rows; privacy, legal and help text one level down, wording unchanged | **UI** | Haiku | No paragraph on the tab; labels fit at the largest text size |
| 17 | **S01 Mobile number** (states: default, ready, invalid number, sending code). Build in Phase 5 if D1 says so | **UI** | Sonnet | Deferred unless the owner moves it into 4b |
| 18 | **S02 OTP** (default with autofill, code entered, verifying, wrong code with attempts left, resend available, expired). Phase 5 | **UI** | Sonnet | Deferred, as 17 |
| 19 | **S03 Create your account** (default, filled, errors on Continue, date-of-birth wheel, self-describe gender). Phase 5 | **UI** | Sonnet | Deferred, as 17 |
| 20 | **S04 Your health profile** (default, yes → add allergy, allergy saved, no known allergies, not sure; Skip for now). Phase 6 for height, weight and blood group; Phase 16 for allergies | **UI** | Sonnet | Deferred, as 17 |
| 21 | **Dose action fields for S07 and the S06 Undo**, only if `api.md` can't already carry them (D-gaps 1–4): recorded time for Taken, an optional skip reason, a Missed action chosen by the user, and undo of the last action. Fill in the request and response shapes for `/medications/doses` and the action route in `api.md`. Regression tests: no route marks Taken without an explicit call, and undo restores Not confirmed, never Missed | **Backend** | Sonnet | PR merged after owner OK; `api.md` updated in the same PR |
| 22 | **Copy pass**: every new string through `t()`, shortened to the text rules | **UI** | Haiku | `en.ts` diff only; no button label over three words |
| 23 | After screenshots of every state below in `docs/phases/04b-screens/after/`, in both themes, at 360 × 800, 402 × 874 and the largest text size | **UI** | Sonnet | Every state in the table below has a screenshot |
| 24 | Deploy the backend change from task 21, if any (`deploy-aws.yml`) | **Owner** | n/a | Dev, then production, green |
| 25 | Phone check (see Tests), then approve | **Owner** | n/a | Written approval |
| 26 | Review against `TEMPLATE-review.md` | Lead | Opus | `04b-review.md` written |

**Order:** 5 first. 6, 7 and 9 can start at once; 8 needs 6. Screens 10–16 need 8. Task 13 waits for 5 (S07 locked) and 21 (if needed); 21 is checked first against the dose action route, and the UI opens a `from-ui` issue for whatever is missing. 22 runs before 23. Merge order: backend, owner deploy, then the app.

## Screens and required states

Every state below is drawn in the canvas in light and dark unless marked *(not drawn)*; those follow roadmap §5 and need a fixture and a screenshot all the same.

| Screen | Canvas states (light and dark) | States not drawn, still required | In 4b? |
|---|---|---|---|
| **S06 Today, returning** | Default (a dose due now, one Not confirmed); after Mark as Taken (toast with Undo, count updated) | Loading (skeleton hero and rows); nothing due today; all done; many doses; error with Retry in the notice slot; offline (cached, stamp, Mark as Taken disabled with a reason); uploading in the notice slot | Yes |
| **S05 Welcome, first time** | Profile incomplete; Get Started sheet open; profile complete | Offline; uploading | Yes (profile card hidden until Phase 6) |
| **Add sheet** (inside S05) | Four options: Health Record, Medication or Supplement, Reading, Appointment | Two options (D3); offline with upload disabled | Yes |
| **S07 Dose update** | Ask (Save disabled); Taken as scheduled; Taken at another time (wheel); Skipped with reason | Missed selected; saving; save failed with Retry; offline | Yes, once locked (D5) |
| **Records, Medications, Profile** | *(not drawn in this canvas)* | Loading, empty, filtered empty, error, offline, success, per the old table | Restyle only (D4) |
| **S01 Mobile number** | Default; ready (number and consent); invalid number; sending code | Offline; too many requests | Phase 5 (D1) |
| **S02 OTP** | Default with keyboard and autofill; code entered; verifying; wrong code (2 attempts left); resend available; expired | Locked after too many attempts; offline | Phase 5 (D1) |
| **S03 Create your account** | Default; filled; errors on Continue; date-of-birth wheel; Other / self-describe | Under 18; saving; error | Phase 5 (D1) |
| **S04 Your health profile** | Default; Yes → add allergy (search suggestions); allergy saved; No known allergies; I'm not sure | Saving; error; allergen not in the list ("Add as typed") | Phases 6 and 16 (D1) |

## Data the new screens need that `api.md` doesn't have

`api.md` lists routes but no request or response shapes yet, so "missing" below means it isn't in the contract; the backend session confirms against the zod schemas and documents what already exists. The UI raises each as a line in `ui-needs.md` plus a `from-ui` issue, linking the canvas board.

| # | Screen | Need | Phase |
|---|---|---|---|
| 1 | S06, S07 | Dose fields: scheduled time, status, recorded time (`takenAt`), strength, quantity and form for the detail line ("5 mg · 1 tablet · 1:00 PM") | 4b (document what exists) |
| 2 | S07 | Taken at a user-chosen time ("as scheduled", "just now", "other time") | 4b, task 21 |
| 3 | S07 | Skip reason: felt unwell, doctor advised, ran out, side effects, other (optional) | 4b, task 21 |
| 4 | S07, S06 | A user-chosen **Missed** action (the contract lists Taken, Skipped and Snoozed only) and **Undo** of the last action, restoring Not confirmed | 4b, task 21 |
| 5 | S05, S06 | The user's first name for the greeting (`/auth/me` shape undocumented) | 4b (document what exists) |
| 6 | S06, Add sheet | A medicine type, medicine or supplement, for the Supplement tag | Phase 14 |
| 7 | S06 | Appointments (doctor, visit type, specialty, date and time) | Hidden in 4b (D3); Phase 18 |
| 8 | Add sheet | Readings (BP, glucose, weight) | Hidden in 4b (D3); Phase 11 |
| 9 | S05, S06 | Notifications and an unread count for the bell | Hidden in 4b (D3) |
| 10 | S05, S06 | Global search across records and medicines (`/timeline` searches records only) | Hidden in 4b (D3); Phase 10 |
| 11 | S01, S02 | Mobile OTP: request a code, verify it, resend cooldown, attempts left, expiry, rate limits; an SMS provider | Phase 5, if D1 says mobile |
| 12 | S03 | Account profile: full name, date of birth (under-18 check), gender with an optional self-description, optional email, the masked verified number, and a consent record (version and time) | Phases 5 and 6 |
| 13 | S04 | Health profile: height (cm or in), weight (kg or lb), blood group (eight groups or "I don't know"), allergy status (has allergies, none known, unknown) | Phase 6 |
| 14 | S04 | Allergies: allergen as typed kept apart from the matched name, reaction, severity (mild, moderate, severe, not sure); add, edit, delete; an allergen search for suggestions | Phase 16 |
| 15 | S05 | Profile completeness ("3 details left") | Phase 6 |

## Tests

- **Gates:** `npm run typecheck` and `npx vitest run` in `myhealthapp` (and in `myhealthapp-backend` for task 21), with counts in Results.
- **New tests:**
  - A render test per screen, covering each 4b state in the table above, in both themes.
  - Only one notice renders at a time; the toast never stacks with an upload notice.
  - Button labels fit at the largest text size.
  - The Add sheet opens from Get Started, the Quick Action tiles and Medications, and shows only the two live choices.
  - Hidden features (appointments, readings, search, the bell) don't render.
  - Old routes `/timeline`, `/reports` and `/medications` still resolve.
  - S07: Save stays disabled until a choice; a paused, stopped or future dose can't open it; Undo after Mark as Taken restores Not confirmed (or Due now), never Missed.
  - The taken count on the hero counts recorded Taken doses only.
- **Clinical tests that must still pass unchanged:**
  - An unrecorded past dose renders "Not confirmed".
  - Notification taps never mark Taken.
  - Paused, stopped and future doses show no Taken or Skip.
  - Adherence counts recorded doses only.
  - Missed appears only after the user chose "No, I missed it".
- **Owner phone check (new APK), light and dark:**
  1. Sign in with a fresh test account. Can you add a medicine without help from the welcome screen?
  2. Today with one dose due: tap Mark as Taken, then Undo. Does it return to Due now?
  3. Tap a Not confirmed dose and record it as taken at another time, then as skipped with a reason.
  4. Upload a prescription. Progress shows in one place, with nothing overlapping.
  5. Records, Medications and Profile: no clipped or overflowing text.
  6. Set the largest system text size and repeat steps 2 to 5.

## Review

Opus reviews the diffs, the before and after screenshots, and the gate results once at the end, using `TEMPLATE-review.md`.

## Cost

About the size of Phase 3 for the 4b screens: an estimated 15–25% of a weekly cap, mostly Sonnet layout work. Restyling Records, Medications and Profile adds little; designing them first (D4) adds 3–5%. Record the plan-limit reading in `usage-log.md` at the end.

## Results

- Task 1: the owner approved every decision on 6 November 2026 and chose to hide Health Plan.
- Task 2 (Sonnet, checked by Opus):
  - Dev-only `/dev/preview/<tab>?state=&fontScale=`, gated by `__DEV__` like the gallery. It renders the real screens from synthetic fixtures (`features/dev-preview/`) with a frozen clock (Fri 6 Nov 2026, 10:30), a fake API over `fetch`, and an in-memory session.
  - The only production-file changes are a dev-only path check in `app/_layout.tsx`, an exported context, and a preview seam in `usePrescriptionUpload.ts` that is null in release.
  - `scripts/screens.mjs` captured 14 states at 360 × 800, 430 × 932 and 360 × 800 with the font scale at 2, saved in `docs/phases/04b-screens/before/` with `index.json` measurements.
  - Adds devDependency `playwright-core` 1.62.0.
  - **Gates (Opus):** typecheck pass; 545 tests in 47 files.
  - **Not covered:** sheets and modals, loading skeletons, the upload review screen, and real Android font behaviour.
- **Task 3 audit (Opus, 6 November 2026),** from the before screenshots and `index.json`:
  - **Today:**
    1. Upload progress and "All done" render as two stacked cards (`allDone-uploading`).
    2. The page scrolls with one dose due (by 324 px at 360 × 800).
    3. The greeting takes two lines at 34 px, and only the greeting moves on scroll; there's no fixed title.
    4. Names wrap ("Testmed A 500 / mg"), and status pills wrap ("Taken at / 7:05 am").
    5. Scan and Add sit as two oversized grey buttons at the bottom, disabled during upload.
    6. Many doses means five stacked sections.
  - **Records:**
    1. About 800 px of controls before the first record: the Timeline/Reports switch, Share PDF, the date strip, a labelled search box and two chip rows (five categories, five years).
    2. "TODAY" is clipped to "TOD…", and the search hint is cut off.
    3. At the largest text size, summaries clip and "Clinical PDF" runs off the edge in Reports.
  - **Medicines:**
    1. "Add medicine" is a large secondary button at the top.
    2. Before the list: the next-reminder card (it repeats Today), the weekly chart with two lines of help text, then Today's schedule.
    3. The list scrolls 2,417 px, and the empty state still scrolls 565 px.
    4. "Snooze 15m" wraps onto two lines.
    5. The Active/Prescribed counts don't line up (owner).
  - **Profile:**
    1. About 1,100 px of content: three section headings, two status badges, and four paragraphs (encryption, time zone, privacy, sign-out warning).
    2. A full-width red "Sign out of Ayoreni" button is the most prominent thing on the page.
    3. The time zone shows a technical id ("Asia/Calcutta").
  - **Every screen:**
    1. At the largest text size, tab labels clip ("Reco…", "Medi…", "Healt…"), and buttons stack unevenly.
    2. The title bar is a small centred label, and the page's own big heading scrolls away under it.
    3. Health Plan is still in the tab bar.
- **Task 3 mockups (Opus, 6 November 2026):** https://claude.ai/artifact/S8f1FKrbV875uhqYMSRS8G (private to the owner). Seven linked phone screens with synthetic data:
  - Today with one dose due, Today all done while a prescription is read, and Today the first time.
  - The Add sheet, Records, Medicines and Profile.
  - Waiting for task 4, the owner's approval.
- **Owner feedback on the mockups (6 November 2026):**
  - Approved: Today (one due, uploading, first time), the Add sheet, Records, Medicines and Profile.
  - Fixed: the Add button clipped the greeting, so it moved onto the date line.
  - The owner asked to design the whole app before building, if that costs less than building, so they can review it with their co-founder.
- **Scope change, task 3 (owner decision, 6 November 2026).** Before any build:
  - Full mockups of every screen the current app has, about 27, so every tap leads somewhere.
  - A component sheet.
  - A low-fidelity map of all 87 roadmap screens.

  Screens for later phases get full designs at the start of their own phase. Estimate: 8–12% of a weekly cap for the screens and 3–5% for the map, against about 20–30% to build the same screens. Opus sets the template and the component sheet; Haiku drafts the screens and the map; Opus reviews.
- **Full current-app mockups and app map (6 November 2026):** published at the same link.
  - The "Current app" page has 26 linked phone screens and a component sheet:
    - Today: one due, uploading, first time, offline.
    - Adding: the Add sheet, Add by hand, Check prescription, Upload a report.
    - Records: list, detail, Filter sheet, Reports and sharing.
    - Medicines: active, prescribed, detail, This week.
    - Profile: main page, Personal details, Reminders, Privacy and security, Terms and policies, Help.
    - Sign-in and system: Sign in, Create account, Update required, Maintenance.
  - The "App map" page shows all 87 roadmap screens by area and phase, with the 20 that exist today linked.
  - Drafted by Haiku from the Opus template; Opus checked structure and links, and fixed one wording.
  - Not checked by eye at 390 × 844 yet; the owner reviews on the phone.
  - Clinical choices shown: scanned medicines start as "Taking it now: off" (historical by default), uncertain values are flagged rather than guessed, the name as written is kept apart from the generic name, history uses "Not confirmed" and no percentages, and the lock screen never shows the medicine name.
- **Design canvas rebuilt on the Phase 3 style sheet (owner request, 7–8 November 2026).** Same link.
  - Four pages:
    - **Blueprint:** 44 editable parts plus a style-sheet board.
      - Two Opus agents built the parts from `design/components/**`, checked against gallery screenshots in `docs/phases/04b-screens/stylesheet/`.
      - They use the real Ionicons font, uploaded as a canvas asset.
    - **Live app v1:** 22 screens recreated from the before screenshots and the code by three Sonnet agents.
    - **App v2:** the 26 approved redesign screens, rewired onto the parts by Haiku.
    - **App map v2.**
  - Editing a part updates every screen that mounts it.
  - Parts that contain other parts (list row, empty and error states) copy the button and pill styles, so edits to those don't propagate.
  - Opus checks: no missing parts, no dead links, no "Missed" wording.
  - Not rendered by eye before publishing; the owner reviews.
- **Final design (lead session, 19 November 2026):** the owner marked the work-account canvas final. It holds S01–S07 ("Calm Glass UI v4"), each with a light and dark states board; S07 is still "In review". The lead session rewrote the build tasks (5–26), the screen and state table and the data gaps above from it, and raised D1–D7 for the owner.
