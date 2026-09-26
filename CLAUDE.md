# MyExpenses

<!-- Project instructions for Claude Code. Keep it short: only what the agent cannot infer from the code. -->

## Personal fork (public repo)

Upstream: https://github.com/mtotschnig/MyExpenses (GPLv3, remote `upstream`, push disabled).
Upstream releases are tags `rNNN`. This repo = upstream history + a few personal
commits kept small and rebased onto every upstream release.

Branches and channels:
- `main` = upstream + personal commits → CI publishes "My Expenses" (`org.totschnig.myexpenses`),
  installed on the phone via Obtainium. Real data.
- `dev` = main + work in progress → CI publishes "My Expenses Dev"
  (`org.totschnig.myexpenses.dev`) as a pre-release. Test data only.
- `feat/<name>` branches off `dev`. Promote with `git switch main && git merge --ff-only dev`.
- Android Studio build variant `externDebug` → `org.totschnig.myexpenses.debug` (inner loop,
  throwaway data, run from Android Studio/`./gradlew myExpenses:installExternDebug`).

Personal commits (keep small; CI rebases them onto every upstream release):
- `personal: unlock-all-features` — `LicenceHandler.kt` + `BuildConfig.UNLOCK_ALL`. The
  published source builds stock, licence-checked behaviour; only a build passing
  `-PUNLOCK=true` runs as if a Professional licence were present.
- `personal: configurable-release-channel` — `myExpenses/build.gradle` properties
  `ID_SUFFIX` / `APP_NAME`, read by CI to build Daily vs. Dev as separate installable apps.

Conventions for new personal features:
- New code goes in new files (e.g. package `org.totschnig.myexpenses.personal.*`) or a new
  module. Touch upstream files only with minimal hooks — large edits to existing files
  cause rebase conflicts forever.
- Never change `versionCode` — the app's upgrade/migration logic depends on it. CI only
  sets `versionName`.
- Respect the existing licence-gating style (`ContribFeature`) rather than bypassing it
  ad hoc in new code.

Security (this repo is public): never print, echo, cat or commit keystores (`*.jks`),
passwords, tokens or GitHub secrets; never add a CI step that echoes a secret; never commit
backups, exported CSV/QIF or database copies (real financial data); synthetic data only in
issues/commits.

CI (`.github/workflows/build.yml`): every push to `main`/`dev` builds and publishes that
channel's signed release APK; a daily cron job rebases `main` (and `dev` on top of it) onto
the newest upstream `rNNN` tag and force-pushes, opening a GitHub issue instead of guessing
on conflict. Secrets required: `KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`,
`KEY_PASSWORD`, `PUSH_TOKEN` (a fine-grained PAT with Contents+Workflows read/write, needed
because the default token can't push commits that touch workflow files).

Common tasks:
- CI failed: `gh run list --limit 5` then `gh run view --log-failed`.
- Manual build: `gh workflow run build.yml -f rebuild=dev|main|both`.
- Upstream rebase issue opened by CI: follow the commands in the issue, resolve conflicts,
  build locally (`./gradlew myExpenses:assembleExternDebug -PUNLOCK=true`), then
  `git push --force-with-lease origin main dev`.
- After CI rebased: `git fetch origin && git switch dev && git pull --rebase`.

## Commands

<!-- Build, test, lint, run. One line each. -->

## Verification

<!-- The one command that proves the project is healthy. Run it before reporting any task done, and show the output. -->

## Conventions

<!-- Naming, layering, error handling, commit style. Things a reviewer would flag. -->

## Architecture

<!-- Five sentences: entry points, main modules, where state lives, how requests flow. -->

## Workflow

`/sdlc-intent` → `/sdlc-spec` → `/sdlc-plan` → `/ai-task`; artefacts in `docs/sdlc/`. Run `/ai-init` if there is no `.ai/`.

## Things Claude Code gets wrong

<!-- Recurring mistakes and their corrections. Grow this list from code review. -->

<!-- claude-agentic:start -->
## AI agent workflow

This repository runs an agentic pipeline under `.ai/`. Read `.ai/AGENTS.md` first: it routes to the policies, workflows and rules, which load on demand; `.ai/policies/` is binding.

- Production behaviour is the source of truth: document problems outside the task, do not fix them.

- A change runs through `/ai-task <request>`; `/ai-status` shows where it stands. Each step names the files it may touch — an edit outside them is refused: answer `SCOPE_CHANGE_REQUIRED`.

- Verify before reporting done: `verify_command` from `.ai/policies/testing.md` once, to the end, then `e2e_command` once; every failure fixed as one batch; show the output.
- No agent commits, merges or deploys; approval is given by a human outside the agent. When a review flags the same mistake twice, the correction goes into this file.
- Files, not chat, carry decisions: `.ai/reports/<task-id>/questions.md` (answer by filling `[Answer]:`), `.ai/state/handoff.md` (read first when resuming), `docs/sdlc/constitution.md` (this project's principles).
<!-- claude-agentic:end -->
