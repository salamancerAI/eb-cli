# Status

Last updated: 2026-10-17

## Where this stands

eb-cli (v0.1.0-alpha.43) is an independent private repo at
`github.com/salamancerAI/eb-cli`. On Claw it installs, builds, passes lint and
**all 101 tests**, and `--help` loads (Node 22.23.3 via PATH prefix). Project memory and
docs (`CLAUDE.md`, `STATUS.md`, `DECISIONS.md`, `GIT_GUIDE.md`,
`docs/PROJECT_OVERVIEW.md`, `docs/ONBOARDING.md`) are written but **not yet committed**.

## Next action

Phase 6: commit "Add project memory and overview docs" and push `main` (with Node 22
on the PATH). Then a separate commit that disables auto-update / `eb update`.

## Blocked on / waiting on

- **The pre-push hook fails on Node 20** (`/usr/bin/node`, the regular shell). Decided:
  push with `PATH=/home/bb_dataops/.nvm/versions/node/v22.23.3/bin:$PATH git push`.
- Decision: the update channel points at the original author's S3 bucket, and the CLI
  **tries to auto-update in the background**. See overview §7.2. Fix planned as a
  separate commit after the baseline push.
- Written confirmation of ownership/license from the original author (§7.1).

## Known gaps

- 18 npm audit vulnerabilities (1 moderate, 17 high) in dependencies, not reviewed.
- README is missing 6 commands and has outdated release info.
- Dry-run on `users delete` / `budgetitems delete` clicks the real Delete button,
  then cancels.
- `feat/playwright-switch` exists only on Claw (unreviewed, not pushed, never pushed).
- 41 old `v*` tags exist locally and must not be pushed.
- CLAUDE.md "Out of scope" still has placeholders (left on purpose).

## Is this still worth doing?

Yes. The team already uses it daily.
