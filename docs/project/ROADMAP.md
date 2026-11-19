# Ayoreni Roadmap: Prototype to Complete App

**Status:** Approved by the owner on 2 November 2026. Phase 2 cut over to AWS on 2 November 2026. Its Terraform rebuild and billing alarm tests passed; the budget alert email is pending, so Phase 2 stays open. Phase 3 (design system, Apple Health-style refresh, Ayoreni logo and icons) was approved on the phone and closed on 5 November 2026 and merged to main. Phase 4 (app shell and rebrand) was approved on the phone and closed on 6 November 2026; both branches merged to main on 6 November 2026 (app 54b239c, backend 0f28284); the backend was deployed to AWS the same day, and app-status is live. Phase 4b (simple, modern screens) was added after the owner's phone check on 6 November 2026; its brief was approved the same day (Health Plan hidden until Phase 18).
**Date:** 2 November 2026
**Sources:** `AYORENI_Master_Product_Blueprint_v1.0.docx`, `Ayoreni_Master_Product_Blueprint_v1_1.docx` (reviewed, not adopted as written; see section 12), the three code repos, `designs/` (9 images), official Claude and AWS pages listed at the end.

---

## 1. The plan in short

- **33 phases.** Phases 1-29 reach public launch and deliver all 34 Phase 1 register items. Phases 30-33 deliver the 6 Phase 2 items.
- **Each phase ships working functionality and its finished UI together.** Phase 3 builds the design system first, so later screens are assembled from shared parts.
- **Claude models only.** Opus plans and reviews, Haiku does the volume building, and Sonnet takes flagged high-risk tasks.
- **AWS from Phase 2, in the Mumbai region, for hosting, database and storage only.** Render and Supabase are retired in that phase. The bill stays at about $40-50 a month for the V1 test, and grows to 10,000 active users through configuration (section 13).
- **Prescription extraction uses Gemini for this version, by owner decision on 2 November 2026.** The plan is to switch back to Haiku 5.5 through the Claude API in a future version (section 4).
- **Sign-in is email and password with email OTP, plus passkeys.**
- **Claude build cost:** built on the owner's Claude Pro subscription with extra usage off, so there is no per-token bill. The limit is the plan's weekly cap. Re-priced in Phase 4: roughly 15–25% of a weekly cap per phase, with a stop rule at one full weekly cap for a single phase (section 4).
- **87 screens and every shared component have a named phase** (sections 7 and 8).

### Why 33 phases

Phase count comes from four limits, not from a target number:

1. **One phase covers one data domain and at most one risky database migration.** This keeps a failure easy to find and undo.
2. **One phase must fit a short Opus brief and a reviewable diff.** Smaller phases are the biggest saving, because Opus never has to reload the whole project.
3. **The blueprint fixes the order.** Identity comes before records, verified records before summaries, the medicine engine before adherence, and allergies before warnings.
4. **Each phase ends in something you can try on a phone,** so you can stop or redirect without losing work.

The earlier draft had 32. Moving to AWS early adds one phase, because infrastructure is a separate risk from stabilizing the code.

---

## 2. Where the project really stands

The app has real, reusable parts but does not match the blueprint's data model.

| Area | Today | What happens to it |
|---|---|---|
| Upload, file checks, private storage | Solid, single file only | Keep, extend to multi-page (Phase 8) |
| AI extraction and review | Works for prescriptions, whole-document level | Replace with Claude and per-field candidates with source links (Phases 2, 9) |
| Timeline | Prescription events, requires a source document | Extend the model (Phases 7, 10) |
| Medicines | One record with reminder times, and a `DoseLog` | Replace with course, version, schedule, occurrence, intake (Phase 12) |
| Patient identity | One user equals one patient | Separate account, patient and access (Phase 6) |
| Hosting | Render and Supabase free tiers | Move to AWS (Phase 2) |
| Not built | Conditions, measurements, symptoms, allergies, reactions, emergency card, Health Plan, family, trends, assistant, offline sync | New phases |
| Mobile UI | 4 tabs, large files (`index.tsx` has 1,987 lines), screens built as modals | Rebuilt on routed screens |

### Problems found during this review

1. **500+ lines of uncommitted backend work.** Auth tokens, password reset, a third-party sign-in route, `DELETE /account`, and migration `20261019175900_add_auth_tokens_audit`, dated 19 October. `CLAUDE.md` says the repos are clean and that these features are missing, so it is out of date. **It is unknown whether this migration reached production, and the local `.env` points at the production database.** Phase 1 checks this first, read-only.
2. **A storage access key is still committed** in `render.yaml`, and `CLIENT_URL` is `*`. There is still no rate limiting in `src/`.
3. **Fabricated dose history is still there.** `generateTrailingWeekDoses` remains in `services/medications/medication-store.ts`. This directly breaks the blueprint's rule on Taken and Not Confirmed.
4. **Tokens are still in AsyncStorage.**
5. **`DoseStatus` has no Not Confirmed state.** The blueprint treats that state as core.
6. **Production extraction uses a non-Claude provider whose data terms were never verified.** Phase 2 removes it from the code and from production configuration.

---

## 3. Decisions

### Made by you

| Topic | Decision |
|---|---|
| AI models | Claude only for building. Gemini for the product's extraction in this version, then Haiku 5.5 later |
| Navigation | Today, Records, Medicines, Health Plan, Profile, in the designer's style |
| Sign-in | Email and password with OTP, plus passkey. No third-party sign-in |
| Infrastructure | AWS from the start for hosting, database and storage, as light as possible for V1, and easy to scale |
| Extraction | Gemini through its paid API for this version; switch to Haiku 5.5 later. No AI on AWS |
| Version 1 | One-month test with about 100 users, all your own testers, using real records |
| Specialists | Clinician, DPDP lawyer and pentester, engaged before V1. Your earlier choice was one engagement before beta; real records in V1 require a short first review, then the full review in Phase 28 |

### Made by me, change any you disagree with

| Topic | Decision | Reason |
|---|---|---|
| Name | Rename to Ayoreni everywhere | Blueprint and designs agree |
| Stack | Keep Expo, Express, Prisma, Postgres, adding only what is needed | A rewrite wastes working code. v1.1 also suggests NestJS or Fastify, with no benefit worth the rewrite |
| OTP channel | Email, at signup, password reset and sign-in on a new device | Follows your email sign-in choice. No SMS cost and no India DLT registration, and it removes the SIM-swap risk |
| Passkeys | Offered after signup, preferred once set up. Password stays as fallback | Budget phones and shared devices will not always support them |
| Expo SDK | Check the upgrade in Phase 1, upgrade in Phase 4 before new screens | Cheaper before there are 87 screens to retest |
| Web app | Stays as a read-friendly build on S3 and CloudFront, not a release gate | Blueprint is Android and iOS; parity testing costs a lot |
| Dark mode | Tokens are ready, not shipped at launch | Blueprint says "if later approved" |
| Language | English at launch with all text externalized; Hindi in Phase 26 | Translation is cheap if strings are externalized early |
| Patient data in dev | Synthetic only, always | No prescription, report or real name ever goes to a coding model |

### A change to the approved register

FR-020 says "Mobile OTP verifies account; email optional". Your email decision reverses this, so it needs recording in the blueprint's decision log as **D41**.

**The product risk:** some older patients and caregivers in India use a phone number and rarely check email. Beta in Phase 29 measures signup completion for this group. If it is poor, adding a phone OTP later is a contained change, because OTP delivery sits behind one interface.

---

## 4. Which model does what, and what it costs

### Prices checked on 2 November 2026

| Model | Input per 1M | Output per 1M | Cache read | Cache write | Role |
|---|---|---|---|---|---|
| Opus 5.5 | $4 | $20 | $0.20 | $5 | The brain: briefs and final review only |
| Sonnet 5.5 | $2 | $10 | $0.10 | $2.50 | High-risk tasks and escalations |
| Haiku 5.5, prompts up to 100K | $0.10 | $0.50 | $0.01 | $0.125 | Volume building and mechanical work |
| Haiku 5.5, prompts over 100K | $0.50 | $2.50 | $0.05 | $0.625 | Avoid: keep sessions small |

Opus output costs 40 times Haiku's, so every line Opus does not write saves real money. Fable 5.1 is not used.

### Who does what

