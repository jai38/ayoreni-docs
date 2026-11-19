# Phase 3 Brief: Design system

**Risk:** Low. **Roadmap:** `docs/project/ROADMAP.md`, sections 5, 6 and 8.
**Goal:** every shared base component exists in its finished Ayoreni style, with each state on show in a developer-only gallery, and the owner approves the look on a real phone.

**Status:** Complete. The owner closed Phase 3 on 5 November 2026 after checking it on the phone: refresh `6cf8312`, logo and icons `0ee4a84` on `phase-3/design-system`, not yet merged to `main`. Phase 2 stays open on the budget alert email, which the owner is closing out.

## Decisions

- **New system beside the old one.** The new tokens and components live in `myhealthapp/design/`. The existing `theme/index.ts` and `components/` stay untouched, because Phase 4 re-skins and replaces the current screens. No existing screen changes in this phase.
- **Blueprint colors are the base.** White `#FFFFFF`, Clear Blue `#2878C7`, Teal `#087E83`, Deep Navy `#16324F`, Slate `#526372` and Cool Grey `#F2F5F7`. Status colors are a separate set.
- **Contrast finding.** On white, Clear Blue measures 4.56:1 and Teal 4.86:1, so both pass for text. On Cool Grey they measure 4.17:1 and 4.44:1, which fails for body text. The rule is that blue or teal text sits only on white, and on Cool Grey only at large size or as a non-text UI color. The contrast test enforces this.
- **Semantic tokens only in components.** Components use names like `color.text.primary`, never a raw hex value. That lets a dark set be added later without changing components. Dark mode is not shipped (roadmap section 3).
- **Font.** Inter, loaded through `@expo-google-fonts/inter` and `expo-font`. Noto Sans Devanagari waits for Phase 26. Text scales with system settings, capped at a maximum multiplier that the gallery shows.
- **Icons.** One `Icon` wrapper over the bundled `@expo/vector-icons`. Medicine-form icons are neutral shapes, never pill pictures.
- **Strings.** A small typed `t('key')` helper over `design/strings/en.ts`, so a missing key fails typecheck. No i18n library until Phase 26 needs plurals and Hindi.
- **New packages** go in only through `npx expo install`, so they match SDK 52. The Expo upgrade stays in Phase 4.
- **Retired components** (`AdherenceDonut`, `AiOmnibar`, `HorizontalDateStrip`) are not ported.

## Hard rules

1. No model opens `.env` or receives a key, cloud credential, production database access or real patient data. Gallery sample data is synthetic: invented names and generic medicine names only.
2. Work goes on `phase-3/design-system` in `myhealthapp` only. The backend is not touched. No commits, pushes or branch switches unless the owner asks.
3. Pushing to mobile `main` publishes preview APKs, so nothing merges until the owner approves the phase.
4. Any task that fails twice goes to the next model up, or to the owner.
5. The gallery (Y4) must not be reachable in a release build. A test proves it.
6. Status display helpers follow `CLAUDE.md` rules 14 and 15: no display path shows Taken or Missed unless the state passed in says so explicitly, and no dose state ever uses the red `danger` tone. An unresolved past dose is labelled "Not confirmed" in a calm neutral color.
7. One component group per Haiku session, under about 200 changed lines, and the session is given the exact token and component names from task 2.

## State found on 2 November 2026

- `myhealthapp`: `main` at `91126bc`, clean. The current theme in `theme/index.ts` is a slate and sky-blue palette that predates the blueprint. Components are in `components/`, and the screens are large (`app/(tabs)/index.tsx` has 2,102 lines).
- Vitest is already set up. `scripts/screenshot.mjs` captures the Expo web build at 390×844, if Playwright is installed.
- `@expo/vector-icons` is already in `node_modules` through Expo. No custom font is loaded yet.
- `designs/` holds 9 reference images. `dashboard tab 5` is the base for Today (roadmap section 5).
- The logo vector has not been supplied yet. A placeholder leaf is used until it arrives.
- Phase 1 Haiku pilot: Haiku built Y2 on the first try for about 76K tokens. This phase tests Haiku on a harder screen, the gallery.

## Tasks

