# Status

Last updated: 2026-10-17

## Where this stands

eb-cli (v0.1.0-alpha.43) is an independent private repo at
`github.com/salamancerAI/eb-cli`. On Claw it installs, builds, passes lint and
**all 101 tests**, and `--help` loads (Node 22.23.3 via PATH prefix). Project docs are
committed and pushed (`e379363`). Background auto-update and `eb update` are disabled
(commit "Disable oclif auto-update and eb update command").

## Next action

Get written ownership/license confirmation, then do the README pass (see Known gaps).

## Blocked on / waiting on

- Written confirmation of ownership/license from the original author
  (`docs/PROJECT_OVERVIEW.md` §7.1).
- Decision: what to do with the S3 `update` block (`package.json:55-64`) and
  `scripts/install.sh`. Both still point at the original author's bucket and are now unused.

## Rules to remember

- **Push with Node 22:**
  `PATH=/home/bb_dataops/.nvm/versions/node/v22.23.3/bin:$PATH git push`. The pre-push
  hook (`npm test`, about 5 minutes) fails on Node 20.
- **Do not publish a GitHub Release (or push any `v*` tag) until `release.yml` is
  reviewed.**
- Push only `main`. `feat/playwright-switch` and the 41 old `v*` tags stay local.

## Known gaps

- **README pass (later):** it still documents `eb update`, shows the original author's
  local folder path (`/Users/ryan/dev/eb-cli/…`) as a default value (`README.md:81`),
  and its install instructions (`npm install -g eb-cli`, S3 release automation) don't
  match our ZIP distribution (`docs/ONBOARDING.md`). It's also missing 6 commands.
- 18 npm audit vulnerabilities (1 moderate, 17 high) in dependencies, not reviewed.
  Don't run `npm audit fix` without a decision.
- Dry-run on `users delete` / `budgetitems delete` clicks the real Delete button,
  then cancels.
- `feat/playwright-switch` is unreviewed and untested.
- CLAUDE.md "Out of scope" still has placeholders (left on purpose).

## Is this still worth doing?

Yes. The team already uses it daily.
