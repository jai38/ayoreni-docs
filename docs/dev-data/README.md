# Dev test data and the dev/prod switch

Built 7 November 2026 during Phase 4b. Everything here is synthetic. Prod never receives it.

## What you get

- **About 33 scenario accounts.** Each one is a named story built to hit an edge case (table below).
- **60 bulk random users** (change with `--users N`), with records spread from **January 2000 to today**. A few accounts go further on purpose: records from 1985, a prescription dated in the future, and 29 February dates.
- A default run makes roughly **1,100–1,300 documents and prescriptions, 3,600 medicines, 9,000 dose logs and 1,100 timeline events**. The numbers move a little with today's date. Use `--now 2026-11-07T06:30:00Z` for an exact repeat.
- Every document has a real file in storage. Printed prescriptions are PDFs whose text matches the record. Handwritten ones show a "SYNTHETIC PLACEHOLDER HW-xx" image until you add the generated images.
- **Future-phase data** (allergies, conditions, BP and glucose series, lab panels, vaccinations, pregnancy, emergency contacts, caregiver links) is generated for the same people. There are no tables for it yet, so `--export <dir>` writes it to `future-phase-data.json`. Each later phase adds a writer for its own part, and testers keep the same stories.
- Every seeded email ends in `@seed.ayoreni.test`. `.test` is a reserved domain that can never be a real inbox. `--reset` and `--remove` touch only these accounts, so real sign-ups in dev are kept.

Sign-in password for all seeded accounts:
- Local: `LOCAL_DEFAULT_PASSWORD` in `myhealthapp-backend/src/scripts/seed-dev/index.ts`.
- AWS dev: the one you type into `./dev-env.ps1 SetSecrets`, stored in Parameter Store as `/myhealth/v1/dev/SEED_USER_PASSWORD`.

## How dev and prod stay apart on one AWS account

| | Prod (unchanged) | Dev (new, `infra/dev.tf`) |
|---|---|---|
| Database | RDS `myhealth-v1`, schema `public` | Same RDS instance, schema `dev` |
| Files | `myhealth-v1-documents-<acct>` | `myhealth-v1-dev-documents-<acct>` |
| Secrets | `/myhealth/v1/*` | `/myhealth/v1/dev/*` (own JWT secret, so prod tokens fail in dev) |
| API | ECS `api`, its own URL | ECS `api-dev`, its own URL |
| Extraction | SQS queue + worker | Inline in the dev API (Gemini) |
| Deploy | "Deploy to AWS (ECS)", `main` only | "Deploy to AWS (dev)", any branch, optional reseed |
| Extra cost | none | about USD 10/month while running; `Stop` pauses it |

Four separate locks keep generated data out of prod:
1. The seed refuses unless `--target dev` is passed, the database is local or uses `schema=dev`, `DEPLOYMENT_ENV=dev` is set, and the bucket is the dev bucket. Each refusal has a test in `test/seed-dev.test.ts`.
2. The API refuses to start if its `DEPLOYMENT_ENV` disagrees with its schema or bucket, for example prod pointed at dev data or dev at prod data. This is check 9 in `validateProductionConfig`, tested in `test/aws-config.test.ts`.
3. The dev roles can read only dev secrets and the dev bucket, plus the shared Gemini key. The dev GitHub role can update only `api-dev` and run only the dev migrate and seed tasks.
4. The prod deploy workflow, roles and task definitions are untouched. The Terraform plan should show additions only.

## Your steps (once)

From `myhealthapp-backend/infra`, in an SSO-signed-in PowerShell:

1. Add `dev_enabled = true` to `envs/v1.tfvars`.
2. `terraform plan -var-file=envs/v1.tfvars -out=v1.tfplan`. Check that it shows **only additions, and no change or destroy to prod**. Then `terraform apply v1.tfplan`.
3. `./dev-env.ps1 SetSecrets`. This builds the dev database URL from prod's with `schema=dev` (never printed), generates a dev JWT secret, and asks for the seed password.
4. Add the dev deploy variables to GitHub:
   ```powershell
   $vars = terraform output -json dev_github_actions_variables | ConvertFrom-Json
   $vars.PSObject.Properties | ForEach-Object { gh variable set $_.Name --body $_.Value --repo jai38/myhealthapp-backend }
   ```