| # | Task | Who | Done when |
|---|---|---|---|
| 1 | Supply the logo vector (SVG), or confirm the placeholder for now | **Owner** | File in `designs/` or placeholder accepted |
| 2 | Token spec: semantic color, type scale, spacing, radius, elevation, motion durations, minimum touch size (48), exact export and component names for every task below | **Opus** | `design/tokens.ts` interface and a name list in this brief's appendix |
| 3 | Implement tokens, Inter font loading, text styles with font-scale cap | Haiku | Typecheck passes; font renders on web and Android |
| 4 | Contrast test: every declared text and background pair at 4.5:1 or above (3:1 for large text and UI parts); touch-size test on every interactive component | Haiku | Tests fail if a pair or size is broken |
| 5 | Status display helpers: dose status → label, tone and icon per roadmap section 5 table; trust label → text and icon | **Sonnet** | Table-driven tests cover every state, including Not confirmed, never mapped to Taken or Missed |
| 6 | String helper and `en.ts` with the keys the components need | Haiku | Missing key fails typecheck |
| 7 | Foundations: Text, Icon, Logo, Wave header, Spacer, Divider, Screen scaffold, Scroll with refresh | Haiku | In gallery with states |
| 8 | Actions: primary, secondary, text and danger buttons, Icon button, Passkey button (visual only), Link row, Undo message | Haiku | In gallery: default, pressed, disabled, loading |
| 9 | Inputs, part 1: text, email, password with rule checks, OTP boxes, search, multiline, number with unit | Haiku | In gallery: empty, filled, focused, error, disabled |
| 10 | Inputs, part 2: checkbox, radio, switch, chips, select sheet, date with partial-date option, time, severity scale | Haiku | Same states as task 9 |
| 11 | Containers: Card, Section header with View All, List row, Icon tile, Accordion, Segmented control, Bottom sheet, Dialog, Stepper | Haiku | In gallery |
| 12 | Navigation: Tab bar, Top bar, Patient pill, Notification bell with count, Patient banner, Filter bar with Clear all | Haiku | In gallery; not wired to routes yet (Phase 4) |
| 13 | Feedback: skeletons, Empty state, Error state, Offline banner, Sync indicator, Toast, Progress, Permission prompt | Haiku | In gallery; respects reduced motion |
| 14 | Trust: Status pill, Source label, Verified or draft label, Uncertain marker, Revision link, Disclaimer block, Freshness stamp | Haiku, using task 5 helpers | In gallery for every state |
| 15 | Gallery screen Y4 at a dev-only route, grouped by section, with a font-scale toggle | Haiku | Opens in dev; release-build test proves it is unreachable |
| 16 | Screenshots of every gallery section at default and largest font scale, compared against `designs/` | Haiku | PNGs in `screenshots/`, with a list of differences |
| 17 | Review the diff and gate results | **Opus** | Review notes recorded below |
| 18 | Try the gallery on a real Android phone at default and large text, and approve the look | **Owner** | Done 5 November 2026: the owner approved the refreshed look, the inputs and buttons, and the logo |

Who is one of: **Owner** (a human does it), **Opus** (decisions and hard reviews), **Sonnet** (multi-file logic, security, clinical invariants), **Haiku** (mechanical edits, templates, scripts, copy).

## Screens and required states

| Screen | Loading | Empty | Error | No connection | Success |
|---|---|---|---|---|---|
| Y4 Component gallery (dev only) | Font loading shows a system-font fallback, not a blank screen | Not applicable | A broken section shows Error state and the rest still renders | Works offline; no network needed | Every component shown in every state from tasks 7-14 |

## Tests

- Typecheck: `npm run typecheck` in `myhealthapp`, 0 errors.
- Unit: `npm test` in `myhealthapp`, with counts recorded in Results. Existing suites still pass.
- New regression tests:
  - Contrast for every declared color pair, and touch size of 48 or more for every interactive component.
  - Dose status helper: every state in roadmap section 5 maps to its label; an unresolved past dose is "Not confirmed" and never Taken or Missed; Missed appears only when the input state is user-declared missed.
  - The gallery route is unreachable when `__DEV__` is false.
  - Every string key used by a component exists in `en.ts`; this is enforced by typecheck.
- Manual owner check: install a dev build on an Android phone and open the gallery. Scroll every section at default text size, then at the largest system text size. Check that nothing clips, overlaps or falls below a comfortable tap size.

## Review

Opus reviews the diffs and gate results once at the end, using `TEMPLATE-review.md`. Usage goes in `usage-log.md` per model.

## Results

- Task 1: the owner chose the placeholder leaf logo for now, with the SVG expected by 3 November 2026. The `Logo` component takes the asset from one file, so swapping it in is a one-file change (2 November 2026).
- Task 2: token spec and names written in the appendix. Every declared contrast pair was computed beforehand and passes (2 November 2026).
- Task 3: branch `phase-3/design-system` created from `main` at `91126bc`. `expo-font ~13.0.4` and `@expo-google-fonts/inter ^0.4.2` were added through `npx expo install`; the `expo-font` config plugin was skipped because fonts load at runtime. Haiku wrote `design/tokens.ts`, `typography.ts`, `fonts.ts` and `index.ts`, plus 6 tests, on the first try in about 85K tokens. Opus review found it matches the appendix with no changes needed. Gates: typecheck 0 errors, tests 113 passing in 16 files. Not yet run on a device. Fonts are not loaded by any screen until the gallery (task 15). Uncommitted (5 November 2026).
- Task 4: Haiku wrote `design/contrast.ts` (WCAG ratio, 17 declared pairs, 2 forbidden pairs, `meetsTouchTarget`) and 28 tests, first try, about 73K tokens. All pairs pass; the lowest are `text.tertiary` on white at 4.53 and `text.link` on white at 4.56. The forbidden pairs measure 4.17 and 4.14. The touch-size helper is ready, but component tests use it only from task 7 on, because no components exist yet (5 November 2026).
- Task 5: Sonnet wrote `design/status.ts` and 24 tests, first try, about 77K tokens. The safety regressions, each a named test:
  - Not confirmed never becomes Taken or Missed.
  - Snoozed never becomes Taken.
  - No dose state uses `danger`.
  - Only an explicit `missedByUser` shows "Marked missed".
  - `formatTime` gets the state's own date.

  Opus changed one thing: the Uncertain trust icon is now `alert-circle-outline`, so it no longer matches the Not confirmed icon. A note for Phase 12: the engine must pass `notConfirmed` for an unrecorded past dose, which this helper cannot enforce (5 November 2026).