| Work | Model | Why |
|---|---|---|
| Phase brief: scope, schema, state rules, screen list, tests | **Opus** | Mistakes here are the most expensive |
| Final review of a phase diff against the brief | **Opus** | One focused pass with no file exploring |
| Summarizing the code a brief needs | **Haiku** | Cheap, and saves Opus reading files |
| Building screens, components and CRUD routes from a brief | **Haiku** | Largest volume of work |
| Comparing screenshots to the design | **Haiku** | Reads images cheaply |
| Writing tests from the brief's list, strings, seed data, docs | **Haiku** | Volume |
| Running checks, fixing type and lint errors, renames | **Haiku** | Cheap and accurate for small edits |
| Database migrations, medicine state rules, auth, encryption, sync, permissions, Terraform | **Sonnet** | Failures here hurt patients or lose data |
| Anything that failed twice with a cheaper model | **Sonnet**, then Opus | Stops wasteful retry loops |

Phase 1 pilots Haiku on one real screen. If it cannot build screens to standard, screen work moves to Sonnet, which is the "high" budget case below.

Each phase below is tagged **Risk: Low, Med or High**, which sets how much goes to Sonnet.

### How every phase runs

1. **Brief.** A new Opus session writes `docs/phases/NN-brief.md` from a Haiku-made code summary. The brief assigns a model to each task and marks tasks for you (section 11).
2. **Build.** Haiku works through tasks on a branch. Sonnet takes tasks marked High.
3. **Gate.** Scripts run typecheck, tests, the safety rule tests and screenshots. No AI tokens are spent unless something fails.
4. **Review.** Opus reviews only the diff and the gate results, once.
5. **Demo.** You try it on a phone and approve. Only then does the next phase start.

### Claude cost estimate: building the app

**Re-priced in Phase 4 (5 November 2026), on the owner's decision to measure against Pro plan limits.**

The build runs on the owner's **Claude Pro subscription**, with extra usage turned off. There is no per-token bill: the subscription fee is the whole Claude cost of building. The real limit is the plan's usage caps. Hitting a cap pauses the work until it resets; it does not cost more. The per-token prices above still apply to the product's own API use (next section) and to any future switch to API billing.

**Measured so far.** One reading, from the app's usage card. It is one data point, not a trend.

| Read on | 5-hour window | Weekly cap (all models) | Work in the weekly window so far |
|---|---|---|---|
| 5 Nov 2026 | 51% | 21% | The window started 3 Nov. It covers the end of Phase 3 (inputs and buttons, logo and icons, close-out) and Phase 4 tasks 2–11 and 9. |

So about one and a half phases of work used about a fifth of a weekly cap. Phase 4 was Medium risk, but heavy: an SDK upgrade, about 10,000 lines restyled across nine agent runs, and seven bug fixes.

**Projection.** Treat this as rough until three more phases are logged.

| | Assumes | Weekly cap per phase | 28 phases left |
|---|---|---|---|
| Light | Phases like Phase 4, with Haiku doing most of the volume | about 15% | about 4–5 weeks of caps |
| Likely | High-risk phases send more to Sonnet, plus rework | about 25% | about 7 weeks of caps |
| Heavy | Sonnet builds most of it, with repeated reviews | about 40% | about 11 weeks of caps |

In practice your phone checks and approvals between phases, not the cap, are likely to set the pace.

**Stop rules.** These replace the old $200 hard stop.
- **Watch:** one phase uses more than 40% of a weekly cap. Check the routing: is Opus writing code, or are Haiku tasks too large?
- **Stop and re-plan:** one phase needs more than a full weekly cap, or two weeks in a row hit the cap with the phase unfinished.
- **Extra usage stays off.** Turning it on, or moving to Max or API billing, is your decision and changes this section back to a dollar model.

**Logging.** After each phase, record the 5-hour and weekly percentages from the usage card in `docs/phases/usage-log.md`, with the dates the window covers. Values are read, never estimated.

### Claude cost estimate: running the product

**This version uses Gemini.** The app already has a Gemini provider behind a provider interface, so the later switch is contained. **Future version: switch back to Haiku 5.5 through the Claude API,** after comparing accuracy on the same test set. Haiku looked cheaper on list price.

For one document page, assuming 2,500 input and 800 output tokens:

| Model | Price per 1M | V1 test: 1,000 pages a month | 10,000 active users: 100,000 pages a month |
|---|---|---|---|
| Gemini 2.5 Flash (current setting) | Confirm on Google's page | Confirm | Confirm |
| Haiku 5.5 (future) | $0.10 in, $0.50 out | about $0.65 | about $65 |

Phase 9 measures real accuracy and token use on handwritten prescriptions.

### Ways to reduce the cost

The dollar savings below come from the earlier API-billing model. On the Pro plan, read them as how much of the weekly cap each method saves; the order still holds.

| # | Method | Saving |
|---|---|---|
| 1 | **Do the tasks in section 11 yourself.** This is the largest lever, because rework is the costliest part of the estimate | $15-30 |
| 2 | **Make Haiku work:** tasks under 200 changed lines, one screen per session, and the brief states the exact component names to use | $20-40 |
| 3 | **Keep Haiku prompts under 100K tokens.** Over that, Haiku costs 5 times as much | Avoids up to 5x on Haiku |
| 4 | **A new session per task,** so no session drags old history | $10-20 |
| 5 | **Prompt caching:** keep `CLAUDE.md` and the brief stable and first. Cache reads cost 5% of Opus input | Already assumed in the estimate; losing it could double the cost |
| 6 | **Batch API at 50% off** for work needing no conversation: AI evaluation runs, translations, test data | $2-5, more on repeated evaluation |
| 7 | **Sonnet reviews Low and simple Med phases,** Opus reviews the rest | $3-5 |
| 8 | **Stop after two failures on a task** and escalate, or hand it to you | Prevents the worst overruns |
| 9 | **Gate scripts instead of model checks** | Already assumed |
| 10 | **Trim `CLAUDE.md`, and no subagents during builds** | $2-5 |
| 11 | **Stop at Phase 29** until there are real users | About $7-14 |
| 12 | **Billing:** decided in Phase 4: stay on Pro with extra usage off. Move to Max or the API only if the stop rule above is hit repeatedly | Keeps the build cost at the subscription fee |

### Safety rules for AI tools

- Coding models never receive `.env` files, keys, AWS credentials, production database access or real patient data.
- Development tools handle code only. **Real patient data reaches an AI provider only through the app's own paid production API key,** and only after the gate in Phase 9 passes. A free-tier Gemini key must never receive real records, because Google may use that content to improve its products.

---

## 5. Design direction

The designer gave a strong visual style across sign-in screens and 5 distinct Today concepts (`dashboard tab 4` and `tab 6` are the same design, and `tab` and `tab 2` are the same layout). Everything else is designed from these rules.

### What is kept
- Soft blue identity, wave motif, rounded cards, tinted icon tiles, status pills, large bold headings.
- Sign-in, signup and OTP screens, with the phone field replaced by email and social buttons replaced by "Use a passkey".
- The Today layout from `dashboard tab 5`: one due dose, Scan and Quick Add, today's medicines, next activity. It is the closest to the blueprint and already uses its five tabs.
- The leaf logo from the other images, as a placeholder until you supply the vector.

### What I am changing, and why

| In the design | Change | Reason |
|---|---|---|
| "Overdue by 1 hr 41 min" in red | Calm "Not confirmed · scheduled 8:00 AM" | Blueprint: an untouched dose is not a failure and is never presumed missed |
| "End-to-end encryption" on sign-in | "Encrypted in transit and in storage", once verified | The server reads documents for extraction, so the original claim would be false |
| Phone number and social buttons on sign-in | Email, password and passkey | Your sign-in decision |
| "Health at a Glance" tiles, Recent Documents, Quick Actions grid, carousel | Removed from Today | Blueprint: Today stays small. These move to Records and Profile |
| Colored pill pictures | Neutral icon by form (tablet, capsule, liquid) | A picture may not match the real medicine |
| "Taken" button only | "Taken" plus an undo message and actual-time edit | Accidental taps must be correctable |
| Tinted gradient over whole screen | White background, wave only in the header | Blueprint asks for white; better contrast and speed on budget Android |
| Two different logos | One logo | Consistency |
| Greeting quote, sun illustration | Removed | Space goes to the task, and it feels wrong on a bad health day |

### Rules for every screen