5. GitHub → Actions → **Deploy to AWS (dev)** → Run workflow, with seed = `reset`. It builds, migrates the dev schema, deploys the dev API and seeds it.
6. In the app's `.env`, set `EXPO_PUBLIC_DEV_API_URL` to the output of `terraform output -raw dev_api_url`. Put the same value in `eas.json` under the `dev` profile.

## Switching (app)

```bash
npm run start:dev
```
```bash
npm run start:prod
```
```bash
npm run start:local
```

For installable builds, `eas build --profile dev` makes a dev build, and the `preview` and `production` profiles are prod. When the app starts against a different backend than last time, it clears the stored session, cached medicines and doses, and scheduled reminders. Dev data can then never sit next to real records on a phone. If the dev address is missing, the dev build fails to connect rather than falling back to prod.

## Day-to-day (owner)

`./dev-env.ps1 Status | Start | Stop | Migrate | Seed -Users 200 | Remove | UploadImages -Dir <folder>`

Local only (Docker Postgres and MinIO):
```bash
npm run seed:dev -- --target dev --reset
```
Other flags: `--users N`, `--max-rx N`, `--seed N`, `--now ISO`, `--dose-days N`, `--no-files`, `--images <dir>`, `--export <dir>`, `--dry-run`, `--remove`. If your `.env` still points storage at the old hosted bucket, the seed refuses. Set `STORAGE_ENDPOINT=http://localhost:9000` and the MinIO keys in that shell first.

## Handwritten images

Prompts: `handwritten-prescription-prompts.md` (20 images, HW-01 to HW-20). Use **Gemini Nano Banana Pro**. Save them as `HW-01.png` and so on, then:
- AWS dev: run `./dev-env.ps1 UploadImages -Dir <folder>`, then `./dev-env.ps1 Seed`.
- Local: run `npm run seed:dev -- --target dev --reset --images <folder>`.

Every seeded document from that template then shows the image. The `handwritten` account has all 20 side by side with their records. Many template patient names differ from the account holder's name, which mirrors the real case of a family member's paper uploaded to the wrong profile.

## Scenario accounts

Sign in with `<name>@seed.ayoreni.test`.