- Task 6: Haiku wrote `design/strings/en.ts` (47 keys) and the `t()` helper, plus 6 tests, first try, about 67K tokens. An unknown key fails typecheck, which an `@ts-expect-error` line proves. A missing param leaves its placeholder visible (5 November 2026).
- Gates after tasks 3-6: typecheck exit 0; tests 171 passing in 19 files. `design/index.ts` exports all six modules. Committed as `f6e47df` and pushed to `origin/phase-3/design-system` at the owner's request (5 November 2026). Branch pushes trigger no CI, because workflows run only for `main`.
- Task 7: Haiku built the Foundations group (Text, Icon, Logo, WaveHeader, Spacer, Divider, Screen, RefreshScroll) and `design/assets/LogoMark.tsx`, plus 11 tests, first try, about 99K tokens. `react-native-svg 15.8.0` was added through `npx expo install`. The placeholder leaf is drawn as SVG paths, because the existing app icon is a shield, not a leaf. Opus added `brand.name` to `en.ts` and switched Logo to `t()` (5 November 2026).
- Task 8: Haiku built the Actions group (Button, IconButton, PasskeyButton, LinkRow, UndoToast), plus 21 tests, in about 92K tokens. Its contrast test caught a spec error: link blue on the pressed blue tint is 4.0:1. Haiku stopped rather than weaken the test. Opus fixed the spec so pressed secondary and text buttons use the pressed blue (4.97:1). At integration, Opus switched Actions to the shared `Icon`, made `testID` optional everywhere, and exported both groups through `design/components/index.ts` and `design/index.ts` (5 November 2026).
- Gates after tasks 7-8: typecheck exit 0; tests 203 passing in 21 files. Components are not rendered anywhere yet, because they first appear in the gallery (task 15). Uncommitted.
- Opus added 44 string keys for inputs (`input.*`, `select.*`, `date.*`, `month.1-12`, `time.*`, `severity.1-5`) before tasks 9-10 began, so the two parallel builds never edited the catalog (5 November 2026).
- Task 9: Haiku built the text inputs (TextField, EmailField, PasswordField with live rule checks, OtpField, SearchField, MultilineField, NumberUnitField) and `inputStyle.ts`, plus 34 tests, first try, about 115K tokens.
  - Disabled fields use `text.secondary` for the placeholder, because tertiary on Cool Grey fails contrast.
  - `parseDecimal` rejects `.5`, `1,5` and `1e3` rather than guess.
  - Numbers are never converted between units (5 November 2026).
- Task 10: Haiku built the choice, date and time inputs (Checkbox, Radio and RadioGroup, Switch, ChipGroup, SelectSheet, DateField, TimeField, SeverityScale) and the pure modules `choiceStyle.ts`, `partialDate.ts` and `time.ts`, plus 16 tests, first try, about 122K tokens.
  - A partial date never gets a default day or month; a test proves `{ year: 2019 }` has no `month` key.
  - The severity scale uses the same neutral colors on all five steps, so color doesn't imply severity (5 November 2026).
- Opus integration for tasks 9-10:
  - Added `checkmark` to `IconName` and removed Haiku's type cast.
  - Made the native switch inside `Switch` non-interactive and hidden from screen readers, so a tap cannot toggle twice.
  - Hid the SelectSheet backdrop from screen readers, which reach the Close button instead.
  - Made every `testID` optional, with derived IDs guarded.
  - Added `Inputs/index.ts` to the exports.
  - Known limit: DateField and TimeField read their initial value once and do not resync if the parent replaces the value. This is fine for forms; revisit if a screen needs it.
- Gates after tasks 9-10: typecheck exit 0; tests 253 passing in 23 files. Not rendered yet. Uncommitted.
- Before tasks 11-12, Opus added the `container.*`, `nav.*`, `patient.*` and `filter.*` keys, plus nine icons: the filled tab icons, chevrons, ellipsis and funnel (5 November 2026).
- Task 11: Haiku built the Containers group (Card, SectionHeader, ListRow, IconTile, Accordion, SegmentedControl, BottomSheet, Dialog, Stepper), plus `containerStyle.ts`, `useReduceMotion.ts` and 10 tests, first try, about 115K tokens.
  - A pressable Card cannot compile without an accessibility label.
  - A Dialog cannot be dismissed by tapping outside it, so it must be answered.
  - Bottom sheet and dialog animations turn off with reduced motion.
  - Stepper states differ by shape, not just color (5 November 2026).