**Tokens.** Calm Glass UI v4, light and dark, from the designer's frozen spec (`designes/design/colour-shape-spec.md` on the `designes/initial-upload` branch of `myhealthapp`; owner decision, 19 November 2026). It replaces the blueprint palette and the white-background rule. Status colors are separate from brand colors. Text is 4.5:1 or more, tested automatically.

**Font.** Plus Jakarta Sans for Latin text (replacing Inter, 19 November 2026), Noto Sans Devanagari for Hindi. Both are free. Text scales with system settings.

**Layout.** One main action per screen. Touch targets at least 48 px. Patient name always visible when more than one profile exists. Works on a small Android phone and at large text sizes.

**Trust marks.** Every health fact can show its source: From document, Added by you, From doctor's record, AI draft, Verified by you. Upload date and event date are never confused.

**Device privacy.** Lock-screen reminders are generic. Health screens block screenshots on Android, with a setting to allow them. The app locks when backgrounded.

### Conditional rendering: required states

Every screen must handle every state that applies to it. A phase is not done until screenshots of each exist.

| State | What the user sees |
|---|---|
| Loading | Skeleton shaped like the content, not a lone spinner |
| Empty, first time | What this is and one clear next step |
| Empty, filtered | "No results" and Clear all filters |
| Partial data | Available data, with missing fields shown as optional |
| Error | Plain message, Retry, nothing lost |
| Offline | Cached data, last-updated time, disabled actions explained |
| Pending sync | Saved on device and waiting to sync |
| Processing | AI or upload progress, safe to leave the screen |
| Uncertain | Marked "Uncertain" or "Unverified", never guessed |
| Permission denied | Why it matters, how to allow it, what still works |
| Other patient active | Clear patient banner on capture and export |
| Caregiver read-only | Edit actions hidden, with a reason |
| Corrected or outdated | Marked, with revision history one tap away |
| AI switched off | Clear message, with the manual path still open |

### Medicine status display

| State | Label | Trigger |
|---|---|---|
| Upcoming | Upcoming · 9:00 PM | Before the dose window |
| Due now | Due now | Inside the window |
| Snoozed | Reminder at 8:15 AM | Reminder moved; dose unresolved |
| Taken | Taken at 7:23 AM | User confirmed |
| Skipped | Skipped | User chose |
| Missed | Marked missed | User chose only |
| Not confirmed | Not confirmed | Time passed with no action |
| Paused or stopped | No dose shown | Course state |

---

## 6. The phases

**Each phase's exit requires:** typecheck and tests pass, safety tests pass, required states have screenshots, Opus review done, your demo approval. Only extra exit conditions are listed below. FR, J and Q numbers refer to the blueprint.

### Stage A: Foundations

#### Phase 1: Stabilize and set up the AI process · Risk: High
- **Functionality:** Read-only check of whether the pending migration reached production, and whether production holds real user data. Review, test and commit or rework the uncommitted backend work, removing its third-party sign-in route. Remove the production database from local `.env` and use Docker locally. Move the storage key out of `render.yaml`, after you rotate it. Remove fabricated dose history. Correct `CLAUDE.md`, add the invariant test kit and synthetic document set.
- **AI setup:** Brief and review templates, per-phase usage log with spending limit, screenshot script. Pilot Haiku on one real screen.
- **UI:** None.
- **Exit:** Local work cannot touch production. Fake Taken history is gone. Haiku's pilot result recorded.

#### Phase 2: AWS foundation · Risk: High · v1.1 section 26A, D15, D20
- **Functionality:** Terraform for the one lean V1 environment in Mumbai, per section 13: API and an on-demand worker on Fargate, API Gateway, RDS Postgres, private S3 with KMS, Parameter Store, SQS, SES, CloudWatch with budget alerts. Apply the scale-ready rules in section 13, including running migrations as a deploy step instead of on boot. GitHub Actions deploys through OIDC with no stored AWS keys. Gemini stays as the extraction provider for this version, with its key in Parameter Store. SES replaces Resend for email. Start fresh on AWS with no data moved, then shut down Render and Supabase.
- **UI:** None. The app points at the new API.
- **Exit:** Existing flows work on AWS. The environment can be rebuilt from Terraform. Billing alerts fire in a test.

#### Phase 3: Design system · Risk: Low
- **Functionality:** Tokens, typography, icons, motion rules, status logic helpers, text externalization.
- **UI:** All base components in section 8, each with its states, in a dev-only gallery (Y4). Automated contrast and touch-size checks.
- **Exit:** You approve the look on a real phone. This is the main design sign-off for the project.

#### Phase 4: App shell and rebrand · Risk: Med
- **Functionality:** Expo upgrade if approved. App name, icons, splash. Routed navigation for five tabs. Split the oversized screens. Global error, offline and update handling. Existing features keep working in the new shell.
- **UI:** W01, T1 (first version), Y1, Y2, tab bar, header. Existing screens re-skinned as a bridge.
- **Exit:** Old flows still work. **Costs re-priced from real usage.**

#### Phase 4b: Simple, modern screens · Risk: Med
- **Why:** added 6 November 2026 after the owner's phone check found the Phase 4 screens overwhelming and dated. Brief: `docs/phases/04b-brief.md`.
- **Functionality:** none new. A dev-only screen preview with synthetic data, render tests, and before and after screenshots.
- **UI:** the final design (Calm Glass UI v4, re-based 19 November 2026): v4 tokens with light and dark, Plus Jakarta Sans, four tabs, then S06 Today and S07 Dose update. S01–S05 move to Phases 5 and 6; Records, Medications and Profile are redesigned after Medication Details.
- **Exit:** a new user can add a medicine without help; nothing overlaps or clips at 360 × 800 or the largest text size.

#### Phase 5: Accounts and sign-in · Risk: High · FR-020 as changed by D41, FRX-15, J01, Q-02, NFR-27, D17, D40
- **Functionality:** Email and password. Email OTP at signup, reset and new-device sign-in, sent through SES. Passkeys through WebAuthn on web and a native module on mobile, which requires a development build and a verified domain. 15-minute access tokens, rotating refresh tokens with reuse detection, device list and revoke. Secure token storage. Rate limits, strict CORS, consent version log. Recovery code, shown once and stored hashed. Re-authentication for sensitive actions. Date of birth check blocking under-18 signup. Real `isVerified`.
- **UI:** W02-W10, I07. The final design's S01 Mobile number, S02 OTP and S03 Create your account land here: the owner chose a mobile number with a one-time code by WhatsApp only (D1, 19 November 2026), which replaces email OTP at sign-in. The functionality line above is rewritten in the Phase 5 brief. Error states: expired OTP, resend timer, email in use, wrong password, passkey unavailable, under 18, no network.
- **Exit:** Tests for brute-force limits, reused OTP, token theft recovery, and recovery rules. Passkey works on a real Android phone and iPhone.

#### Phase 6: Patients and profile · Risk: High · FR-016 (base), Q-01, Q-03
- **Functionality:** Separate Account, Patient and PatientAccess. All data access checks patient access through one guard, with Postgres row-level security as a second layer. Move existing data safely. Progressive profile, medical background, emergency contact fields, profile-switch context in every request.
- **UI:** I01, I02 (basic), I06. From the final design: S04 Your health profile (height, weight, blood group, allergy status and allergy list, stored but not used for warnings until Phase 16) and S05 Welcome, first-time Today.
- **Exit:** Negative tests prove no cross-patient access on every route.

#### Phase 7: Trust core · Risk: High · FR-027, FRX-06, FRX-13, FRX-17, FRX-18, J22, D38
- **Functionality:** Audit records, revision history, provenance labels, verification events, invalidation events, partial and uncertain dates, time zone handling, background job retries and outbox. Per-patient keys wrapped by KMS for the most sensitive free text. KeyRecord, AccessGrant and ConsentRecord entities.
- **UI:** C4, shared Source and Revision components.
- **Exit:** A corrected fact keeps its old version, and dependents get flagged.

### Stage B: Records

#### Phase 8: Capture and document library · Risk: Med · FR-002, FR-007, Q-04, Q-07, Q-08
- **Functionality:** One scan engine for all entry points. Multi-page capture, crop, rotate, reorder. Document pages. Type detection with override. Duplicate detection. Patient check before saving. Save drafts while offline. Delete with impact review (D09). HEIC support and metadata stripping on display copies. Malware scanning before processing. Parsing runs in the isolated worker, with no database or secret access. S3 versioning, and 5-minute view links.
- **UI:** R01 (basic), R05, R06, R07, R08, R21, R22, Y3.

