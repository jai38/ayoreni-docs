# Phase 3 Review: Design system

**Reviewer:** Opus. **Date:** 5 November 2026. **Brief:** `docs/phases/03-brief.md`.
**Branches reviewed:** `myhealthapp` `phase-3/design-system` at `7922d3e`, plus the review fixes below (uncommitted when this was written). `myhealthapp-backend` and `landing`: not touched.

## 1. Brief match

- [x] Every task in the brief's table has a result in its Results section, except tasks 17 (this review) and 18 (owner phone check, still open).
- [x] Each task's "done when" holds:
  - Tokens, contrast and touch tests, status helpers, strings and all seven component groups are built and exported.
  - The gallery opens in dev and is unreachable in release, with a test.
  - 25 screenshots were compared with `designs/`.
- [x] All work is on `phase-3/design-system`. Nothing was merged to `main`, and branch pushes run no CI.
- [x] Scope. Outside `design/` and `test/design/`, only three files changed:
  - `app/_layout.tsx`: a 12-line dev-only gallery bypass;
  - `app/dev/gallery.tsx`: the route;
  - `package.json`: `expo-font`, `@expo-google-fonts/inter` and `react-native-svg`, all added through `npx expo install`.
  - No existing screen, `theme/` or `components/` file changed.
- [x] Recorded deviations:
  - The in-gallery font-scale toggle was dropped; the large-text check moves to the phone.
  - The logo placeholder is an SVG component, not a PNG.

## 2. Rules (`CLAUDE.md`, Non-negotiable rules)

- [x] **1-2 AI tools and secrets.** No model opened `.env` or held credentials. All gallery and test data is synthetic: Asha Rao, Ravi Rao and generic medicines. Storage is untouched.
- **3-9 Ownership and security.** n/a: no backend, auth, storage or upload change.
  - Rule 8: no secrets and no health data were added to the mobile bundle or logs, and there is no `console` call in `design/`.
- [x] **Medical data.** Phase 3 only displays states, but it fixes how they look:
  - **Rules 14 and 15** have named regression tests in `test/design/status.test.ts`:
    - Not confirmed never shows as Taken or Missed.
    - Snoozed never shows as Taken.
    - Only an explicit `missedByUser` shows "Marked missed".
    - No dose state uses the red `danger` tone.
    - The helper takes no clock.
  - **Rule 13:**
    - Partial dates never get a default day or month (tested).
    - `parseDecimal` rejects `1,5`, `.5` and `1e3` rather than guess (tested).
    - Number fields never convert units.
    - `UncertainMarker` always speaks its hint.
  - **Rule 10:** `VerificationLabel` shows "AI draft" until verified.
  - **Rules 11 and 12:** n/a.

Notes: Phase 12 must pass `notConfirmed` for an unrecorded past dose. The display helper cannot enforce that, and it is recorded in the brief.

## 3. Gate results

| Gate | Command | Result | Notes |
|---|---|---|---|
| Mobile typecheck | `npm run typecheck` in `myhealthapp` | Pass, 0 errors | After review fixes |
| Mobile tests | `npm test` in `myhealthapp` | 315 passing in 28 files, 0 skipped | 208 new design tests on top of the 107 that existed |
| Backend typecheck | n/a | Not run | Backend untouched |
| Backend tests | n/a | Not run | Backend untouched |
| Backend build | n/a | Not run | Backend untouched |
| CI | GitHub Actions | Not run | Workflows trigger only on `main` |
| Migrations | n/a | n/a | No schema change |
| Web render | Expo web, built-in browser, 375 px | Done | 25 screenshots in `myhealthapp/screenshots/phase-3/` |
| Owner demo | Task 18, Android phone, default and largest text | **Not done** | Main design sign-off |

## 4. Findings

| # | Severity | Where | Finding | Owner or model to fix | Status |
|---|---|---|---|---|---|
| 1 | Medium | `test/design/*` | Tests cover the pure style and logic modules only. No component is render-tested, because there is no React Native testing library, so layout bugs (wave, list row, button wrap) were found only by eye in the gallery. | Opus decides in the Phase 4 brief: add `@testing-library/react-native` | Open, Phase 4 |
| 2 | Medium | `Inputs/DateField.tsx`, `Inputs/TimeField.tsx` | Both read `value` once on mount and do not resync when the parent replaces it. Fine for forms; wrong for a screen that loads a saved record after first render. | Haiku, when a screen first needs it (Phase 11 or 14) | Open |
| 3 | Low | `Inputs/DateField.tsx`, `Inputs/TimeField.tsx` | Day, month, year, hour and minute boxes were announced as just "Day" or "Hour", so two date fields on one screen were indistinguishable for screen-reader users. | Opus | **Fixed**: now "Date started, Day" |
| 4 | Low | `design/gallery/sections/InputsSection.tsx` | Sample fields said "Mobile number", contradicting the email-only sign-in decision. | Opus | **Fixed**: neutral "Height" samples |
| 5 | Low | `design/gallery/Gallery.tsx` | Said "Not in release builds", but the gallery code is bundled; it is only unreachable. | Opus | **Fixed**: wording. Excluding the code from release bundles is optional later |
| 6 | Low | `Inputs/DateField.tsx` | The label uses `text.secondary`, while `TextField` labels use `text.primary`, so labels are inconsistent across inputs. | Haiku | Open, Phase 4 polish |
| 7 | Low | `Inputs/DateField.tsx` | Dates up to 2100 are accepted, so a future "date started" is possible. Callers must apply the rule for their field. | Phase 11 and 14 briefs | Open |
| 8 | Low | `Foundations/iconNames.ts` | `injection` maps to a dropper, and `drops` and `liquid` share one icon. | Opus picks icons before Phase 14 | Open |
| 9 | Low | `Containers/useReduceMotion.ts` | The first animation in the app may run briefly before the reduce-motion setting is read. Later mounts use the cached value. | n/a | Accepted |
| 10 | Low | `Inputs/OtpField.tsx` | The hidden 1×1 input for the code is unverified with TalkBack and Android code autofill. | Owner, during task 18 or Phase 5 | Open |
| 11 | Info | Whole phase | Found and fixed during the build: a spec contrast error on pressed buttons (4.0:1), the pinned and clipped wave, list-row title squeeze, button label overflow, double-toggle risk in `Switch`, double screen-reader announcement in `UncertainMarker`, and the SVG `accessible` leak on web. | Opus | Fixed |

## 5. Verdict

**Pass with follow-ups.** The code side of Phase 3 is complete. The phase closes only after task 18, the owner's approval on a real phone.

- **Blocking items:** task 18, the owner phone check at default and largest system text size.
- **Follow-ups carried forward:**
  - Phase 4: findings 1 and 6.
  - Phase 11 or 14: findings 2 and 7.
  - Before Phase 14: finding 8.
  - Phase 5: finding 10.
  - The owner's logo SVG.
  - The owner's calls on wave strength, heading weight and the patient-pill avatar.
- **Not verified in this review:**
  - Any native device run (Android or iOS).
  - TalkBack and VoiceOver.
  - Large system text.
  - Real-device performance of the skeleton shimmer on a budget phone.
