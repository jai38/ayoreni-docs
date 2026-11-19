# Phase 5 Brief: Accounts and sign-in with WhatsApp OTP

**Risk:** High. **Roadmap:** `docs/project/ROADMAP.md`, section 6 (Phase 5), as changed by D1.
**Goal:** a new user signs up with their mobile number and a code sent by WhatsApp, creates their account (S01–S03), and protects the app with app lock or a passkey; a returning user signs in the same way. No code, number or name ever reaches a log.

**Status:** Draft by the lead session, 19 November 2026. Waiting for the owner's approval and the open decisions below. Building can start before Meta approves; **going live is blocked** on Meta business verification and the authentication template (tasks 1–3).

## What changed (D1)

- **Owner decision, 19 November 2026 (revised the same day):** sign-in is a mobile number plus a one-time code sent **by WhatsApp only**. No SMS and no DLT for now; `docs/ops/dlt-registration.md` is kept for a later SMS fallback. Setup checklist: `docs/ops/whatsapp-otp-setup.md`.
- This replaces the roadmap's email, password and email-OTP sign-in. Email becomes an optional profile field (S03). Passkeys stay, but as a second step after the first sign-in, alongside app lock.
- Designs: S01 Mobile number, S02 OTP and S03 Create your account in the final design (`designes/design/screens.md` on `designes/initial-upload` in `jai38/myhealthapp`; canvas https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9). They were drawn for SMS, so the designer updates them (task 4).

## Decisions

Proposed by the lead; the owner confirms or changes each.

| # | Decision | Recommendation |
|---|---|---|
| P1 | **After the first sign-in: passkey or app lock** | **App lock in Phase 5** (device biometrics or PIN, through the maintained `expo-local-authentication`), offered right after S03 and on by default. **Passkeys follow** once the app has a verified domain and a real release key (task 3); they need both. |
| P2 | **New-device sign-in on a recycled number.** Indian operators reassign deactivated numbers, so a code alone could open someone else's records | When a number signs in on a new device and the account has a passkey or recovery code, ask for one of them as well as the code. When it has neither and the account has been inactive for 90 days or more, keep its records hidden until support verifies the user. Owner to confirm the 90 days and the support path |
| P3 | **Recovery code** | Show one once after S03, stored only as a hash; it unlocks P2 and replaces a lost device's passkey |
| P4 | **Under 18** | The roadmap blocks under-18 signup; `screens.md` shows a guardian note pending legal review. Recommend **block at launch** with a calm message, until legal review settles the guardian path |
| P5 | **The old email and password routes** | Keep them only until the app no longer calls them (working agreement rule 6), then remove them in a later PR. Existing accounts are synthetic test accounts (production started fresh in Phase 2); no migration |
| P6 | **Sending route** | AWS End User Messaging Social or the Meta Cloud API, per the checklist, to avoid a provider markup. The backend session compares them in task 6; the owner sets up the chosen one |
| P7 | **Change of number** | Out of scope; Profile, Phase 6. A user who loses the number signs in through P2/P3 |

## Hard rules

1. No model opens `.env` or receives a key, cloud credential, production database access or real patient data. The WhatsApp token lives in Parameter Store, set by the owner. Tests and the dev preview use a fake sender and synthetic numbers.
2. Work goes on `phase-5/whatsapp-otp` in both repos. Push and open draft PRs freely; merge to `main` only when the owner says.
3. Production stays untouched until the owner deploys. Terraform changes are written by the backend session and applied only by the owner.
4. Any task that fails twice goes to the next model up, or to the owner.
5. All `CLAUDE.md` invariants hold. In particular: identity comes only from the verified JWT (rule 3); no health data, phone numbers, codes or names in logs or audit records (rule 8); `validateProductionConfig()` stays intact and fails closed (rule 9).
6. **Codes:** 6 digits from a cryptographic random source; stored only as an HMAC-SHA256 with a server secret (a plain hash of six digits is reversible), compared in constant time; single use; valid 10 minutes; a new code cancels the old one.
7. **Limits** (from `screens.md`; the backend keeps them in one config): a resend no sooner than 30 seconds; 3 wrong attempts per code; 3 resends, then a 10-minute cool-down; per-number, per-device and per-IP limits; a daily cap on codes sent across the whole service, with an alarm. Rate-limit keys use an HMAC of the number, never the number itself.
8. **Production refuses fakes:** `validateProductionConfig()` fails if the fake sender, a fixed test code or a test-number allowlist is enabled in production.
9. Every string goes through `t()`. Screens use only the v4 `design/` tokens and components from Phase 4b.