#### Phase 9: Extraction and review · Risk: High · FR-013 (part), FR-030, J02-J04, Q-05, Q-06, Q-45, Q-49, D15
- **Functionality:** Typed per-field candidates with page and text source. Rule checks on numbers, units and dates. Versioned prompts and evaluation set for handwritten Indian prescriptions. Fixed output schema, prompt injection defenses, red-team document set. AI switch per feature and per patient. Separate tests and follow-ups from medicines.
- **The written gate before real data:** Google's paid API terms and data processing agreement, no training, retention, and the lawyer's view on processing outside India all confirmed. The same gate applies to Anthropic at the later Haiku switch.
- **UI:** R09, with side-by-side source and per-field accept or correct. R10.
- **Also fix in the provider:** missing dosage form and route must not default to TABLET and ORAL, and an empty result must not invent a placeholder medicine (blueprint rule: never guess).
- **Exit:** Measured accuracy and cost report for Gemini on the test set, with Haiku 5.5 run on the same set to inform the later switch. No unconfirmed field can become a fact.

#### Phase 10: Timeline and search · Risk: Med · FR-004, FRX-01, FRX-10, J16, Q-10, Q-54
- **Functionality:** Timeline as a view over all event types. Uncertain-date group. Dose summaries collapsed. Indexed patient-scoped search. Filters with one Clear all. Pagination with a 15-year test set.
- **UI:** R01 (complete), R02, R03, R04.

#### Phase 11: Quick Add, measurements, symptoms · Risk: Med · FR-003, FRX-04, FRX-05, J13, J14, Q-32, Q-33
- **Functionality:** Observations for BP, glucose, temperature, pulse, weight, SpO2 and others, with units and context. Symptom episodes, including open-ended. Manual medical events.
- **UI:** T5, R11, R16, R17, R19. Blood pressure and glucose validation messages, with no diagnosis wording.

### Stage C: Medicines and safety

#### Phase 12: Medicine engine · Risk: High · FR-001, FR-008, FR-009 (logic), FRX-07, FRX-08, FRX-19, Q-15-Q-22, Q-24, Q-25, Q-53
- **Functionality:** Catalog item, course, course version, schedule rule, occurrence, append-only intake ledger. Not Confirmed state. All recurrence types, time zone and clock rules. Migrate existing medications without inventing history.
- **UI:** T1 dose card wired to the new engine, M07.
- **Exit:** Complete state rule test table. Highest review effort of the project.

#### Phase 13: Reminders · Risk: High · FR-028, FR-032, J05, J06, Q-17, Q-23, Q-43, Q-44
- **Functionality:** Local notifications from occurrences, stable IDs, reconcile on reboot and edit, grouping, quiet settings, permission and battery guidance. Generic lock-screen text with no drug names. Notification actions require unlock, so nobody records a dose for a medicine they cannot see.
- **UI:** T2, T3, T4, permission states. Android real-device test matrix.

#### Phase 14: Medicine screens and lifecycle · Risk: Med · FR-009, FR-014, FR-022, FR-029, J07-J10, Q-13, Q-26, Q-27
- **Functionality:** Manual add, one-time log, course changes, medication review, intake history, refill reminder (off by default, no stock).
- **UI:** M01, M02, M03, M05, M06, M08, M09, M10, M12, M13.

#### Phase 15: Prescription to reminder, package scan · Risk: Med · FR-013, FR-021, J03, Q-14, Q-37
- **Functionality:** Verified lines create a course and reminders without retyping. Follow-up candidates held for Phase 18. Barcode and label candidates, always confirmed by the user. Medicine catalog source (D07).
- **UI:** R10 (full), M04.

#### Phase 16: Allergies, reactions, warnings · Risk: High · FR-010, FR-011, FRX-09, J09, J11, Q-28-Q-30
- **Functionality:** Separate suspected, confirmed and non-allergic categories. Reaction reports. Warnings at medicine entry, exact match only, with recorded acknowledgment. General emergency guidance wording, marked for clinician review.
- **UI:** S1, S2, S3, S4, S5.

#### Phase 17: Emergency card, offline, device privacy · Risk: High · FR-012, FR-031, J21, Q-21, Q-31, Q-51, D03, D39
- **Functionality:** Emergency card with freshness and export. Encrypted local cache, durable write queue with idempotent sync, conflict rules. App lock with 60-second timeout, Android screenshot blocking, rooted-device warning.
- **UI:** S6, C2, I08, offline states across all existing screens.
- **Exit:** Airplane-mode test on real devices, with no duplicate doses after reconnecting.

### V1 gate: before the first real record · Risk: High

This is a checkpoint between Phases 17 and 18, not a numbered phase. V1 uses real health records from your own testers, so DPDP duties and clinical risk apply even in a private test. **No real record enters the system until every item passes.**

- **Consent and legal (lawyer):** tester notice and consent text, privacy notice, a named contact for complaints, breach duties, and a decision on sending records to the Claude API outside India. Anthropic's data processing terms signed. If this decision is not cleared, AI stays off and testers enter records manually.
- **Clinical (clinician):** medicine states, reminder behavior, allergy warnings and emergency wording reviewed. Testers are told in writing that reminders can fail and must not be their only safeguard for important medicines.
- **Security (pentester, focused test):** API, patient isolation, sign-in, recovery, file upload and document links. Critical and high findings fixed.
- **Build, from Phases 25 and 27:** in-app account deletion and data export, including files. AI off switch for each tester. Log scrubbing tested with planted data. Point-in-time database backups, plus one recorded restore drill.
- **Environment:** the AWS environment is wiped of synthetic data and becomes V1-only. Code is tested locally and on a temporary Terraform copy before each release. Access is limited to named people with MFA.
- **Rules during the test:** adults only, each tester's own records, no real data in development tools, logs, tickets or chat with coding models.
- **At the end of the month:** each tester chooses to keep their account, or to export and delete. Accounts with no answer after a stated notice period are deleted.
- **Exit:** evidence for each item is recorded, and you sign the go decision.

Phases 25, 27 and 28 later complete the full versions of this work for public launch.

### Stage D: Connected views

#### Phase 18: Health Plan and final Today · Risk: Med · FR-025, FR-026, FRX-03, FRX-12, J19, J20, Q-35, Q-36
- **Functionality:** Appointments, tests, follow-ups, recurring tasks, reminders, completion with or without results, prescription-driven candidates.
- **UI:** H1-H5. T1 finished, with every combination of due, none, new user and all-done states.

#### Phase 19: Lab results and trends · Risk: Med · FR-017, FRX-11, J12, Q-34
- **Functionality:** Lab extraction into verified observations, comparable-unit rules, duplicates, source-specific reference ranges.
- **UI:** R18, with chart and accessible table. Every point opens its source.

#### Phase 20: Conditions · Risk: Med · FR-005, J15, Q-09, Q-50
- **Functionality:** Condition model, many-to-many links, AI-suggested links awaiting review, aliases and reversible merge, conflicting diagnoses kept with their sources.
- **UI:** R12, R13, R14.

#### Phase 21: AI summaries and assistant · Risk: High · FR-006, FR-034, J17, Q-11
- **Functionality:** 2-5 line evidence-linked summaries, as-of date, abstaining when evidence is thin, stale marking. Grounded assistant limited to the patient's verified facts and explaining terms. Evaluation for citations and made-up claims. Refuses diagnosis and dosing advice.
- **UI:** R15, A1.
- **Decision:** Structured explanations rather than open chat at launch (D30).

#### Phase 22: Doctor sharing · Risk: Med · FR-023, FR-024, FRX-02, J18, Q-12, Q-52
- **Functionality:** Scope selection, snapshot, branded PDF with sources and disclaimer, reuse of current `pdf-lib` work, device share, stale marking. Genomic files excluded by default.
- **UI:** D1, D2, D3, D4.

#### Phase 23: Adherence and engagement · Risk: Med · FR-018, FR-019, Q-41, Q-42
- **Functionality:** Separate counts for confirmed on time, confirmed taken, skipped or missed, and unknown, with the formula shown. No single unexplained score. Opt-in logging encouragement only.
- **UI:** M11. Encouragement component.

### Stage E: Full breadth