- Task 12: Haiku built the Navigation group (TabBar, TopBar, PatientPill, NotificationBell, PatientBanner, FilterBar), plus `navStyle.ts` and 11 tests, first try, about 100K tokens.
  - The active tab shows three signals: a filled icon, color and an indicator bar.
  - The patient banner cannot be dismissed.
  - No routing yet; Phase 4 wires it.
  - Opus added `filter.remove` and replaced Haiku's stand-in label (5 November 2026).
- Gates after tasks 11-12: typecheck exit 0; tests 274 passing in 25 files. No export-name clashes across the five groups. Not rendered yet. Uncommitted.
- Before tasks 13-14, Opus added 11 `feedback.*`, `permission.*` and `trust.*` keys and 5 icons (5 November 2026).
- Task 13: Haiku built the Feedback group (Skeleton and SkeletonGroup, EmptyState, ErrorState, OfflineBanner, SyncIndicator, Toast, ProgressBar, PermissionPrompt), plus `feedbackStyle.ts` and 10 tests, first try, about 125K tokens.
  - Error, offline and sync-error states use calm `caution` or `neutral` tones, never red; a test asserts this for sync.
  - Skeleton shimmer and indeterminate progress stop under reduced motion.
  - Opus changed `useReduceMotion` to remember the last value across mounts and to follow live setting changes, so only the app's first animation can briefly run before the setting is read (5 November 2026).
- Task 14: Haiku built the Trust group (StatusPill, SourceLabel, VerificationLabel, UncertainMarker, RevisionLink, Disclaimer, FreshnessStamp), plus `trustStyle.ts` and 28 tests, first try, about 98K tokens.
  - Every dose and trust pill is checked for contrast, soft and filled.
  - `isStale` takes `now` as an argument; a test proves it never reads the clock.
  - An unverified fact shows "AI draft".
  - Opus hid the inner pill and hint from screen readers in UncertainMarker, so the combined label and hint is read once (5 November 2026).
- Gates after tasks 13-14: typecheck exit 0; tests 312 passing in 27 files. All seven component groups are built and exported, with no raw hex in `design/components`. Committed as `00a3eed` and pushed to the branch at the owner's request (5 November 2026).
- Task 15, gallery Y4 (5 November 2026):
  - Opus wrote the shell: `design/gallery/galleryGuard.ts`, `Gallery.tsx`, `GallerySection.tsx` with a per-section error boundary, `app/dev/gallery.tsx`, and a guard test.
  - `app/_layout.tsx` gains a dev-only branch: when `__DEV__` is true and the path is `/dev/gallery`, the gallery opens without the sign-in gate, so it needs no backend and no test sign-in. In a release build, `isGalleryEnabled(false)` is false, the route redirects to `/`, and a test proves it.
  - Two Haiku sessions wrote the seven section files (about 137K and 130K tokens), first try, with synthetic data only.
- **Deviation from task 15.** The in-gallery font-scale toggle was dropped. Many components render RN `Text` directly, so an app-level toggle would not scale them and would give a false result. Large-text checking moves to the owner's phone with the system text-size setting (task 18).
- First render, 5 November 2026, Expo web at 375 px in the built-in browser. All seven sections render; Inter, the leaf placeholder and the wave load; no section tripped its error boundary. Opus fixed:
  - The screen wave was pinned and drawn over scrolling content; it now scrolls with the content.
  - A list row with a trailing pill squeezed its title to about 67 px and split "Paracetamol" mid-word. The title now keeps at least half the row, and pill labels wrap.
  - Long button labels could overflow; they now wrap.
  - The SVG logo leaked an `accessible` attribute to the DOM on web.
  - The web switch thumb turned teal when on.
- Gates after task 15: typecheck exit 0; tests 315 passing in 28 files. Uncommitted.
- Task 16, screenshots (5 November 2026). With the app window in front, Opus captured the gallery in the built-in browser at 375×812 (2× pixel density), at default text size:
  - 23 section screens plus the open bottom sheet and the delete dialog, saved as `myhealthapp/screenshots/phase-3/01-foundations.jpg` to `25-delete-dialog.jpg`. The folder is gitignored.
  - Playwright was not used. Large text size was not captured on web; that check is on the phone (task 18).
  - One more fix from the captures: the wave was clipped at the left gutter. The screen gutter now sits on the scroll content, so the wave runs edge to edge.
  - Gates after this: typecheck exit 0; tests 315 passing in 28 files.
  - Tasks 15-16 committed as `7922d3e` and pushed to `origin/phase-3/design-system` at the owner's request (5 November 2026). The working tree is clean, and `main` is untouched.
- Task 17, Opus review (5 November 2026), written up in `docs/phases/03-review.md`. **Verdict: pass with follow-ups.**
  - Fixed during the review:
    - Date and time boxes are now announced with their field label.
    - Gallery samples no longer mention a mobile number.
    - The gallery subtitle now says "Not reachable in release builds".
  - Gates after the fixes: typecheck exit 0; tests 315 passing in 28 files.
  - The fixes are uncommitted.
  - The only blocking item is task 18.