## State found on 19 November 2026

- **Backend:** email and password auth (`/auth/signup`, `/login`, `/refresh`, `/logout`, reset and verify-email), documented in `docs/contracts/api.md`. `GET /auth/me` returns `userId`, `email` and `patientProfileId`. Email goes through SES.
- **App:** email sign-in screens from Phase 4; Phase 4b is in progress on `phase-4b/simple-screens`.
- **Package names:** `com.ayoreni.app`, and `com.ayoreni.app.dev` for the dev build (UI session).
- **Release signing: blocked.** CI signs the APK with the public React Native debug key, so Android autofill and passkeys can't be set up yet. A real release key comes first (owner).
- **Android autofill needs native code:** zero-tap and one-tap need a handshake with WhatsApp before the code request, plus a receiver. There's no maintained Expo package, so it's a config plugin in a development build. iPhone gets copy code and paste only: iOS doesn't autofill from WhatsApp.
- **Meta:** the owner is starting business verification (`docs/ops/whatsapp-otp-setup.md`).

## Tasks

| # | Task | Who | Model | Done when |
|---|---|---|---|---|
| 1 | Meta business verification, the WhatsApp Business number and the display name "Ayoreni", per the checklist | **Owner** | n/a | Checklist status table filled in |
| 2 | Authentication template in English: security and expiry lines, **copy code** button first (autofill added after task 3); token or AWS link stored in Parameter Store; spend alarm next to the AWS budget alarm | **Owner** | n/a | Template approved; non-secret IDs recorded in the checklist |
| 3 | A real release signing key (EAS-managed, or a keystore as a CI secret), and a domain for passkeys and Android app links | **Owner** | n/a | CI APK signed with the release key; domain chosen |
| 4 | **Designer:** update S01/S02 for WhatsApp: the copy ("We'll send a 6-digit code on WhatsApp", "We sent a code on WhatsApp to …"); replace the iOS "From Messages" suggestion with copy-and-paste; add the **"no WhatsApp on this number"** state (it may arrive only after the code request, so draw it on S02 too); add app lock, recovery code and the under-18 message | **Designer** | n/a | States locked in `screens.md` and the canvas |
| 5 | Confirm P1–P7 | **Owner** | n/a | Written answers |
| 6 | **Sending route spike:** compare AWS End User Messaging Social and the Meta Cloud API on price, Terraform support, delivery receipts and how each reports a number with no WhatsApp (straight away, or later through a webhook) | **Backend** | Sonnet | A short note in this brief; owner picks |
| 7 | **OTP endpoints**, with `api.md` in the same PR (draft shapes below): `POST /auth/otp/request`, `POST /auth/otp/verify`, and a status call or webhook path for late delivery failure. A `WhatsAppSender` interface with a Meta/AWS implementation and a memory fake for tests | **Backend** | Sonnet | Tests below pass; shapes in `api.md` |
| 8 | **Account and profile:** `phoneE164` unique and verified on the user, `fullName`, `dateOfBirth` (under-18 per P4), gender and an optional self-description, optional email, and a consent record (terms version from S01, health-data consent version from S03, with times). `fullName` on `GET /auth/me` | **Backend** | Sonnet | Migration reviewed by Opus; tests pass |
| 9 | **Sessions:** 15-minute access tokens; rotating refresh tokens with reuse detection, which revokes the whole family; device list and revoke; recovery code (P3); the new-device check (P2) | **Backend** | Sonnet | Token-theft and reuse tests pass |
| 10 | **Terraform** for the chosen route: Parameter Store entries (names only), IAM for the sender, the spend alarm | **Backend** writes, **Owner** applies | Sonnet | `terraform plan` clean when the owner runs it |
| 11 | **S01, S02, S03** to the updated design, with every state below; secure token storage (`expo-secure-store`); paste fills all six boxes; auto-verify on paste | **UI** | Sonnet | Render tests per state, light and dark |
| 12 | **App lock** (P1): offered after S03, on by default, locks when the app goes to the background, with the device PIN as fallback | **UI** | Sonnet | Render and behaviour tests pass |
| 13 | **Android autofill:** a config plugin for the WhatsApp handshake and code receiver, in a development build; copy code stays as the fallback | **UI** | Sonnet | Works on a real Android phone once task 3 lands |
| 14 | **Passkeys** (after P1, needs task 3): register after sign-in, use for P2 | **UI** and **Backend** | Sonnet | Works on a real Android phone and iPhone |
| 15 | Switch the app off email sign-in; then the backend removes the old routes (P5) | **UI**, then **Backend** | Sonnet | Old routes unused, then removed |
| 16 | Deploy to dev, then production, after Meta approval | **Owner** | n/a | Real code arrives on the owner's phone |
| 17 | Review against `TEMPLATE-review.md`, including a security pass on tasks 7–9 | Lead | Opus | `05-review.md` written |