#### Phase 24: Family and caregivers · Risk: High · FR-016, FR-033, Q-39, Q-40, D02, D10
- **Functionality:** Adult caregiver invite with limited permissions and a 7-day expiry, action attribution, revoke, role matrix. Child profiles with guardian attribution stay behind a feature flag until the lawyer approves them.
- **UI:** I02 (complete), I03, I04, I05, patient banners on all capture and export flows.

#### Phase 25: Privacy center, export, deletion, genomic · Risk: High · FR-015, FRX-14, FRX-20, Q-38, Q-46
- **Functionality:** Consent history and withdrawal, full data export, deletion covering storage, derived data and backups, sensitive document handling.
- **UI:** C1, C3, C5, R20.

### Stage F: Launch

#### Phase 26: Accessibility and language · Risk: Low · FRX-16, NFR-12-14, Q-47, Q-48
- **Functionality:** Screen-reader labels, focus order, large text, reduced motion, Hindi strings, date and unit formats.
- **UI:** Fixes across all screens. TalkBack and VoiceOver walk-through.

#### Phase 27: Production infrastructure · Risk: Med · NFR-03, 10, 16, 17, 22, v1.1 Appendix A
- **Functionality:** Separate production environment from the same Terraform, sized by the tiers in section 13: Multi-AZ database and two API tasks when real patient numbers justify them, WAF, S3 object lock review, backups with a restore drill, scrubbed error tracking tested with planted data, uptime alerts, dependency scans, performance budgets on low-end Android, runbooks, SDK inventory, written support-access and breach procedures.
- **UI:** Only what the checks surface.

#### Phase 28: Specialist review · Risk: High · NFR-25, D01 and other [V] decisions
- **Functionality:** Prepare review packs. Clinician reviews medicine rules, wording, warnings and AI behavior. Lawyer reviews DPDP, policies, CDSCO classification, breach duties and guardian rules. Pentester tests app and API. Breach tabletop. Fix all findings.
- **UI:** Copy and flow changes from findings. Published policy pages.
- **Exit:** No open critical safety or security issue. **Public users are allowed only after this.** V1 testers are covered earlier by the V1 gate.

#### Phase 29: Beta, stores, launch · Risk: Med · Blueprint section 34 exit criteria
- **Functionality:** Closed beta with varied ages and phones, including older adults. Measure email OTP completion. Fix cycle. Play and App Store builds, privacy forms, store listings. Landing page rebrand using v1.1 Appendix C copy, with a working waitlist or none. Go or no-go evidence pack.
- **UI:** Beta feedback fixes, store screenshots.
- **Exit:** All 34 register items have test evidence. **Public launch.**

### Stage G: After launch (register Phase 2)

#### Phase 30: Smarter notifications and share links · Risk: Med · FR-036, FR-037, Q-56
- **Functionality and UI:** Adaptive preferences. Links with 7-day default and 30-day maximum expiry, revocation and a view log (ShareLink entity). Screens T4 (advanced), F4.

#### Phase 31: Inventory and devices · Risk: Med · FR-039, FR-040, Q-57, Q-58
- **Functionality and UI:** Stock, expiry and refill estimates kept apart from treatment. Health Connect and Apple Health import, attributed and deduplicated. Screens F2, connected-devices list in Profile.

#### Phase 32: ABHA and ABDM · Risk: High · FR-035, Q-55
- **Functionality and UI:** Consent-based linking and record import with source tracking, after partner and legal agreements. Screen F1.

#### Phase 33: Advanced AI · Risk: High · FR-038
- **Functionality and UI:** Natural-language record search, consultation preparation, deeper comparisons, each clinically validated before release. Screen F3.

---

## 7. Every screen

**87 entries:** the blueprint's 65 screens (some split for clarity), plus the extras it needs but does not list, marked **+**. All screens also follow the state table in section 5. "Key states" lists what is specific to each.

### Welcome and sign-in

| ID | Screen | Phase | Key states |
|---|---|---|---|
| W01 | Launch and auth gate + | 4 | Restoring session, expired, update needed |
| W02 | Welcome carousel | 5 | First launch only |
| W03 | Sign in | 5 | Wrong details, locked, passkey unavailable or cancelled |
| W04 | Create account | 5 | Password rules live, email in use, under 18 blocked |
| W05 | Email OTP verify | 5 | Timer, expired, too many tries, check spam |
| W06 | Forgot password + | 5 | Unknown email gives same reply |
| W07 | Reset password + | 5 | Code expired, other devices signed out, sensitive actions paused |
| W08 | Consent and legal viewer | 5 | New version needs re-consent |
| W09 | Set up passkey + | 5 | Unsupported device, skipped, added |
| W10 | Recovery code + | 5 | Shown once, confirm saved, regenerate |

### Patient and profile

| ID | Screen | Phase | Key states |
|---|---|---|---|
| I01 | Patient onboarding | 6 | Skippable fields, draft kept |
| I02 | Patient switcher | 6, 24 | One patient, many, caregiver scope |
| I03 | Add dependent and guardian consent | 24 | Hidden behind flag until lawyer approval |
| I04 | Caregiver invite and access | 24 | Pending, active, expired, revoked |
| I05 | Accept caregiver invite + | 24 | Expired, wrong account |
| I06 | Profile and medical background | 6 | Incomplete prompts, quiet |
| I07 | Account and security + | 5 | Devices, password, passkeys, recovery code, re-authentication |
| I08 | App lock and device privacy + | 17 | Biometric missing, fallback PIN, screenshot setting, rooted warning |

### Today

| ID | Screen | Phase | Key states |
|---|---|---|---|
| T1 | Today | 4, 12, 18 | New user, due, several due, all done, no medicines, review pending, offline |
| T2 | Today's schedule | 13 | All statuses, none today |
| T3 | Notification center | 13 | Empty, grouped, link to another patient |
| T4 | Notification preferences | 13, 30 | Permission off, battery guidance |
| T5 | Quick Add sheet | 11 | Items added by phase |

### Records

| ID | Screen | Phase | Key states |
|---|---|---|---|
| R01 | Records landing | 8, 10 | Segments fill in as phases ship |
| R02 | Timeline | 10 | Uncertain dates, filters, collapsed doses |
| R03 | Event detail | 10 | Source, revisions, no source |
| R04 | Universal search | 10 | No results, recent, offline limited |
| R05 | Document library | 8 | Processing, failed, duplicate flag, quarantined |
| R06 | Scan or import | 8 | Camera denied, wrong patient warning |
| R07 | Capture review | 8 | Blurry warning, reorder, too many pages |
| R08 | Document viewer | 8 | Link expired, can't open, offline copy |
| R09 | Extraction field review | 9 | Uncertain, unreadable, partial failure, AI off |
| R10 | Apply suggestions | 9, 15 | Historic vs current, nothing selected |
| R11 | Manual medical event | 11 | Partial date, attachments |
| R12 | Condition list | 20 | None, suggested |
| R13 | Condition detail | 20 | Little data, conflicting sources |
| R14 | Condition create, edit, merge | 20 | Duplicate label, undo merge |
| R15 | AI condition summary | 21 | Outdated, not enough evidence, AI off |
| R16 | Measurements list and detail | 11 | By type, corrected value |
| R17 | Add measurement | 11 | Unit choice, unusual-value check |
| R18 | Trend chart and table | 19 | One point, mixed units blocked |
| R19 | Symptom episode | 11 | Open-ended, approximate start |
| R20 | Genomic document viewer | 25 | Sensitive notice, no interpretation |
| R21 | Processing status + | 8 | Queued, scanning, running, failed, retry |
| R22 | Duplicate comparison + | 8 | Keep both, link, mark duplicate |

### Medicines

| ID | Screen | Phase | Key states |
|---|---|---|---|
| M01 | Medicine list | 14 | Active, paused, stopped, completed, none |
| M02 | Medicine detail | 14 | Prescribed vs Added by You, version history |
| M03 | Add medicine manually | 14 | Allergy warning, no catalog match |
| M04 | Package scan review | 15 | Ambiguous, no match, label only |
| M05 | Create or edit course | 14 | Open-ended, taper needs manual review |
| M06 | Schedule builder | 14 | Each recurrence type, day-31 rule, preview |
| M07 | Dose action sheet | 12 | Taken, snooze, skip, missed, correct |
| M08 | Intake history | 14 | Scheduled vs actual, corrected |
| M09 | Course change wizard | 14 | Pause, restart, modify, stop, complete, replace |
| M10 | Medication review | 14 | Items needing confirmation, none |
| M11 | Adherence calendar and summary | 23 | Unknowns shown, too little data |
| M12 | One-time medicine log | 14 | Past date, free text |
| M13 | Refill reminder setting | 14 | Off by default |