| Account | Story | What it tests |
|---|---|---|
| `empty` | Brand-new user, nothing uploaded | Empty Today, Records, Medicines and Timeline; First-run onboarding |
| `no-profile` | Account with no patient profile | Profile missing: screens must not crash; Profile-scoped API calls |
| `unverified` | Email not verified, notifications off | Sign-in refused with EMAIL_NOT_VERIFIED (403): the verify-email message; notificationsEnabled=false, no daily summary; passwordChangedAt set |
| `google-provider` | GOOGLE auth provider, no password hash | Password sign-in must fail cleanly; Legacy auth provider values |
| `minimal` | One prescription, one medicine | Smallest non-empty state; One active medicine with two reminders |
| `handwritten` | All 20 handwritten prescriptions (HW-01 to HW-20) | Document viewer with generated images (after attach-images); Cursive, Hindi, faded, carbon copy, two pages, missing and future dates |
| `heavy` | Worst-case volume: 25 years of records | Timeline and list pagination; Search across hundreds of records; Report PDF size; Slow devices |
| `elderly-polypharmacy` | 86-year-old on 15 active medicines | Today screen with 30+ doses a day; High-alert medicines (warfarin, insulin); Large-text layouts |
| `newborn` | 4-month-old baby (parent-managed) | Age in months, not years; Syrups and drops in ml; Vaccination schedule (later phases) |
| `sparse-profile` | No date of birth, gender or blood group | Null profile fields everywhere; Age cannot be shown |
| `odd-profile` | Unusual profile values | Gender "Other" and free text; Blood group "Unknown" and Bombay (hh); Born 29 Feb; Age 105 |
| `long-text` | Very long names and Indian scripts | Text overflow and truncation; Devanagari, Tamil, Bengali, Urdu rendering; Emoji in free text |
| `uncertain-dates` | Every kind of uncertain or missing date | Undated prescriptions; Dates flagged uncertain; Future-dated prescription; Month and year precision on the timeline; Leap days and year boundaries |
| `pipeline` | Documents in every processing state | Uploaded, extracting, stuck, ready for review, failed, cancelled; Review drafts with uncertain fields; Upload intents: fresh, expired, stuck, rejected; Duplicate upload (same SHA-256) |
| `review-states` | Prescriptions in DRAFT and DISCARDED review states | Screens must hide or label non-confirmed prescriptions; No medicines or timeline from drafts |
| `med-lifecycle` | Medicines in every status and boundary | ACTIVE, PAUSED, COMPLETED, DISCONTINUED, INACTIVE; Active without reminders; Ends today, ended yesterday, starts tomorrow; Reminders at 00:00 and 23:59; Six reminders a day; Weekly and SOS medicines (no daily reminders); User-added medicine with no prescription |
| `adherence-perfect` | Adherence: perfect | Taken on time, late, skipped, missed, snoozed; Adherence summaries |
| `adherence-poor` | Adherence: poor | Taken on time, late, skipped, missed, snoozed; Adherence summaries |
| `adherence-never-records` | Adherence: never-records | Past doses stay unrecorded (PENDING): never shown as Taken or Missed (Rule 15) |
| `tz-new-york` | Time zone America/New_York | Reminder times across time zones; 02:30 reminder skipped on the spring-forward day |
| `tz-chatham` | Time zone Pacific/Chatham | Reminder times across time zones; Offsets that are not whole hours or near midnight |
| `tz-dubai` | Time zone Asia/Dubai | Reminder times across time zones; Offsets that are not whole hours or near midnight |
| `tz-utc` | Time zone UTC | Reminder times across time zones; Offsets that are not whole hours or near midnight |
| `duplicates` | Same medicine many ways | Same generic under different brands (Dolo, Crocin, Calpol, PCM); Same brand, different strengths; Same prescription confirmed twice; Two active copies of one medicine |
| `all-routes` | Every route and dosage form | Oral, topical, IV, IM, SC, inhaled, eye, nose, ear, rectal, other; Units, puffs, drops, ml, sachets; Traditional medicines with no generic name |
| `isolation-a` | Twin account for cross-patient tests | Use one account's IDs from the other: every call must return 404 (Rule 4) |
| `isolation-b` | Twin account for cross-patient tests | Use one account's IDs from the other: every call must return 404 (Rule 4) |
| `chronic` | Diabetes and hypertension since 2000 | Long refill history; Dose changes over years; Lab and BP trends (later phases) |
| `allergy-conflicts` | Allergic to penicillin and NSAIDs, yet prescribed them | Allergy warnings against prescribed medicines (Phase 16); Resolved and refuted allergies |
| `pregnant` | Pregnant, 18 weeks | Pregnancy flag for medicine safety (later phases); Antenatal prescriptions |
| `caregiver` | Caregiver for a parent and a child | Family and caregiver links (Phase 24); Switching between patients |
| `historical` | Records from 1985 onwards | Timeline before 2000; Old brands, faded paper (HW-11) |
| `busy-today` | Crowded Today: 3 doctors in one week | Many doses due right now; Overlapping courses from different doctors; Snoozed doses in the last hours |

Bulk accounts are `user0001` to `user0060`. They have random ages (0–95), mostly Asia/Kolkata with a few abroad, a mix of chronic refills and acute visits, every adherence pattern, and some uploads stuck in review or failed.

## What the first local run found (for the redesign)

- `heavy` returns **1,519 medicines in one unpaginated response** from `GET /medications`, and 421 records from `/prescriptions/timeline`. Test list performance and paging with it.
- `unverified` cannot sign in (403 `EMAIL_NOT_VERIFIED`) even with `EMAIL_VERIFICATION_REQUIRED=false`. That is the current design: only new sign-ups skip verification.
- Medicine reminders are daily only. Weekly methotrexate, the vitamin D 60K weekly capsule and SOS medicines are seeded **active with no reminders**, because the schema cannot express weekly schedules. Phase 12/13 should cover this; it is a safety case.
- `no-profile` signs in, and every list comes back empty instead of erroring.
- `DRAFT` and `DISCARDED` prescriptions (`review-states`) are rows the current code never writes. Check that screens ignore them.
