# Phase N Brief: <short name>

**Risk:** <Low | Medium | High>. **Roadmap:** `docs/project/ROADMAP.md`, section <x>.
**Goal:** <one sentence: the outcome the owner can check when the phase ends>.

## Hard rules

1. No model opens `.env` or receives a key, cloud credential, production database access or real patient data.
2. Work goes on a `phase-N/*` branch in each repo. No commits, pushes or branch switches unless the owner asks.
3. Production stays untouched: no push to `main` that auto-deploys a migration or config change the phase has not cleared.
4. Any task that fails twice goes to the next model up, or to the owner.
5. Invariants in `CLAUDE.md` section 6 still hold. A task that touches them adds a regression test.
6. <Phase-specific rule, e.g. files or folders other agents own that must not be touched>.

## State found on <date>

- <repo>: <branch, clean or dirty, what is pending>.
- <known blocker or fact the tasks depend on>.

## Tasks

| # | Task | Who | Done when |
|---|---|---|---|
| 1 | <task> | **Owner** | <observable result> |
| 2 | <task> | **Opus** | <observable result> |
| 3 | <task> | **Sonnet** | <observable result> |
| 4 | <task> | **Haiku** | <observable result> |

Who is one of: **Owner** (a human does it), **Opus** (decisions and hard reviews), **Sonnet** (multi-file logic, security, clinical invariants), **Haiku** (mechanical edits, templates, scripts, copy).

## Screens and required states

For every screen the phase adds or changes, list each state that must render.

| Screen | Loading | Empty | Error | No connection | Success |
|---|---|---|---|---|---|
| <screen> | <what shows> | <what shows> | <what shows> | <what shows> | <what shows> |

## Tests

- Typecheck: `npm run typecheck` in each touched repo, 0 errors.
- Unit and integration: `npm test` in each touched repo, with counts recorded in Results.
- New regression tests: <list the behaviour each one proves>.
- Manual owner check: <device or browser steps, e.g. fresh install, add a medicine, confirm no past dose shows as Taken>.

## Review

Opus reviews the diffs and gate results once at the end, using `TEMPLATE-review.md`.

## Results

- Task 1: <result, date>.
- Task 2: <result, date>.
- Gates: typecheck <pass/fail>, tests <n passing, n skipped>, build <pass/fail>.
- Not run or not verified: <list anything skipped, such as CI or a device run>.