### Safety

| ID | Screen | Phase | Key states |
|---|---|---|---|
| S1 | Allergies and sensitivities | 16 | None recorded vs none known |
| S2 | Add allergy or sensitivity | 16 | Self-reported vs documented |
| S3 | Report a reaction | 16 | Unknown cause, severe-symptom notice |
| S4 | Reaction detail and history | 16 | Linked course and source |
| S5 | Medication warning review | 16 | Suspected vs confirmed, acknowledged |
| S6 | Emergency Health Card | 17 | Stale, empty fields, offline, phone Medical ID guidance |

### Health Plan

| ID | Screen | Phase | Key states |
|---|---|---|---|
| H1 | Plan dashboard | 18 | None, past target date, prescription suggestions |
| H2 | Appointment detail and editor | 18 | Rescheduled, cancelled, attended |
| H3 | Test detail and editor | 18 | Recurring, recommended vs scheduled |
| H4 | Follow-up detail and editor | 18 | Recommended, scheduled, completed |
| H5 | Completion and attach result | 18 | No result yet, link later |

### Sharing and AI

| ID | Screen | Phase | Key states |
|---|---|---|---|
| D1 | Doctor summary setup | 22 | Wrong patient guard, nothing selected |
| D2 | Doctor summary preview | 22 | Omitted sections, sensitive excluded |
| D3 | Export branded PDF | 22 | Failed, retry, can't-revoke notice |
| D4 | Past summaries + | 22 | Outdated marker |
| A1 | Health assistant + | 21 | Refusal, no evidence, consent off, AI off |

### Settings

| ID | Screen | Phase | Key states |
|---|---|---|---|
| C1 | Privacy and consent center | 25 | Withdrawal impact explained, AI switches |
| C2 | Offline and sync status | 17 | Queue, conflict, all synced |
| C3 | Export, recovery, deletion | 25 | Re-authenticate, export ready, deletion confirm and grace |
| C4 | Revision and audit history | 7 | No edits, compare versions |
| C5 | Help and about + | 25 | Support, grievance contact, version |

### System

| ID | Screen | Phase | Key states |
|---|---|---|---|
| Y1 | Force update and maintenance + | 4 | Required vs optional |
| Y2 | Global error and no connection + | 4 | Retry |
| Y3 | Permission explanation sheets + | 8, 13 | Camera, notifications, files |
| Y4 | Component gallery + (developer only) | 3 | Not in release builds |

### Future (register Phase 2)

| ID | Screen | Phase |
|---|---|---|
| F1 | ABHA linking and consent | 32 |
| F2 | Inventory and expiry | 31 |
| F3 | Advanced AI conversation | 33 |
| F4 | Share-link manager | 30 |

---

## 8. Every shared component

All are built in Phase 3 unless a phase is named. Each has its states shown in the gallery.

| Group | Components |
|---|---|
| Foundations | Text styles, Icon, Logo, Wave header, Spacer, Divider, Screen scaffold, Scroll with refresh |
| Actions | Primary, secondary, text and danger buttons; Icon button; Passkey button; Link row; Undo message |
| Inputs | Text, email, password with rule checks, OTP boxes, search, multiline, number with unit, checkbox, radio, switch, chips, select sheet, date with partial-date option, time, severity scale |
| Containers | Card, Section header with View All, List row, Icon tile, Accordion, Tabs/segmented control, Bottom sheet, Dialog, Stepper |
| Navigation | Tab bar, Top bar, Patient pill, Notification bell with count, Patient banner, Filter bar with Clear all |
| Feedback | Skeletons, Empty state, Error state, Offline banner, Sync indicator, Toast, Progress, Permission prompt |
| Trust | Status pill, Source label, Verified or draft label, Uncertain marker, Revision link, Disclaimer block, Freshness stamp |
| Security (Phase 5, 17) | Re-authentication prompt, Recovery code display, Device row, App lock screen |
| Medicine (Phase 12, 14) | Dose card, Dose row, Medicine form icon, Course status, Schedule summary, Version compare |
| Records (Phase 8-11) | Document card, Page thumbnail editor, Timeline item, Date group, Field review row, Source highlight, Measurement row, Symptom card |
| Safety (Phase 16, 17) | Allergy chip, Warning panel, Emergency card block |
| Plan (Phase 18) | Plan task card, Appointment row, Next-activity card |
| Data views (Phase 19, 23) | Trend chart, Chart data table, Reference band, Adherence calendar, Count summary |
| AI (Phase 21) | Summary card, Citation link, Assistant message, Refusal message, AI-off notice |
| Sharing (Phase 22) | Scope checklist, PDF preview, Share notice |

Existing `AdherenceDonut`, `AiOmnibar` and `HorizontalDateStrip` are retired. The donut would imply one simple adherence score, which the blueprint rejects.

---

## 9. Blueprint coverage

### Register items to phases

| FR | Phase | FR | Phase | FR | Phase | FR | Phase |
|---|---|---|---|---|---|---|---|
| 001 | 12 | 011 | 16 | 021 | 15 | 031 | 17 |
| 002 | 8 | 012 | 17 | 022 | 14 | 032 | 13 |
| 003 | 11 | 013 | 9, 15 | 023 | 22 | 033 | 24 |
| 004 | 10 | 014 | 14 | 024 | 22 | 034 | 21 |
| 005 | 20 | 015 | 25 | 025 | 18 | 035 | 32 |
| 006 | 21 | 016 | 6, 24 | 026 | 18 | 036 | 30 |
| 007 | 8, 11 | 017 | 19 | 027 | 7, 8 | 037 | 30 |
| 008 | 12 | 018 | 23 | 028 | 13 | 038 | 33 |
| 009 | 12, 14 | 019 | 23 | 029 | 14 | 039 | 31 |
| 010 | 16 | 020 | 5 | 030 | 9 | 040 | 31 |

Journeys J01-J22 are listed in phases 5-25. Test scenarios Q-01 to Q-58 run as automated tests where possible and as device checks otherwise.

### Open decisions: proposed defaults

**[S]** means I build the default, and the specialist confirms in Phase 28.

| ID | Topic | Default | Phase |
|---|---|---|---|
| D01 | Intended use | Personal organizer, no clinical decisions; v1.1 draft statement [S] | 1 |
| D02 | Family roles | Single adult at first; adult caregivers next; child profiles after legal approval [S] | 24 |
| D03 | Emergency card access | Inside the app only, plus guidance on the phone's own Medical ID [S] | 17 |
| D04 | Missed | User-declared only [S] | 12 |
| D05 | Adherence math | Separate counts, no single score [S] | 23 |
| D06 | Complex schedules | Day 31 falls on last day of month; tapers and as-needed stored as text with manual schedule [S] | 12 |
| D07 | Medicine catalog | Free text plus a small curated list; paid source only if needed [S] | 15 |
| D08 | Cross-reactivity | None; exact match only [S] | 16 |
| D09 | Deleting a source | User chooses; verified facts keep a warning [S] | 8 |
| D10 | Who verifies | Patient only at first; caregivers may view and log, not confirm clinical facts [S] | 9, 24 |
| D11 | File limits | PDF, JPEG, PNG, HEIC; keep 15 MB; raise PDF pages after testing | 8 |
| D12 | Languages | English, then Hindi; extraction tested on both | 9, 26 |
| D13 | Trend analytes | Start with common labs; add by allow-list [S] | 19 |
| D14 | Source grades | Document, doctor, self, AI draft labels | 7 |
| D15 | Extraction AI vendor | Gemini paid API for this version, Haiku 5.5 later; real data only after the written gate [S] | 2, 9 |
| D16 | Search | Postgres full text, patient-scoped; encrypted fields excluded | 10 |
| D17 | Recovery | See section 12 [S] | 5 |
| D18 | Offline cache | Schedules, card and chosen documents, encrypted, behind app lock | 17 |
| D19 | Retention and deletion | 30-day deletion, backups aged out, exact periods from lawyer [S] | 25 |
| D20 | Data location | Storage, database and backups in India. AI extraction may process outside India, subject to the lawyer's approval [S] | 2, 9 |
| D21 | AI confidence | Doses, units and dates always confirmed by user | 9 |
| D22 | PDF identity | Name, age, date, disclaimer [S] | 22 |
| D23 | Sharing | Local PDF only; links in Phase 30, as the register says | 22 |
| D24 | Proceeding after warning | Allowed, with acknowledgment logged [S] | 16 |
| D25 | Reaction wording | General "seek emergency care" text [S] | 16 |
| D26 | Notifications | Group doses at the same time; snooze capped at 3; generic lock-screen text | 13 |
| D27 | Test completion | Self-declared allowed, result optional | 18 |
| D28 | Metrics | Set in Phase 27 from beta baseline | 27 |
| D29 | Devices | Android 10+, iOS 16+, phones first | 4 |
| D30 | Assistant form | Structured explanations, not open chat [S] | 21 |
| D31 | Engagement | Opt-in logging nudges, no dose streaks [S] | 23 |
| D32 | Pricing | Not decided; no paywall in the build | After 29 |
| D33 | Genomic | Sensitive, off in exports by default [S] | 25 |
| D34 | Doctor data | Free text | 18 |
| D35 | ABHA readiness | Keep external IDs and source fields in schema | 7 |
| D36 | Imported corrections | Local correction beside the unchanged source | 32 |
| D37 | Self-reported allergy | "Reported by you" vs "Confirmed by doctor" [S] | 16 |
| D38 | End-to-end encryption | Not at launch; server-readable, and the privacy policy says so plainly [S] | 7 |
| D39 | Rooted devices | Warn and reduce risk, do not block | 17 |
| D40 | Under-18 signup | Blocked at launch [S] | 5 |
| D41 | Account verification | Email OTP replaces mobile OTP in FR-020 (your decision) | 5 |