- Owner test on the phone, 5 November 2026: the gallery opened on Expo Go for SDK 52.
  - The earlier sign-in failures came from a stale bundle built while `myhealthapp/.env` pointed at a dead server. Once `.env` was emptied, the Android bundle carried only the AWS URL; Opus verified this by fetching the bundle from Metro.
  - Browser testing at `localhost:8081` cannot sign in, because the AWS API's CORS allows only `https://jai38.github.io`. This is by design.
  - The owner approved the system's structure and asked for an Apple Health-style refresh: modern, not government-like, with design freedom given to Opus.
- Design refresh (5 November 2026), per the section above.
  - **Opus:**
    - Changed the tokens: grey grouped screen, deeper link blue, tertiary `#627080`, teal `#06706F`, seven tile colors, `radius.control`, button height 50, flat cards.
    - Moved to an Apple-like type scale with a 17 pt body.
    - Updated the contrast pairs for the grey screen. The new forbidden pairs are fill blue as text on the screen, and tertiary on `bg.subtle`.
    - Switched medicine icons to MCI glyphs through a new `MedicineIcon`.
    - Added the filled tile icons.
    - Gave the severity selection a blue outline so it shows on Android.
    - Gave the card shadow Android elevation 1.
    - Removed the tab underline.
    - Turned the wave off in the gallery, because it read as a stripe on grey.
  - **Haiku, two parallel sessions, about 131K and 119K tokens:**
    - Tinted secondary buttons, borderless flat cards and the new `ListGroup` (iOS inset grouped list), Settings-style icon tiles, iOS segmented control, and a white tab bar.
    - Filled chips with a check, rounded checkboxes and rings, a single-track severity scale, and white fields on grey.
  - Gates: typecheck exit 0; tests 332 passing in 28 files.
  - The "after" screenshots are in `myhealthapp/screenshots/phase-3-refresh/`; the "before" set stays in `phase-3/`.
  - Committed and pushed as `6cf8312` on `phase-3/design-system`.
  - Still open: the wave is only suited to white screens such as sign-in; decide in Phase 4 whether to keep it there.
- Inputs and buttons polish (5 November 2026, owner asked after the phone check):
  - **Fields:**
    - The new shared `FieldBox` gives every field a 14 pt corner (the same as buttons) and a soft decorative halo while focused or in error. The 2 px border still carries the state at 3:1.
    - Padding compensates for the border width, so text never shifts on focus.
    - The leading icon turns blue on focus.
  - **Labels:** the new `FieldLabel` is a semibold slate caption, used by TextField, OtpField, DateField, TimeField and SelectSheet.
  - **Kept on purpose:** the 3:1 resting border. Borderless grey fields fail WCAG 1.4.11 on white cards.
  - **Buttons:**
    - 52 px tall, matching the fields.
    - A 0.97 press spring, skipped under reduce motion.
    - While loading, the label stays laid out but hidden under the spinner, so the width never jumps.
    - `fullWidth` stays opt-in; Phase 4 screens should set it on form actions.
  - **Web only:** the browser's default focus outline is removed, because the field draws its own.
  - Gates: typecheck exit 0; tests 338 passing in 28 files.
  - Committed and pushed as `6cf8312` on `phase-3/design-system`.
- Logo and app icons (5 November 2026):
  - **Concept:** the designer's infinity-plus-person concept, redrawn by Opus as a flat SVG.
    - An even-width ribbon with one crossing gap.
    - Gradient `#0A5CE6` → `#0AA5F0` → `#00C9A7`.
    - A head dot above the right loop.
  - **Files:**
    - Master: `design/assets/logo/ayoreni-mark.svg`.
    - `LogoMark.tsx` draws the same mark, with a `monoColor` option; this replaces the leaf placeholder.
  - **Icons:** `scripts/make-icons.py` (Pillow only) renders the PNGs from the same geometry:
    - `icon.png`: gradient, full bleed, white mark.
    - Android adaptive foreground, gradient background and monochrome layers.
    - `splash.png`: gradient mark on white; the splash background changed from `#111418` to white.
    - `notification-icon.png`: white silhouette; accent `#0A5CE6`, replacing the old orange.
  - **Owner calls:**
    - Run a trademark search before public use; infinity-person marks are common in health.
    - The app display name is still "My Health App".
  - Committed and pushed as `0ee4a84` on `phase-3/design-system`.
- Differences from `designs/`:
  - **Intended, per roadmap section 5:**
    - "Due now" is a calm blue filled pill, not red.
    - Buttons are flat blue, not gradient.
    - Medicines use neutral form icons, not colored pill pictures.
    - The "Tonight" purple pill is dropped.
    - Input borders are darker grey (`#7D8B97`), so they pass the 3:1 rule for UI parts.
  - **Logo:** the two-leaf placeholder differs from the design's three-petal mark. It changes when the owner's SVG arrives, as a one-file swap in `design/assets/LogoMark.tsx`.
  - **Wave:** paler and lower-contrast than the design's cyan wave, at 40% opacity. **Owner's call on the phone:** keep it subtle or make it stronger.
  - **Headings:** the design's greeting looks heavier than Inter Bold 700. Close enough for now; Inter 800 can be added if the owner wants it.
  - **Patient pill:** shows initials in a blue circle where the design has a person icon. Owner's call.
  - **Tab bar:** matches the design (five tabs, filled active icon, underline). In the gallery's 343 px box "Health Plan" truncates to "Health Pl…"; at the full 375 px phone width each tab gets 75 px. **Check on the phone.**
  - **Small follow-ups found, not yet fixed:**
    - The `injection` form icon is a dropper (`eyedrop-outline`), and `drops` and `liquid` share the same icon. Better icons are needed before Phase 14.
    - Gallery sample fields say "Mobile number", which contradicts the email-only sign-in decision. Gallery copy only; no component change needed.

