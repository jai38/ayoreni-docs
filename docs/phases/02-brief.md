# Phase 2 Brief: AWS foundation

**Risk:** High. **Roadmap:** `docs/project/ROADMAP.md`, sections 6 and 13.
**Goal:** the app runs on a lean AWS environment in Mumbai, rebuildable from Terraform and able to scale by configuration.

## Decisions

- Fresh start: no data moves from Supabase.
- Extraction stays on Gemini for this version. A future version switches back to Haiku 5.5.
- 2 November 2026, owner: Gemini `gemini-2.5-flash` is approved as the extraction provider for this version, and `ai_provider_approved = true` in v1. Any change of provider or model comes as a new version.
- One lean environment, about $40-50 a month.

## Hard rules

1. No model holds AWS credentials or runs `terraform apply`. The owner applies.
2. No secret values in Terraform or the repo. Parameter Store entries are created with placeholder values that Terraform ignores afterwards, and the owner sets real values.
3. Render keeps running until the owner confirms AWS works.
4. Work on `phase-2/aws` in the backend repo. Do not push to `main` until cutover is ready, because that deploys to Render.

## Tasks

| # | Task | Who | Done when |
|---|---|---|---|
| 1 | AWS account, MFA, budget alert, domain decision | **Owner** | Account ready |
| 2 | New paid Gemini key | **Owner** | Set in `.env` and Parameter Store; old key deleted |
| 3 | Terraform: network, ECR, ECS Fargate ARM API and worker, API Gateway HTTP API with Cloud Map, RDS, S3, KMS, SQS, Parameter Store, SES identity, CloudWatch, budget alarm, OIDC deploy role | Sonnet | `terraform validate` passes; sizes are variables |
| 4 | Migrations as a deploy step, not on boot | Sonnet | Dockerfile no longer runs them; deploy runs one task first |
| 5 | Real SQS worker path, so extraction runs in the separate worker with no database access | Sonnet | Tests pass, in-process mode stays for tests |
| 6 | SES email sender replacing Resend | Haiku | Tests pass; production guard updated |
| 7 | Database connection limit per task, S3 config without a custom endpoint, optional-endpoint guard | Sonnet | Tests pass; guard still rejects local values |
| 8 | GitHub Actions deploy to ECS through OIDC, kept inactive until cutover | Sonnet | Workflow parses; old Render step removed at cutover |
| 9 | `terraform apply`, set parameters, first deploy | **Owner** | Health check passes |
| 10 | Cutover: app points at the new API, retire Render and Supabase, revoke old keys | **Owner + Opus** | Old services off |

## Results

- 2 November 2026: tasks 3-9 done. `phase-2/aws` merged to backend `main` (`c208003`, fixes in `37ce336` and `9ecb7e4`). v1 applied in `ap-south-1`; first deploy passed its health check. Test on the AWS APK (`v0.2.0-aws-preview`, mobile `bc832a4`): signup, login, dashboard, profile and upload work. Extraction waits for step 11.
- First-deploy fixes: deploy role trusts GitHub's immutable OIDC subject; the API no longer gets the AI provider settings (in queue mode only the worker calls Gemini); re-runs reuse the commit's image because ECR tags are immutable; Cloud Map uses SRV records, since API Gateway needs the port; secrets ignore `key_id`.
- AI on (README step 11): backend `e066fb3` sets `ai_provider_approved = true` and makes the worker refuse a placeholder key; mobile `812f23f` waits up to 4 minutes for the worker's cold start. Deployed and confirmed end to end by the owner: upload, extraction and draft review work on AWS. Only task 10, cutover, remains.
- Cutover code (task 10): backend `b93cf5a` removes the Render deploy job and `render.yaml`; mobile `91126bc` points the app default, web build, preview profile and CI at AWS. Owner shutdown of Render and Supabase and key revocation pending.
- Cutover done 2 November 2026: Render suspended (verified: `/healthz` returns 503 Service Suspended); owner reports Supabase paused, old keys revoked, machine `GEMINI_API_KEY` cleared and Terraform state backed up.
- Exit status: existing flows work on AWS (met). Rebuild from Terraform: passed on 2 November 2026. A clean `test` workspace applied from the committed code with 84 resources added and nothing changed; v1 stayed healthy. The copy was destroyed the same day (its API address no longer resolves). Billing alerts fire in a test: alarm path passed on 2 November 2026 (SNS subscription confirmed, `set-alarm-state` email received); budget email via a temporary $1 budget pending. Phase closes when both pass and the owner approves.
- Original build notes: tasks 3-8 built on `phase-2/aws`. 345 tests pass. `terraform fmt` and `validate` pass in the official container. Nothing has run against real AWS, and SQS, SES and S3 are tested only against fakes.
- Follow-ups: delete S3 versions explicitly on erasure before the V1 gate (now aged out after 1 day). Use a separate database app role instead of the master user. Move Terraform state to an encrypted S3 backend before real data. A failed hand-off to SQS leaves the job queued until the client retries; fix with the outbox in Phase 7.

- The key Google blocked as leaked is the `GEMINI_API_KEY` in this machine's environment, not the `.env` key. Environment values override `.env`, so run local tools with it unset until the owner removes it. Earlier statements blaming the `.env` and Render key were wrong.
- Handwriting test, `gemini-2.5-flash`, 3 public samples, 10-17 seconds each: all medicines, frequencies and durations correct. Errors, all marked confident: `typical.jpg` date read as 2020-01-31 where the form appears to show 31/12/20 (partly hidden), and the 1924 doctor's name misread. Too small a sample to claim accuracy, with no Indian prescriptions. Phase 9 must require user confirmation of dates and names, and flag low-legibility fields.
- Provider now sends the key in a header, not the URL.
- Handwriting test samples, with what each should return, are kept outside the repo. Expected for `typical.jpg`: Calcium Sandoz, one daily, 10 days; Calcium + Vit D 800 IU, daily, 3 months. Expected for `whiskey1924.jpg`: whiskey, as directed, dated 1924.