---

## 10. What only you can do, and other costs

Costs here are separate from the Claude estimate. AWS figures are my estimates from the prices in section 13.

| When | Item | Cost |
|---|---|---|
| Phase 1 | Create the Claude API spending limit; rotate the storage key | None |
| Phase 2 | Create the AWS account with MFA and budget alerts; run the first Terraform apply; request SES production access; own the domain | Domain about ₹1,000 a year; AWS below |
| Phase 2 onward | One lean AWS environment | About $35-45 a month, reduced by stopping it when unused and up to $200 in new-account credits |
| V1 gate | Short lawyer, clinician and pentester reviews; tester consent; sign the go decision | One-time fees, on top of Phase 28 |
| V1 test | Lean AWS environment plus a temporary test copy; run support and the end-of-test deletion | About $40-50 a month; time |
| Phase 3 | Get logo vector and approve the design system | Possibly designer time |
| Phase 5 | Apple Developer account, needed for iPhone passkey testing; Google Play account | Annual and one-time fees |
| Phase 2 | New paid Gemini API key, because the old one is blocked as leaked | Pay per page |
| Phase 9 | Accept Google's data processing terms and ask the lawyer about processing outside India | Time |
| Phase 27 | AWS production | About $60-80 a month for a small launch, about $300-450 at 10,000 active users, excluding AI use |
| Phase 28 | Clinician, DPDP lawyer, pentester | One-time fees |
| Phase 29 | Beta testers including older adults; company registration, trademark search for "Ayoreni", insurance quotes | Time and fees |

---

## 11. Where you are cheaper or better than a model

You said you can take tasks where this saves money. Each brief marks them **You**, with the model tasks built around them.

### Do these yourself

| Task | Phase | Why you beat a model |
|---|---|---|
| Review and split the 500+ uncommitted lines | 1 | You know why you wrote them; a model has to infer intent at Sonnet rates |
| AWS account, IAM, first `terraform apply`, DNS, SES request | 2 | Needs credentials that models must not hold |
| Passkey domain files, app signing, EAS and store settings | 5, 29 | Console work with secrets, and slow for a model to get right by trial |
| Expo SDK upgrade and native build failures | 4, 5 | Long log-reading loops are the costliest pattern for a model |
| Final look and small pixel tweaks | All UI | A one-line style edit takes you seconds; a screenshot loop costs tokens and may not converge |
| Real-device testing: reminders, reboot, battery savers, passkeys, airplane mode | 5, 13, 17 | Models cannot hold the phones |
| Creating the synthetic prescription set by hand | 1, 9 | Realistic handwriting cannot be generated well |
| Merge conflicts, renames, git housekeeping | All | Faster by hand |
| Approving each database migration before it reaches AWS | All High | Your check is cheap insurance on the most costly failure |
| Backup restore drill and breach tabletop | 27, 28 | They test people and process |

### Needs another human

| Task | Who |
|---|---|
| Medicine rules, safety wording, hazard log | Clinician |
| DPDP, policies, CDSCO classification, breach duties, children's data, AI data transfer terms | Lawyer |
| Independent security test | Pentester |
| Logo and any custom illustration | Designer |
| Usability with older adults | Real beta users |

### Leave to models

Repetitive screens from an existing component, state screenshots, CRUD routes, test writing from a list, string extraction, Hindi first draft (reviewed by a person), and doc updates. These are cheap on Haiku and dull for you.

---

## 12. Review of blueprint v1.1

**How I reviewed it:** a text comparison against v1.0, then a check of each new section against the register, the code and live vendor information. v1.1 only adds material, apart from the name and the contents list. I treated it as unverified, per your note that the changes came only from Haiku. **Do not treat it as approved:** its own text says v1.0 stays the baseline until you approve it.

### Verdict by change

| v1.1 item | Verdict | What the plan does |
|---|---|---|
| 1. Name | Accept | Ayoreni |
| 2. Sign-in and recovery | **Change** | Your email and passkey decision replaces phone OTP. Tokens, device list and re-authentication are kept |
| 3. Encryption and keys | **Change** | KMS everywhere, but per-patient keys only for the most sensitive free text. Encrypting everything per patient blocks search and adds cost with little gain. Honest wording on server-readable data kept |
| 4. Device privacy | **Change** | Generic reminders, app lock and rooted warning kept. Screenshot blocking gets a user setting. Locked-device emergency card rejected (below) |
| 5. Uploads and sharing | **Change** | File checks, malware scan and isolated worker move forward to Phase 8. Hosted links stay in Phase 30. Patient-phone PIN dropped |
| 6. Third-party SDKs | Accept | Phase 27 inventory, event allow-list, scrub test |
| 7. AI and residency | **Change** | Written gate and AI switches kept, with India-only AI processing left to the lawyer. "No AI in MVP" rejected (below) |
| 8. Support access | **Change** | No support access to health content at launch, not even with a ticket flow. Simpler and safer until there is a support team. Written break-glass procedure only |
| 9. Breach playbook | Accept | Phases 27, 28; timelines from the lawyer |
| 10. Children and caregivers | Accept, with a note | Under-18 signup blocked. Child profiles behind a flag |
| 11. Regulatory positioning | Accept | Lawyer reviews in Phase 28 |
| 12. Technical decisions | **Change** | AWS and Express fit it. OCR and AI rows are replaced by a Phase 9 test |
| 13. MVP | **Change** | Not adopted as written (below). Useful as an optional early release |
| 14. Document split | Skip for now | No build value; do it when the blueprint is next revised |
| 15. Production checklist | Accept | Becomes the Phase 27-29 checklist |
| 16. Landing page copy | Accept, after a fix | Removes adherence promises. The URL needs correcting (below) |

### Errors in v1.1

1. **It says it changes no approved assignment, but it does.** "No AI in MVP" and Wave D delay v1.0's Wave A upload and review work. D02 limits launch to one adult, while FR-016 and FR-033 are Phase 1. D23 brings hosted links into Phase 1, while FR-037 is approved as Phase 2.
2. **The MVP has gaps.** It says every ID stays mapped, but Quick Add, conditions, measurements, symptoms, Health Plan and search appear in neither its included nor deferred lists. Its doctor PDF covers "one condition", though conditions are not included.
3. **It contradicts itself on the emergency card.** Section 32A says "Minimal locked-device fields" while section 28A.3 makes the lock-screen card off by default.
4. **Part of the locked-device card looks unbuildable.** It wants each locked-device display logged, and a normal app cannot show and record its own screen on a locked phone, especially on iOS. The plan keeps the card inside the app and points users to the phone's own Medical ID. Phase 17 can test whether anything better exists.
5. **"No AI in MVP" gives up too much.** AI supplies much of the product's value, and Haiku 5.5 makes it very cheap. v1.1's India-only gate would block a direct API call, so the plan asks the lawyer whether processing outside India is acceptable. If the terms fail, AI stays off for real patient data and the manual path ships.
6. **Managed OCR as the MVP path is weak for this market.** Textract's handwriting support is documented as English-only in the sources I found, and they are dated. Phase 9 measures it against Claude.
7. **The 72-hour recovery wait fits the wrong risk now.** It was designed for SIM takeover and would make every forgotten password take three days if applied to email reset. See below.
8. **The recovery design assumes an email the register calls optional.** Your email decision resolves this.
9. **The share PIN goes to the user's own phone,** which doesn't verify the doctor and brings back SMS. Dropped.
10. **The landing URL looks wrong.** Appendix C names `sisanc.github.io/MyHealth`, while `CLAUDE.md` records `jai38.github.io/MyHealth`. Phase 29 uses whichever you confirm.
11. **Small errors.** The contents list has no new sections and stale page numbers. It mixes "organised" with the v1.0 "organized" tagline. "Logs are reviewed weekly and monthly reviews are recorded" is unclear. D38 and D39 are out of order in the log.