## Design refresh: calm, Apple Health-style (owner request, 5 November 2026)

The owner approved the system and gave Opus design freedom: "not a government portal, more like Apple Health; modern, attractive, easy to understand". Only tokens and a few components change, so every screen inherits the result. The appendix tables below still list the original values; this section overrides them where they differ.

1. **Grouped layout.** The screen background is Cool Grey `#F2F5F7`. Content sits in white cards and grouped lists with no borders and no shadows. This overrides the earlier "white background" decision in roadmap section 5, at the owner's request.
2. **Color.**
   - Links and tinted text use `#1F69B0`: 5.67:1 on white and 5.18:1 on grey.
   - Tertiary text uses `#627080`: 4.62:1 on grey.
   - Teal accent uses `#06706F`.
   - Filled blue stays `#2878C7` with white text.
   - Blue `#2878C7` as **text** on grey (4.17:1) remains forbidden, and the test asserts it.
3. **Category tiles.** These are Apple Settings-style rounded squares with a white filled glyph: blue, teal, red `#D14343`, orange `#B85E0A`, purple `#6A4FD0`, green `#2E8B57`, plus neutral grey. Glyphs need 3:1 against the fill, and every pair passes.
4. **Type.** Larger, Apple-like sizes, with tighter letter spacing on big headings:
   - display 34/41;
   - title1 28/34;
   - title2 22/28;
   - title3 17/22;
   - **body 17/24**, larger for older readers;
   - callout 16/22;
   - caption 13/18.
5. **Buttons.**
   - Height 50, radius 14.
   - Secondary is a **tinted** button (blue-tint fill, blue text, no outline).
   - Text buttons use the link blue.
6. **Lists.** A new `ListGroup` holds rows in one white rounded card with inset hairline separators, iOS-style.
7. **Choices.**
   - Chips: unselected is a grey fill; selected is a solid blue fill with white text and a check.
   - Checkboxes are rounded squares filled blue when checked.
   - The severity scale becomes one segmented track, not five boxes.
8. **Tab bar.** A filled tinted icon and label mark the active tab; the underline is dropped. The active state still has two signals, icon shape and color.
9. **Medicine icons.** Clear monochrome glyphs from Material Community Icons, already bundled with `@expo/vector-icons`:
   - tablet: scored circle;
   - capsule: pill;
   - liquid: bottle;
   - injection: needle;
   - inhaler: spray;
   - drops: dropper;
   - topical: lotion;
   - other: medical bag.

   They are shapes, not colored pictures, so roadmap section 5's rule still holds.

## Appendix: token and component names

Written by task 2 on 2 November 2026. Every Haiku task uses these names exactly. A name change needs Opus approval and an edit here first.

### Files

```
myhealthapp/design/
  tokens.ts            colors, space, radius, elevation, motion, size, font tokens
  typography.ts        textStyles and the font-scale cap
  fonts.ts             useAppFonts() hook (Inter 400, 500, 600, 700)
  status.ts            task 5 helpers (Sonnet)
  strings/en.ts        string catalog
  strings/index.ts     t() helper
  contrast.ts          contrastRatio(), and the declared pairs list used by the test
  index.ts             re-exports everything public
  components/<Group>/<Name>.tsx, one component per file, with groups as below
  assets/LogoMark.tsx  the only file Logo reads: an SVG leaf mark drawn with react-native-svg
                       (placeholder until the owner's SVG arrives; the swap edits this file only)
app/dev/gallery.tsx    Y4, guarded by __DEV__
test/design/*.test.ts  tests for tasks 4, 5, 6 and 15
```

Import from `design` only through `design/index.ts`. Components never import `theme/index.ts`, and old code never imports `design/` in this phase.

### Color tokens

`color` is a nested `as const` object. The raw palette stays private inside `tokens.ts`; components use only the semantic names.

