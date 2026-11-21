# Phase 4b Brief: Simple, modern screens

**Risk:** Medium. **Roadmap:** `docs/project/ROADMAP.md`, section 5 (design direction) and section 6 (Phase 4b).
**Goal:** a new user opens Ayoreni and, without help, sees what to take now and how to add a medicine. Every tab fits the phone cleanly: nothing overlaps or clips, there's little text, and it feels like a current app, not a 2010 one.

**Status:** Approved by the owner on 6 November 2026. Tasks 1–4 are done. The design is final (19 November 2026) and the build is re-based on it: foundations first, then S06 Today and S07 Dose update. The build tasks below (5–19) replace the old tasks 5–15.

**Design sources, in order of precedence:**
1. The designer's handoff on the branch [`designes/initial-upload`](https://github.com/jai38/myhealthapp/tree/designes/initial-upload) in `jai38/myhealthapp` (designs only; never merged to `main`):
   - [`designes/DEV-HANDOFF.md`](https://github.com/jai38/myhealthapp/blob/designes/initial-upload/designes/DEV-HANDOFF.md): status, frozen system, product rules, gaps against `main`.
   - [`designes/design/screens.md`](https://github.com/jai38/myhealthapp/blob/designes/initial-upload/designes/design/screens.md): per-screen content, behaviour and decisions. Read it before building a screen.
   - [`designes/design/colour-shape-spec.md`](https://github.com/jai38/myhealthapp/blob/designes/initial-upload/designes/design/colour-shape-spec.md): Calm Glass UI v4, frozen 16 November 2026.
   - `designes/claude-design/*.dc.html`: the canvas source, for exact CSS values. Logo: `designes/assets/logo/` (never redraw or recolour).
2. The canvas, "AYORENI · Calm Glass UI v4": https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9, for the look of each state in light and dark.

Where the two differ, the handoff wins. The earlier mockups (`S8f1FKrbV875uhqYMSRS8G` and the 26-screen "App v2" set) are superseded.

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

## Decisions (approved 6 November 2026)

The build decisions below change decisions 1 (the brand), 3 (Today fits one screen), 4 (the Add sheet waits for S05 in Phase 6) and 5 (Medications is the tab name).

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

## Build decisions (owner, 19 November 2026)

| # | Question | Decision |
|---|---|---|
| D1 | Sign-in method | **Mobile number plus a one-time code by WhatsApp only** (owner, 19 November 2026, revised the same day; see `STATUS.md` and `docs/ops/whatsapp-otp-setup.md`). No SMS and no DLT for now. S01–S03 are built in Phase 5, which waits on Meta business verification and the authentication template. The designer adds a "no WhatsApp on this number" state to S01/S02 |
| D2 | The new visual system | **Yes.** Calm Glass UI v4: the spec's tokens in light and dark, Plus Jakarta Sans. Noto Sans Devanagari stays for Hindi (Plus Jakarta Sans has none). Roadmap §5 updated |
| D3 | Search and the notification bell in the header | **Hidden** until their phases. The header keeps the logo lockup |
| D4 | Records, Medications and Profile aren't designed | The designer designs them **after Medication Details**. Until then the UI builds the foundations and S06/S07 only; the three tabs keep their current layouts and pick up the new tokens |
| D5 | S07 was "In review" | **Locked by the owner.** Missed comes only from the user's "No, I missed it"; an untouched past dose shows Not confirmed. The designer marks it locked in `screens.md` and the canvas |
| D6 | "Today fits one screen" (decision 3) | **Dropped** for Today, per the spec ("content scrolls, never squeezed"). Nothing overlaps or clips, and the single notice slot stays |
| D7 | The hero ring and percentage | Keep the ring and "N of M taken", counted from recorded Taken doses only. **No percentage label** |
| D8 | Where S01–S05 go | **Phases 5 (S01–S03) and 6 (S04, S05).** S04's allergies need backend storage (Phase 6) and are **not** used for safety checks until Phase 16 |

**Not decided by the owner yet, so the brief takes the safe default (owner to confirm):** S06's Upcoming Appointment card and the Add Appointment and Add Reading tiles need data that doesn't exist (appointments, Phase 18; readings, Phase 11). Under hard rule 8 they are hidden, and Quick Actions shows Upload Record and Add Medication. The Supplement tag shows only once a medicine can carry that type.

What the build must keep from the design: "Not confirmed" with "Tap to update" for an unrecorded past dose; Missed only from the user; Mark as Taken logs the current time only on an explicit tap, with Undo for about 5 seconds; "other time" can't be in the future; "This only updates your log. It doesn't change your schedule or reminders."; status chips always pair an icon with a word.

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
5. Screens use only `design/` tokens and components (v4, from tasks 7 and 8). A new shared piece goes into `design/components/` with a gallery entry. Every string goes through `t()`.
6. The owner's open wording decisions (legal text "MyHealth", the store link, the "Mumbai/AES-256/S3" claims, the privacy badge) are not decided here. Moving that text one level down doesn't change its wording.
7. Fixtures and screenshots use the `CLAUDE.md` synthetic set, not the names drawn in the canvas.
8. A feature with no data behind it is not built half-way: no tile, button or empty card for it.
9. Designs come from the `designes/initial-upload` branch; never merge that branch or its `designes/` folder into app `main`.

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

## Contrast failures for the designer to fix in the spec

The spec requires text at 4.5:1 or more, secondary included. Measured on 19 November 2026, with glass and tinted colours blended over the screen background. Dark mode passes everywhere; these light-mode pairs don't. The designer picks passing shades and updates `colour-shape-spec.md`; the UI uses the fixed values.

| Pair (light mode) | Where it's used | Ratio |
|---|---|---|
| text-tertiary `#8A97A8` on bg `#F7FAFC` | Counts, metadata, placeholders | 2.83:1 |
| text-tertiary `#8A97A8` on white or glass card | Group counts, helper text, "Tap to update"-style lines | 2.94–2.97:1 |
| text-tertiary `#8A97A8` on the tab bar (white 80%) | Inactive tab labels | 2.94:1 |
| action-blue `#0071FF` on blue-tint `#E4F0FC` | Active tab label | 3.78:1 |
| action-blue `#0071FF` on bg `#F7FAFC` / white | Links ("Change number", "View Details") | 4.17 / 4.37:1 |
| white on primary-gradient top `#0080FC` | Primary button text, Due now chip | 3.82:1 (5.15:1 at `#0060FC`) |
| teal-text `#028265` on teal-tint `#E6F8F3` | Supplement tag | 4.35:1 |
| Taken `#0B7F3F` on `#E3F5EA` | Taken chip | 4.49:1 |
| white on the hero's teal end `#00C69C` | Hero text, if it reaches the teal side | 2.20:1 (the spec keeps text on the blue side; the UI must too) |
| disabled button text `#8A97A8` on `#E3EAF2` | Disabled Continue, Verify, Save | 2.45:1 (WCAG exempts disabled controls, but the spec's rule doesn't; designer to say) |

## Tasks

Tasks 1–4 are done (see Results). **Who** is UI (the UI session, `myhealthapp`), Backend (the backend session, `myhealthapp-backend`), Designer, or Owner. The model column follows `CLAUDE.md`.

| # | Task | Who | Model | Done when |
|---|---|---|---|---|
| 5 | D1 is decided (WhatsApp OTP). Start Meta business verification and the authentication template, per `docs/ops/whatsapp-otp-setup.md`; this gates Phase 5, not 4b | **Owner** | n/a | The checklist's status table filled in |
| 6 | Fix the contrast failures above in the spec; mark S07 locked in `screens.md` and the canvas; add the "no WhatsApp on this number" state to S01/S02 (for Phase 5); then design Medication Details, then Records, Medications and Profile | **Designer** | n/a | Spec updated on `designes/initial-upload`; S07 shows Locked |
| 7 | **v4 tokens** in `design/tokens.ts`: every colour token in light and dark, the dose chip colours, hero gradient, mesh background, radii, spacing, elevation and motion from the spec, with a theme that follows the system setting. Use the designer's fixed contrast values (task 6); until they land, keep the failing pairs behind a single token each so the swap is one line | **UI** | Sonnet | Gallery shows every token in both themes; the Phase 3 contrast and touch-size checks pass |
| 8 | **Plus Jakarta Sans** in `design/fonts.ts` (400, 600, 700) replacing Inter, with the spec's type scale, tracking and tabular figures for times and doses; Noto Sans Devanagari kept for Hindi; larger system text works without truncation | **UI** | Sonnet | Gallery type sheet matches the spec; no clipping at the largest text size |
| 9 | **Four tabs**: Today, Records, Medications, Profile, in the floating frosted tab bar (inset 16, bottom 30, height 68). Remove `health-plan`, `reports` and `timeline` as tabs in `app/(tabs)/`, keeping their screens reachable: Reports and Timeline from Records, Health Plan's route kept but unlinked until Phase 18. Old links `/timeline`, `/reports` and `/medications` still resolve | **UI** | Sonnet | Four tabs render; every old route resolves; tab labels fit at the largest text size |
| 10 | **Render tests** with React Native Testing Library and Vitest (Phase 4 finding 18), with checks that fail if two notices render at once or a button label is wider than its button at the largest text size. **Mostly done** as old task 5 in draft PR jai38/myhealthapp#3; still open: the label-fit check (needs the screenshot measurements, task 18) and flipping the `allDone-uploading` known failure when S06 lands | **UI** | Sonnet | Setup lands with one passing render test per tab |
| 11 | **Shared pieces** in `design/components/`, each with a gallery entry in both themes: glass card, hero card with ring, header (logo lockup; search and bell hidden), section header with count, medicine row (icon chip, name, dose · time, Supplement tag, status chip with taken time, ⋮), status chip (Taken, Due now, Upcoming, Not confirmed, Missed), Mark as Taken button, Quick Action tile, bottom sheet, radio option row, segmented control, time wheel, toast with Undo, single notice slot | **UI** | Sonnet | Gallery shows each in every state; render tests pass |
| 12 | **Dev preview fixtures** for every S06 and S07 state below, in both themes, with synthetic data only: the canvas's people and medicines become the `CLAUDE.md` set ("Asha Test", "Testmed A 500 mg") | **UI** | Haiku | `/dev/preview` renders every state |
| 13 | **S06 Today** to `screens.md`, on `todayModel.ts` and `doseStatusDisplay` unchanged: greeting by time of day with first name; hero ("Today's medications", N of M taken, Next, "N needs confirming", ring, no %); Morning, Afternoon and Evening groups, empty groups hidden, 4–5 rows then "Show all"; one Mark as Taken under the Due now row, logging the current time, toast with Undo for about 5 s, hero updating; all done shows "All done for today ✓" with a full ring; Quick Actions (Upload Record, Add Medication). A row tap opens the current medicine detail until Medication Details is designed | **UI** | Sonnet | Clinical tests unchanged and passing; every S06 state matches in both themes |
| 14 | **S07 Dose update sheet** to `screens.md`: Cancel and Save (disabled until a choice); medicine card; Yes, I took it (as scheduled, just now, other time on a wheel that can't go into the future); No, I skipped it (optional reason: Felt unwell, Doctor advised, Ran out, Side effects, Other); No, I missed it; the log-only footnote. Opens from a Not confirmed or due dose only; paused, stopped and future doses never open it (`canActOnDose`). Needs task 16 | **UI** | Sonnet | Every S07 state matches; dose-action and clinical tests pass |
| 15 | Raise the data gaps below: one line each in `ui-needs.md` and a `from-ui` issue in `myhealthapp-backend` linking the canvas board | **UI** | Haiku | Issues open and linked |
| 16 | **Dose actions for S06 and S07** (data gaps 2–5 below): accept `MISSED` on the POST and PATCH action routes; reject an `actionTime` in the future; a structured, optional skip reason; `fullName` on `GET /auth/me`. Update `api.md` in the same PR. Regression tests: nothing marks Taken or Missed without an explicit call; PATCH to `PENDING` (Undo) never leaves Missed; a future `actionTime` is refused | **Backend** | Sonnet | PR merged after owner OK; `api.md` updated in the same PR |
| 17 | **Copy pass**: every new string through `t()`, using the copy in `screens.md` | **UI** | Haiku | `en.ts` diff only; no button label over three words |
| 18 | After screenshots of every S06 and S07 state in `docs/phases/04b-screens/after/`, light and dark, at 360 × 800, 402 × 874 and the largest text size, plus each tab with the new tokens | **UI** | Sonnet | Every state in the table below has a screenshot |
| 19 | Deploy task 16 (`deploy-aws.yml`), dev then production; phone check (see Tests), then approve | **Owner** | n/a | Written approval |
| 20 | Review against `TEMPLATE-review.md` | Lead | Opus | `04b-review.md` written |

**Order:** 7, 8, 10 and 12 can start at once; 9 and 11 need 7 and 8. Then 13 (it needs 16 only for the greeting's first name, and falls back to no name). Task 14 needs 11 and the backend's task 16; 15 goes out as soon as 13 starts, so 16 can run alongside. 17 runs before 18. Merge order: backend, owner deploy, then the app.

**Moved out of 4b:**
- **S01 Mobile number, S02 OTP, S03 Create your account:** Phase 5, with the code sent by WhatsApp (D1).
- **S04 Your health profile:** Phase 6 (height, weight, blood group, allergy status and the allergy list, with backend storage). The allergy list is a record only; it drives no warning until Phase 16.
- **S05 Welcome, first-time Today:** Phase 6, with the profile card it depends on. Until then the existing first-run Today stays, on the new tokens.
- **Records, Medications, Profile redesign:** after the designer's task 6.

## Screens and required states (this phase)

| Screen | Drawn states (light and dark) | Required, not drawn |
|---|---|---|
| **Foundations** | Cover board: tokens, type, shape | Every tab on the new tokens and four-tab bar; largest text size |
| **S06 Today, returning** | Default (one Due now, one Not confirmed); after Mark as Taken (toast with Undo, count updated) | All done ("All done for today ✓"); nothing due today; many doses ("Show all"); loading (skeleton hero and rows); error with Retry in the notice slot; offline (cached, stamp, Mark as Taken disabled with a reason); uploading in the notice slot |
| **S07 Dose update** | Ask (Save disabled); Taken as scheduled; Taken at another time (wheel); Skipped with reason | Missed selected; saving; save failed with Retry; offline |

## Data the 4b screens need that `api.md` doesn't have

Checked against `api.md` as the backend filled it in on 19 November 2026 (backend `main` `b9a99e4`). Already covered: `DoseLog` (scheduled time, status, `actionTime`), `PENDING` in the past shown as Not confirmed, Taken at a chosen time (`actionTime` on POST), and Undo (PATCH back to `PENDING`). The name, strength and form come from the medicine, joined in the app.

| # | Screen | Need | When |
|---|---|---|---|
| 1 | S06 | Quantity per dose ("1 tablet") isn't stored anywhere. Show strength and form only ("5 mg · Tablet · 1:00 PM") until Phase 12 | Phase 12; nothing in 4b |
| 2 | S07, S06 | **Missed** as an action: `DoseLog.status` has `MISSED`, but neither action route accepts it | 4b, task 16 |
| 3 | S07 | "Other time" can't be in the future: the POST route takes any `actionTime` | 4b, task 16 |
| 4 | S07 | Skip reason (felt unwell, doctor advised, ran out, side effects, other): today only free-text `notes` exists. A structured optional field keeps it out of free text | 4b, task 16 |
| 5 | S06 | First name for the greeting: `GET /auth/me` returns `userId`, `email` and `patientProfileId` only (signup does take `fullName`) | 4b, task 16 |

Later phases, for their briefs: medicine or supplement type (Phase 14); appointments (Phase 18); readings (Phase 11); notifications and search (hidden); WhatsApp OTP with resend, attempt and cool-down limits and a "no WhatsApp" result (Phase 5); account profile with date of birth, gender and self-description, optional email and a consent record (Phases 5–6); height in cm, weight, blood group, allergy status, and allergies with the allergen as typed apart from the matched name, reaction, severity, a Medicine/Food/Environment tag and kept history on delete (Phase 6); profile completeness (Phase 6).

## Tests

- **Gates:** `npm run typecheck` and `npx vitest run` in `myhealthapp` (and in `myhealthapp-backend` for task 16), with counts in Results.
- **New tests:**
  - A render test per tab, and for each S06 and S07 state, in both themes.
  - Only one notice renders at a time; the Undo toast never stacks with an upload notice.
  - Button and tab labels fit at the largest text size.
  - Four tabs only; `/timeline`, `/reports` and `/medications` still resolve.
  - Search and the bell don't render; nor do the appointment card and the Appointment and Reading tiles.
  - S06: Mark as Taken records only on tap, Undo returns the dose to Due now, and the hero count counts recorded Taken doses only.
  - S07: Save stays disabled until a choice; "other time" can't be in the future; paused, stopped and future doses can't open it.
- **Clinical tests that must still pass unchanged:**
  - An unrecorded past dose renders "Not confirmed".
  - Notification taps never mark Taken.
  - Paused, stopped and future doses show no Taken or Skip.
  - Adherence counts recorded doses only.
  - Missed appears only after the user chose "No, I missed it".
- **Owner phone check (new APK), light and dark:**
  1. Four tabs; Reports and Timeline open from Records.
  2. Today with one dose due: tap Mark as Taken, then Undo. Does it return to Due now?
  3. Tap a Not confirmed dose: record it taken at another time, then skipped with a reason, then missed.
  4. Upload a prescription. Progress shows in one place, with nothing overlapping.
  5. Set the largest system text size and repeat steps 1 to 4.

## Review

Opus reviews the diffs, the before and after screenshots, and the gate results once at the end, using `TEMPLATE-review.md`.

## Cost

Smaller than first planned now that S01–S05 and the three tab redesigns moved out: an estimated 10–15% of a weekly cap, mostly Sonnet on tokens, components and S06/S07. Record the plan-limit reading in `usage-log.md` at the end.

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
- **Final design (lead session, 19 November 2026):** the owner marked the canvas final, and the designer pushed the handoff to `designes/initial-upload`. The lead re-based the build on the handoff (tasks 5–20) with the owner's decisions D2–D8; D1 was decided as WhatsApp OTP the same day. S01–S05 move to Phases 5–6.
- **UI tasks 7–13 and 15 (UI session, 19–21 November 2026),** in draft PR [jai38/myhealthapp#3](https://github.com/jai38/myhealthapp/pull/3) (not merged):
  - **7:** v4 tokens in light and dark (`themes`, `useTheme()`). The failing light pairs each sit behind one interim token until the designer's fix lands.
  - **8:** Plus Jakarta Sans and the spec's type scale.
  - **9:** four tabs in the floating bar; old routes resolve; Health Plan unlinked.
  - **10:** the render tests, plus the `--check-labels` label-fit check in `screens.mjs`.
  - **11:** the Glass parts in the gallery, in both themes.
  - **12:** the S06 preview states (S07's come with task 14).
  - **13:** S06 Today.
  - **15:** issues #5–#8.
  - **Gates:** typecheck passes; 709 tests in 56 files (573 in 50 before task 7).
  - **Open choices for the lead:**
    - The ring counts Taken only, so all done with a skip isn't a full ring.
    - Until S07, the row ⋮ offers Taken and Skip.
    - Blur is on web only.
    - The first-run card's button clips at the largest text size (S05, Phase 6).
