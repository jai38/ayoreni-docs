# Phase N Review: <short name>

**Reviewer:** <Opus or owner>. **Date:** <date>. **Brief:** `docs/phases/<NN>-brief.md`.
**Branches reviewed:** `myhealthapp` <branch @ commit>, `myhealthapp-backend` <branch @ commit>, `landing` <branch @ commit>.

## 1. Brief match

- [ ] Every task in the brief's table has a result in its Results section.
- [ ] Each task's "done when" condition is shown to hold, with evidence.
- [ ] Work is on `phase-N/*` branches; nothing merged to `main` unless the brief says so.
- [ ] Nothing outside the brief's scope changed. List any extra files: <none or list>.
- [ ] Files other agents own (for example `test/` or mobile components, if the brief excludes them) were not touched.

## 2. Rules (`CLAUDE.md`, Non-negotiable rules)

Tick each rule from 1 to 15 that the phase could affect, and mark the rest "n/a".

- [ ] AI tools and secrets: 1-2
- [ ] Ownership and security: 3-9
- [ ] Medical data: 10-15

Notes on any invariant that failed or changed: <none or detail>.

## 3. Gate results

| Gate | Command | Result | Notes |
|---|---|---|---|
| Mobile typecheck | `npm run typecheck` in `myhealthapp` | <pass/fail, error count> | |
| Mobile tests | `npm test` in `myhealthapp` | <n passing, n skipped> | |
| Backend typecheck | `npm run typecheck` in `myhealthapp-backend` | <pass/fail, error count> | |
| Backend tests | `npm test` in `myhealthapp-backend` | <n passing, n skipped> | Note any skipped integration suite |
| Backend build | `npm run build` in `myhealthapp-backend` | <pass/fail> | |
| CI | GitHub Actions on the pushed branch | <pass/fail/not run> | |
| Migrations | `prisma migrate status` against local Docker only | <result> | Never against production |
| Owner demo | Steps listed in the brief | <done/not done> | |

## 4. Findings

| # | Severity | Where | Finding | Owner or model to fix | Status |
|---|---|---|---|---|---|
| 1 | <High/Medium/Low> | <file:line> | <what is wrong and why it matters> | <who> | <open/fixed> |

## 5. Verdict

**<Pass | Pass with follow-ups | Fail>.**

- Blocking items: <none or list>.
- Follow-ups carried to the next phase: <none or list>.
- Not verified in this review: <list, such as CI or a device run>.