| Semantic name | Value | Use |
|---|---|---|
| `color.bg.screen` | `#FFFFFF` | Screen background |
| `color.bg.subtle` | `#F2F5F7` | Cool Grey: recessed areas, chips, skeleton base |
| `color.bg.card` | `#FFFFFF` | Cards |
| `color.bg.tintBlue` | `#E8F1FB` | Icon tiles, hero card fill, info pill |
| `color.bg.tintTeal` | `#E3F4F4` | Scan icon tile, source pill |
| `color.text.primary` | `#16324F` | Deep Navy: headings, body |
| `color.text.secondary` | `#526372` | Slate: supporting text, on white or Cool Grey |
| `color.text.tertiary` | `#6A7884` | Timestamps, placeholders. **White background only** (4.53:1) |
| `color.text.inverse` | `#FFFFFF` | On filled buttons |
| `color.text.link` | `#2878C7` | Clear Blue links. **White background only** |
| `color.action.primary` | `#2878C7` | Primary button fill, focus ring, active tab |
| `color.action.primaryPressed` | `#1F69B0` | Pressed state |
| `color.action.secondary` | `#087E83` | Teal accents. **White background only** for text |
| `color.action.danger` | `#B42318` | Danger button fill and text |
| `color.border.subtle` | `#E3E8ED` | Card and divider hairlines (decorative) |
| `color.border.input` | `#7D8B97` | Input outline (3.49:1, meets the 3:1 rule for UI parts) |
| `color.border.focus` | `#2878C7` | Focused input |
| `color.wave.from` | `#BFE3F7` | Header wave gradient start |
| `color.wave.to` | `#7FD3E6` | Header wave gradient end |

Status tones, used by `StatusPill` and other trust parts through `status.ts`. They are never chosen by hand in a screen.

| Tone | Text (`color.tone.<t>.fg`) | Fill (`color.tone.<t>.bg`) | Contrast |
|---|---|---|---|
| `info` | `#1F69B0` | `#E8F1FB` | 4.97 |
| `success` | `#1E7A45` | `#E7F5EC` | 4.76 |
| `caution` | `#8A5300` | `#FFF3DD` | 5.76 |
| `danger` | `#B42318` | `#FDECEA` | 5.75 |
| `neutral` | `#46525C` | `#EDF0F3` | 7.00 |
| `source` | `#066A6E` | `#E3F4F4` | 5.62 |

`danger` is for errors, destructive actions and allergy warnings only, never for a dose state.

### Declared contrast pairs (task 4 test input)

Normal text needs 4.5:1 or more, and large text and UI parts need 3:1 or more. The pairs are: each `text.primary` and `text.secondary` on `bg.screen` and `bg.subtle`; `text.tertiary` and `text.link` on `bg.screen`; `text.inverse` on `action.primary`, `action.primaryPressed` and `action.danger`; each tone fg on its bg; `border.input` and `border.focus` on `bg.screen` at 3:1. **Not allowed, and the test asserts it fails:** `text.link` or `text.tertiary` on `bg.subtle`.

### Other tokens

| Group | Names and values |
|---|---|
| `space` | `xxs 2`, `xs 4`, `sm 8`, `md 12`, `lg 16`, `xl 24`, `xxl 32`, `xxxl 48`. Screen side gutter is `lg` |
| `radius` | `sm 8` (chips), `md 12` (inputs, buttons), `lg 16` (cards, tiles), `xl 24` (bottom sheet top, hero card), `full 9999` (pills) |
| `elevation` | `none`, `card` (y 2, blur 8, opacity 0.06, Android 2), `raised` (y 6, blur 16, opacity 0.08, Android 4), `sheet` (y -4, blur 20, opacity 0.12, Android 8). Shadow color `#16324F` |
| `motion` | `fast 120`, `base 200`, `slow 320` ms; easing `standard` = ease-out. With reduced motion on, every duration becomes 0 and no looping animation runs (skeleton shimmer becomes a static fill) |
| `size` | `touchMin 48`, `iconSm 16`, `iconMd 20`, `iconLg 24`, `tile 48`, `tileLg 56`, `avatar 32`, `buttonHeight 52`, `inputHeight 52`, `tabBarHeight 64` |
| `font` | `regular 'Inter_400Regular'`, `medium 'Inter_500Medium'`, `semibold 'Inter_600SemiBold'`, `bold 'Inter_700Bold'`. Fallback is the system font while loading |

### Text styles (`textStyles`)

| Name | Size / line height | Font | Use |
|---|---|---|---|
| `display` | 30 / 36 | bold | Greeting, sign-in title |
| `title1` | 24 / 30 | bold | Screen title |
| `title2` | 20 / 26 | semibold | Section header |
| `title3` | 17 / 22 | semibold | Card title, list row title |
| `body` | 16 / 24 | regular | Default text |
| `bodyStrong` | 16 / 24 | semibold | Emphasis, button text |
| `callout` | 15 / 20 | regular | Secondary lines in rows |
| `caption` | 13 / 18 | medium | Pills, timestamps, helper text |
| `overline` | 12 / 16 | semibold, letter spacing 0.4 | Small group labels |

`maxFontSizeMultiplier` is 1.6 on every `Text` by default, and `display` and `title1` are capped at 1.3. Letter spacing is 0 except `overline`. Text styles carry no color; `Text` takes a `tone` prop of `'primary' | 'secondary' | 'tertiary' | 'inverse' | 'link' | 'danger'`.

### Status helpers (`status.ts`, task 5)

```ts
type DoseDisplayState =
  | { kind: 'upcoming'; scheduledAt: Date }
  | { kind: 'dueNow' }
  | { kind: 'snoozed'; remindAt: Date }
  | { kind: 'taken'; takenAt: Date }
  | { kind: 'skipped' }
  | { kind: 'missedByUser' }
  | { kind: 'notConfirmed'; scheduledAt: Date }
  | { kind: 'pausedOrStopped' };

doseStatusDisplay(state, formatTime): { labelKey, labelParams, tone, icon } | null
```