### Recovery design, replacing D17

| Situation | What happens |
|---|---|
| Forgot password, email works | Email OTP resets right away. Other devices sign out and get a notice. Export, sharing, deletion and caregiver changes pause for 24 hours, unless confirmed by passkey or recovery code |
| Lost email, has recovery code or passkey | Sign in and change the email, with a notice to the old address |
| Lost email and everything else | 72-hour manual check through support, with notices to all devices and old email. Reviewed by the lawyer |
| Any recovery | Caregiver and share settings need re-approval |

### Optional early release

v1.1 is right that the scope is large for a small team. The phase order already allows a smaller private release after Phase 17.

- **V1 test:** one month, about 100 of your own testers, using real records, on the lean AWS environment. It starts after Phase 17 and the V1 gate in section 6, which brings part of the specialist review and privacy work forward.
- **Wider private release:** needs Phases 22 (a simplified PDF) and 25 (export, deletion) brought forward, and Phases 27-28 run first. **You decide at the end of Stage C.**

No register item is removed.

---

## 13. AWS setup

**I agree with moving now.** It costs more per month than free tiers, but it avoids a second migration and gives one bill and one security model. AWS runs hosting, database and storage only. AI runs on your machine for development, and extraction calls Gemini directly for this version, then Haiku 5.5 later.

### Size by stage

These are my estimates. Scaling needs Terraform value changes and an added load balancer, with no application rewrite.

| Part | V1 test: about 100 users | Small launch | 10,000 active users |
|---|---|---|---|
| Environments | One, plus a temporary test copy before releases | Separate production | Separate production and staging |
| API | 1 Fargate task, 0.25 vCPU, 0.5 GB | 1-2 tasks | 2-3 tasks, 0.5 vCPU, 1 GB, autoscaling |
| Worker | Runs only when SQS has jobs | Same | Same, with more at peaks |
| API entry | API Gateway HTTP API | Same or load balancer | Load balancer with WAF |
| Database | `db.t4g.micro`, single-AZ, 20 GB | `db.t4g.small`, single-AZ | `db.t4g.medium` or larger, Multi-AZ, about 100 GB |
| Files | S3, a few GB | S3 | S3, about 1 TB |
| **AWS per month** | **About $40-50** | **About $60-80** | **About $300-450** |
| Extraction per month | About $0.65 | Small | About $65 |

A year at 10,000 active users comes to about $4,000-6,000 for AWS and extraction together. Reminders are local to the phone, so they add little server load. Move up a stage when real patients join, or when CPU, memory or database load stays high.

**The V1 trade-off:** with one database and one API task, a failure or a deploy can cause some downtime, and restoring from backup takes time. That is acceptable for your own testers, not for a public launch.

### V1 test bill

| Item | Per month |
|---|---|
| Database and storage | About $18 |
| API task, with public IP | About $11 |
| Worker on demand | About $2 |
| API Gateway | About $1 |
| S3, KMS, SES, SQS, Parameter Store | About $2 |
| Logs and alarms | About $2-5 |
| Temporary test copy, a few days a month | About $5 |
| **Total, with margin** | **About $40-50** |

New accounts can get up to $200 in credits, which may cover the test month.

### Scale-ready rules, from Phase 2

These cost almost nothing now, and they are what make scaling a small job.

- **Stateless API:** no files or sessions on the server, so more tasks can be added at any time.
- **Long work goes through SQS:** extraction and PDFs never run inside an API request.
- **Files live only in S3,** uploaded directly with signed links, so large files never load the API.
- **The app uses a custom domain** such as `api.<domain>`, so the entry point can change with no app update.
- **Migrations run as a deploy step.** Today they run on boot, which would collide once there are several tasks.
- **Database connections are limited per task,** and a proxy is added only when load requires it.
- **Everything is in Terraform,** with sizes as variables per environment.

### Services

| Need | Choice | Why |
|---|---|---|
| Region | `ap-south-1` Mumbai | Storage, database and backups stay in India |
| API | ECS Fargate on ARM | Docker image already exists, scales by adding tasks, and needs no servers |
| API entry | API Gateway HTTP API at first | Costs about $1 at this volume, against about $20 for a load balancer that is not needed yet |
| Worker | Separate Fargate task reading SQS, no database access | Isolation required for document parsing, and costs nothing when idle |
| Database | RDS PostgreSQL | Prisma works unchanged, and resizing or adding Multi-AZ is a settings change |
| Files | S3 private, KMS encryption with a bucket key, versioning | Bucket key lowers KMS request costs |
| Secrets | Parameter Store, encrypted | No monthly charge on the standard tier, and nothing in the repo |
| Email OTP | SES | $0.10 per 1,000 |
| Malware | GuardDuty for S3, or ClamAV in the worker if cheaper | Compared in Phase 8 |
| Monitoring | CloudWatch, budget alerts | One place |
| Edge | WAF and CloudFront at Phase 27 | Not worth paying for before real traffic |
| Infrastructure | Terraform or OpenTofu | All environments stay identical |
| Landing page | Stays on GitHub Pages | Public, no patient data, free |

**Cost controls:** no NAT gateway, ARM tasks, the worker scaled to zero when idle, the database stopped when idle during the build, 14-day log retention on V1, and a budget alarm from day one.

**Price caution:** RDS micro in Mumbai is about $0.021 an hour single-AZ from a third-party listing. AWS's Fargate page shows only US examples, about $0.032 per vCPU-hour on ARM. Other AWS figures are from memory. All monthly figures are estimates until you run them through the AWS Pricing Calculator in Phase 2.

---

## 14. Risks to this plan

| Risk | Response |
|---|---|
| Haiku output needs too many fixes | Phase 1 pilot; shift that work to Sonnet and re-price |
| Build usage runs past the plan cap | Plan-limit readings logged per phase; re-priced in Phase 4; stop and re-plan when one phase needs more than a full weekly cap |
| AWS bill grows during a long build | Budget alarms, stop the database when idle |
| Email OTP excludes some older users | Measured in beta; phone OTP can be added behind the same interface |
| Passkey native module breaks with the Expo version | Checked at the start of Phase 5; password path stays complete |
| AI provider terms or processing outside India fail the written gate | AI stays off for real data, and manual entry ships |
| V1 test has a failure or deploy downtime | Accepted for your own testers, with backups and a tested restore; testers are told not to rely only on reminders. Move up a stage before public launch |
| V1 gate delays the test | Specialists are booked by Phase 9. If only the AI transfer decision is late, V1 starts with AI off |
| A real record reaches a coding tool during V1 | Separate keys and access, named people only, written rule, synthetic data for every bug reproduction |
| Handwritten prescription accuracy is poor | Phase 9 reports it early; manual entry is always available |
| Specialists require large changes | Rules live in one place; Phase 28 has fix time, and the highest-risk wording is flagged |
| Blueprint scope is large for launch | Your call at the end of Stage C (section 12) |
| Local `.env` touches production | Phase 1's first job |

---

Sources: [Claude pricing](https://claude.com/pricing), [Fargate pricing](https://aws.amazon.com/fargate/pricing/), [SES pricing](https://aws.amazon.com/ses/pricing/), [RDS micro price listing, third party](https://sparecores.com/database/aws/db.t4g.micro), [Textract handwriting languages](https://aws.amazon.com/about-aws/whats-new/2020/11/amazon-textract-supports-handwriting-and-five-new-langauges/)