**Order:** 1–3 start now; they take days to weeks. 4, 5 and 6 next. 7–9 then 11 and 12 can be built and tested against the fake sender before Meta approves. 13 and 14 wait for task 3. Merge order: backend, owner deploy, then the app. Production sign-in switches over only when tasks 1–2 are done.

## Draft API shapes (the backend session finalises them in `api.md`)

```ts
POST /auth/otp/request
  body: { phone: string }      // 10 digits starting 6–9, or +91…; the server normalises to E.164
  200:  { data: { requestId, codeLength: 6, expiresInSeconds: 600, resendAfterSeconds: 30, channel: 'WHATSAPP' } }
  400 INVALID_PHONE · 429 RATE_LIMITED { retryAfterSeconds } · 422 WHATSAPP_UNAVAILABLE (if known straight away)

GET /auth/otp/request/:requestId   // only if task 6 shows failures arrive late
  200:  { data: { delivery: 'PENDING' | 'SENT' | 'DELIVERED' | 'WHATSAPP_UNAVAILABLE' | 'FAILED' } }

POST /auth/otp/verify
  body: { requestId, code, device: { name, platform } }
  200:  { data: { accessToken, refreshToken, isNewUser, needsSecondFactor } }   // needsSecondFactor per P2
  400 WRONG_CODE { attemptsLeft } · 410 CODE_EXPIRED · 429 LOCKED { retryAfterSeconds }
```

The same response for a known and an unknown number, so the request call doesn't reveal who has an account.

## Screens and required states

| Screen | Drawn (final design) | Added for WhatsApp and Phase 5 |
|---|---|---|
| **S01 Mobile number** | Default; ready; invalid number; sending code | No internet; too many attempts ("try again in 10 minutes"); no WhatsApp on this number; under 18 (S03) |
| **S02 Code** | Default; code entered; verifying; wrong code with attempts left; resend available; expired | Copy-and-paste in place of "From Messages"; no WhatsApp (late failure); locked after too many attempts; verified tick (~0.6 s); no internet |
| **S03 Create your account** | Default; filled; errors on Continue; date-of-birth wheel; self-describe | Under 18; saving; save failed with Retry |
| **App lock** | *(to design)* | Offer after S03; locked screen; biometric failed, use PIN |
| **Recovery code** | *(to design)* | Shown once, with copy; entered on a new device (P2) |
| **New-device check** | *(to design)* | Passkey or recovery code needed; records hidden pending support (P2) |

## Tests

- **Gates:** `npm run typecheck` and the test suites in both repos, with counts in Results.
- **Backend regression tests (new):**
  - Codes are stored as an HMAC, never in plain text; a used or expired code fails; a resend cancels the old code.
  - The fourth wrong attempt is refused; the fourth resend inside the window is refused; limits apply per number, device and IP.
  - No phone number, code or name appears in any log line or audit record (a log-capture test across request and verify).
  - The request call answers the same for known and unknown numbers.
  - A reused refresh token revokes the whole family.
  - `validateProductionConfig()` fails with the fake sender, a fixed code or a test allowlist in production.
  - Identity still comes from the JWT; cross-patient access still returns 404.
- **App tests (new):** each S01–S03 state in both themes; paste fills all boxes; app lock engages on return from the background; tokens only in secure storage.
- **Owner check, after Meta approval:**
  1. A fresh install on Android: sign up with your number. Does the code arrive on WhatsApp, and autofill (once task 13 lands) or paste work?
  2. The same on an iPhone with copy code.
  3. A number with no WhatsApp: is the message calm and clear?
  4. Wrong code three times, then wait out the cool-down.
  5. App lock on, background the app, reopen.

## Review

Opus reviews the diffs, migrations and gate results at the end using `TEMPLATE-review.md`, with a security pass on the OTP, session and logging code.

## Cost

High-risk Sonnet work across both repos: an estimated 20–30% of a weekly cap, plus WhatsApp's per-message charge once live. Record the plan-limit reading in `usage-log.md` at the end.

## Results

- (none yet)
