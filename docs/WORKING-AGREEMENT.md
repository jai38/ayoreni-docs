# How the lead, UI and backend sessions work together

There are three Claude cloud sessions:
- The **lead session** works in this repo. It plans phases, writes briefs, reviews PRs on both sides, and keeps the roadmap and the "Now" section of `STATUS.md` current when the owner decides something. It writes no app or backend code.
- The **UI session** works in `jai38/myhealthapp`.
- The **backend session** works in `jai38/myhealthapp-backend`.

All three read and write this repo. The sessions can't message each other, so GitHub is their shared memory, and the owner carries one-line pointers between them.

## Who owns what

| | Lead session | UI session | Backend session | Owner |
|---|---|---|---|---|
| Code | none (reads both repos to review) | `myhealthapp` | `myhealthapp-backend` (not `infra/envs/`) | Terraform values, AWS, `.env` |
| In this repo | Roadmap, phase briefs, reviews, the "Now" section of `STATUS.md` (on the owner's word) | `docs/contracts/ui-needs.md`, the UI section of `STATUS.md`, UI results in briefs | `docs/contracts/api.md`, the backend section of `STATUS.md`, backend results in briefs | "Now" and "Owner decisions" in `STATUS.md`, approving briefs |
| Merges to `main` | This repo only | Only when the owner says (it deploys Pages and the APK) | Only when the owner says (it runs CI) | Runs `deploy-aws.yml` and `terraform apply` |

Briefs and reviews: the lead session writes the phase brief and the review, with every task tagged **UI**, **Backend** or **Owner**. Each session adds results only under its own tasks.

## Rules

1. **The contract comes first.** The app calls only what `docs/contracts/api.md` describes. Only the backend session changes that file, in the same PR as the code change.
2. **The UI asks; the backend answers.** When the UI needs a new endpoint or field, it:
   - adds a line to `docs/contracts/ui-needs.md`, and
   - opens an issue in `jai38/myhealthapp-backend` labelled `from-ui`, linking the screen in the design canvas.

   The backend closes the issue with a PR and updates `api.md`. The UI never edits backend code, and the backend never edits app code.
3. **Read before you start; write when you stop.** Each session reads `STATUS.md`, the phase brief and the open `from-ui` issues first. It finishes by updating its own `STATUS.md` section: what changed, the PR link, and anything the other side needs.
4. **Use the same branch name in both repos for a phase:** `phase-5/<topic>`. Open draft PRs early, so the other side and the owner can see the work.
5. **Merge order:** the backend first, then the owner deploys, then the app. A UI change that needs a new endpoint waits until that endpoint is live, or hides the feature behind a check that falls back calmly.
6. **Breaking API changes** need a new field or route, never a silent change of meaning. The old shape stays until the app no longer uses it.
7. **The owner passes messages between sessions.** When one session finishes something the other needs, the owner pastes one line into the other chat. For example: "Backend closed issue #12, see `api.md` and `STATUS.md`; wire the Profile screen."
8. **Pull this repo at the start of each session,** and commit small changes often, so the two sessions rarely touch the same lines.
9. **Keep the plan map current.** The owner's clickable map of every phase, task and subtask is https://claude.ai/artifact/BwYL4EwQyyhPkY2nkEWras. Its data is `docs/project/plan/plan.json`. When you update your `STATUS.md` section, also set the `status` of your own tasks (and their `sub` items) in that file: `done`, `active`, `waiting`, `blocked` or `todo`, with a one-line `notes` and the PR link. Change nothing else in it. The lead session owns the rest of the file (the "now" block, decisions, new phases) and republishes the map. A session without the Artifact tool only edits the JSON.

## Starter prompts

**UI session**
> You are the UI session for Ayoreni. Clone and read `jai38/ayoreni-docs`: `CLAUDE.md`, `docs/STATUS.md`, `docs/WORKING-AGREEMENT.md`, the current phase brief, and `docs/contracts/api.md`. The design source of truth is https://claude.ai/artifact/7f6T8pLUF6V4CTzCKeYmU9. Build only in `jai38/myhealthapp`, on the phase branch. If you need anything from the backend, follow rule 2. When you stop, update the UI section of `STATUS.md`.

**Backend session**
> You are the backend session for Ayoreni. Clone and read `jai38/ayoreni-docs`: `CLAUDE.md`, `docs/STATUS.md`, `docs/WORKING-AGREEMENT.md`, the current phase brief, and the open `from-ui` issues in `jai38/myhealthapp-backend`. You own `docs/contracts/api.md`. Never touch Terraform, AWS, `infra/envs/` or `.env`; deploys are the owner's. When you stop, update the backend section of `STATUS.md`.
