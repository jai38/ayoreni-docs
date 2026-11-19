# Phase 4b Brief: Simple, modern screens

**Risk:** Medium. **Roadmap:** `docs/project/ROADMAP.md`, section 5 (design direction) and section 6 (Phase 4b).
**Goal:** a new user opens Ayoreni and, without help, sees what to take now and how to add a medicine. Every tab fits the phone cleanly: nothing overlaps or clips, there's little text, and it feels like a current app, not a 2010 one.

**Status:** Approved by the owner on 6 November 2026, with Health Plan hidden until Phase 18. In progress.

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

## Decisions (proposed, owner to confirm)

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
5. Screens use only `design/` tokens and components. A new shared piece goes into `design/components/` with a gallery entry. Every string goes through `t()`.
6. The owner's open wording decisions (legal text "MyHealth", the store link, the "Mumbai/AES-256/S3" claims, the privacy badge) are not decided here. Moving that text one level down doesn't change its wording.

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

| # | Task | Who | Done when |
|---|---|---|---|
| 1 | Confirm the decisions above, including whether Health Plan is hidden | **Owner** | Written yes, or changes |
| 2 | A dev-only **screen preview** route that renders each tab from synthetic fixtures with no backend. It covers new user, nothing due, due now, all done, many doses, offline, uploading and error. It's excluded from release builds like the gallery. Add a screenshot script that captures each state at 360 × 800, 430 × 932 and the largest text size from the web export. | **Sonnet** | Screenshots of every current state exist in `docs/phases/04b-screens/before/` |
| 3 | **Audit and mockups.** List every overlap, clip and overflow from the task 2 screenshots. Then design the new Today, Records, Medicines, Profile, the Add sheet and the first-run Today. They're delivered as a clickable phone-size page the owner can open on the phone, with each screen's states (see the table below). | **Opus** | Audit list in Results; mockup link shared |
| 4 | Approve the mockups on the phone, or mark changes. Repeat until approved. | **Owner** | Written approval of each screen |
| 5 | **Render tests.** Set up React Native Testing Library with Vitest (finding 18). Add a check that fails if two notices render at once, or if a button label is wider than its button at the largest text size. | **Sonnet** | Setup lands with one passing render test per tab |
| 6 | **Shared pieces** in `design/`: the large title that shrinks on scroll, the single notice slot, the Add sheet and a Filter sheet, each with a gallery entry | **Sonnet** | Gallery shows each in every state; render tests pass |
| 7 | **Today** rebuilt to the approved mockup. It keeps `todayModel.ts` and `doseStatusDisplay` unchanged. | **Sonnet** | Clinical tests unchanged and passing; normal states fit one screen in the preview screenshots |
| 8 | **Medicines** rebuilt to the approved mockup. Its dose actions keep the `canActOnDose` and future-dose gates. | **Sonnet** wiring, **Haiku** layout | Dose-action tests unchanged and passing; stats on one line |
| 9 | **Records** rebuilt as one list with search and a Filter button; Reports opens as its own page | **Haiku** | Old `/timeline` and `/reports` links still resolve; render test passes |
| 10 | **Profile** rebuilt as a name card plus grouped rows. Privacy, legal and help text move to sub-pages unchanged. | **Haiku** | No paragraph on the tab; button labels fit at the largest text size |
| 11 | **First run:** new-user Today with the two Add choices and three tips | **Haiku** | Preview screenshot matches the mockup |
| 12 | **Copy pass:** shorten every label and heading on the four tabs to the text rules; strings only | **Haiku** | `en.ts` diff only; no label over three words on a button |
| 13 | After screenshots of every state in `docs/phases/04b-screens/after/`, next to the before set | **Sonnet** | Every state in the table below has a screenshot |
| 14 | Phone check (see Tests) | **Owner** | Written approval |
| 15 | Review against `TEMPLATE-review.md` | **Opus** | `04b-review.md` written |

Order: task 1, then 2, then 3 and 4. Tasks 5 and 6 can start once 2 lands. Screens 7 to 11 start only after 4 approves them, and each needs 6. Task 12 runs last, before 13.

## Screens and required states

| Screen | Loading | Empty | Error | No connection | Success |
|---|---|---|---|---|---|
| Today | Skeleton of the next-dose card | New user: two Add choices and three tips. Nothing due: one calm line plus Add | Card-level error with Retry in the notice slot | Last loaded data, offline banner and stamp | Next dose, short list of the rest, all done as one line; uploading shows in the notice slot only |
| Add sheet | n/a | n/a | n/a | Scan disabled with a reason; Add by hand still works | Scan a prescription, Add by hand |
| Records | Skeleton rows | First time: what Records holds, plus one Add. Filtered: "No results" and Clear filters | Inline error with Retry | Cached list, offline stamp | Search, Filter button, Reports row, one list |
| Medicines | Skeleton list | No medicines: one line plus Add | Inline error with Retry | Cached list, actions explained | Active list first; counts on one line; schedule and weekly view one tap down |
| Profile | Skeleton name card | n/a | Inline error with Retry | Cached, sign-out still works | Name card, grouped rows, sign out at the bottom |

## Tests

- **Gates:** `npm run typecheck` and `npx vitest run` in `myhealthapp`, with counts in Results.
- **New tests:**
  - A render test per tab, covering each state in the table.
  - Only one notice renders at a time, so upload progress and "all done" never overlap.
  - Button labels fit at the largest text size.
  - The Add sheet opens from Today and from Medicines.
  - Old routes `/timeline`, `/reports` and `/medications` still resolve.
- **Clinical tests that must still pass unchanged:**
  - An unrecorded past dose renders "Not confirmed".
  - Notification taps never mark Taken.
  - Paused, stopped and future doses show no Taken or Skip.
  - Adherence counts recorded doses only.
- **Owner phone check (new APK):**
  1. Sign in with a fresh test account. Can you add a medicine without help?
  2. Today with one dose due, then after tapping Taken: does it fit one screen?
  3. Upload a prescription. The progress shows in one place, with nothing overlapping.
  4. Records: find a document with search, then with Filter.
  5. Medicines and Profile: no clipped or overflowing text.
  6. Set the largest system text size and repeat steps 2 to 5.

## Review

Opus reviews the diffs, the before and after screenshots, and the gate results once at the end, using `TEMPLATE-review.md`.

## Cost

About the size of Phase 3: an estimated 15–25% of a weekly cap, mostly Opus mockups and Sonnet layout work. Record the plan-limit reading in `usage-log.md` at the end.

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