| kind | Label | Tone | Icon |
|---|---|---|---|
| `upcoming` | "Upcoming · 9:00 PM" | `info` | `time-outline` |
| `dueNow` | "Due now" | `info` (filled variant) | `alarm-outline` |
| `snoozed` | "Reminder at 8:15 AM" | `caution` | `notifications-outline` |
| `taken` | "Taken at 7:23 AM" | `success` | `checkmark-circle` |
| `skipped` | "Skipped" | `neutral` | `remove-circle-outline` |
| `missedByUser` | "Marked missed" | `neutral` | `ellipse-outline` |
| `notConfirmed` | "Not confirmed" | `neutral` | `help-circle-outline` |
| `pausedOrStopped` | returns `null` (no dose shown) | | |

The helper takes no clock and makes no inference. Deciding that time has passed belongs to the Phase 12 engine, which passes `notConfirmed`. The helper must not accept a raw `DoseStatus` from the old store.

`trustLabelDisplay(kind)` takes `'fromDocument' | 'addedByYou' | 'fromDoctor' | 'aiDraft' | 'verifiedByYou' | 'uncertain'`. It returns tone `source` for document and doctor, `neutral` for added by you, `caution` for AI draft and uncertain, and `success` for verified by you. Each kind has its own icon.

### Icons

`Icon` takes `name: IconName`, a union of Ionicons names listed in `design/components/Foundations/Icon.tsx`. It also takes `size: 'sm' | 'md' | 'lg'` and `tone`. The medicine-form icons (`tablet`, `capsule`, `liquid`, `injection`, `inhaler`, `drops`, `topical`, `other`) map to neutral outline icons, never colored pill pictures.

### Component names and key props

| Group folder | Components |
|---|---|
| `Foundations` | `Text`, `Icon`, `Logo` (`variant: 'mark' \| 'full'`), `WaveHeader`, `Spacer`, `Divider`, `Screen` (scaffold: safe area, gutter, optional `WaveHeader`, `scroll`, `onRefresh`), `RefreshScroll` |
| `Actions` | `Button` (`variant: 'primary' \| 'secondary' \| 'text' \| 'danger'`, `loading`, `disabled`, `icon`), `IconButton` (requires `accessibilityLabel`), `PasskeyButton` (visual only), `LinkRow`, `UndoToast` (`message`, `onUndo`, 6 s) |
| `Inputs` | `TextField` (`label`, `helper`, `error`, `leadingIcon`), `EmailField`, `PasswordField` (`rules?: PasswordRule[]` shows live checks), `OtpField` (`length: 6`), `SearchField`, `MultilineField`, `NumberUnitField` (`units: string[]`), `Checkbox`, `Radio`/`RadioGroup`, `Switch`, `ChipGroup` (`multi`), `SelectSheet`, `DateField` (`allowPartial`: year, month and year, or full date), `TimeField`, `SeverityScale` (1-5, labelled words, no colors alone) |
| `Containers` | `Card` (`variant: 'plain' \| 'hero'`), `SectionHeader` (`onViewAll?`), `ListRow` (`leading`, `title`, `subtitle`, `trailing`, `onPress`), `IconTile` (`tone`), `Accordion`, `SegmentedControl`, `BottomSheet`, `Dialog`, `Stepper` |
| `Navigation` | `TabBar` (5 tabs: Today, Records, Medicines, Health Plan, Profile; presentational only), `TopBar`, `PatientPill`, `NotificationBell` (`count`, shows 9+), `PatientBanner`, `FilterBar` (`onClearAll`) |
| `Feedback` | `Skeleton` (`shape: 'line' \| 'block' \| 'circle'`), `EmptyState` (`title`, `body`, `action?`), `ErrorState` (`onRetry`), `OfflineBanner` (`lastUpdated`), `SyncIndicator` (`state: 'synced' \| 'pending' \| 'error'`), `Toast`, `ProgressBar`, `PermissionPrompt` |
| `Trust` | `StatusPill` (takes a `doseStatusDisplay` result), `SourceLabel`, `VerificationLabel`, `UncertainMarker`, `RevisionLink`, `Disclaimer`, `FreshnessStamp` (`asOf: Date`) |

Shared rules for every component:
- Every pressable has a hit area of at least `size.touchMin`, an `accessibilityRole` and an `accessibilityLabel`.
- Every user-visible string comes from `t()`. Gallery sample data may be literal, but must be synthetic.
- Every component has a `testID` prop.
- No component reads global state or the network.

### Design-review changes from the reference images

These follow roadmap section 5 and are applied in the gallery.
- The design's red "Due Now" pill becomes the `info` filled pill. Red is never used for a dose.
- The gradient sign-in button becomes a flat `action.primary` fill.
- Colored pill pictures become neutral form icons.
- The "Tonight" purple pill is dropped. Upcoming covers it.
- The wave appears only in `WaveHeader`, and the rest of the screen is white.
