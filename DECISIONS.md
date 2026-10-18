# Decisions log

Append-only. When you make a call on something you could have done a
different way, add an entry.

Format:

## YYYY-MM-DD — [short title of the decision]

**Decided:** what you chose
**Because:** the reasoning
**Alternatives considered:** what else was on the table, if relevant

---

## 2026-10-17 — Independent copy, not a fork

**Decided:** cloned `github.com/haackr/eb-cli` once, removed the `origin` remote, and
created a new private repo `salamancerAI/eb-cli` with `gh repo create --source=. --push`.
**Because:** the tool is now the company's; we want no ongoing link to the
original author's repo.
**Alternatives considered:** a GitHub fork, rejected because it keeps a visible link and
can't be made fully independent.

## 2026-10-17 — Keep the full commit history

**Decided:** pushed all 148 original commits unchanged.
**Because:** the history documents how the tool was built.
**Alternatives considered:** squashing to a single fresh commit.

## 2026-10-17 — Preserve the unmerged Playwright branch locally

**Decided:** created local branch `feat/playwright-switch` from the original repo's
branch before removing the remote. Not pushed.
**Because:** Playwright may run better on Chrome Remote Desktop; it was unreachable
once the remote was gone.
**Alternatives considered:** discard it.

## 2026-10-17 — Repo cloned directly into ~/bb_spec/EB-CLI

**Decided:** cloned into the existing empty folder instead of a nested `eb-cli/` subfolder.
**Because:** avoids an `EB-CLI/eb-cli` double folder.

## 2026-10-17 — Project memory added without restructuring

**Decided:** copied only STATUS.md, DECISIONS.md, GIT_GUIDE.md and `docs/` from
`salamancerAI/project-template`. Skipped its README.md, .gitignore, CLAUDE.md, src/, assets/
and output/. CLAUDE.md comes from `eb_cli.txt`, with Constraints and Folder layout
updated to match the real oclif layout.
**Because:** don't overwrite eb-cli files or change the folder structure.

## 2026-10-17 — Hard constraints for the tool

**Decided:** (1) no SSO login to e-Builder; (2) must run on Chrome Remote Desktop;
(3) every NEW command must include `--dry-run`.
**Because:** work device is a Chromebook; owner's requirements.

## 2026-10-17 — Update only via git pull (temporary)

**Decided:** until further notice, update `eb` only by `git pull` + rebuild. Never
`eb update`.
**Because:** `eb update` downloads from the original author's S3 bucket
(`package.json:57-66`), which we don't control.
**Alternatives considered (to decide):** remove `@oclif/plugin-update`, or point it at a
company-owned bucket.

## 2026-10-17 — No original-author references changed yet

**Decided:** documented every reference (see `docs/PROJECT_OVERVIEW.md` §7.4) but
changed none.
**Because:** owner wants to decide together what to repoint.

## 2026-10-17 — Use existing Node 22.23.3 via a PATH prefix (not nvm use)

**Decided:** run project commands with
`PATH=/home/bb_dataops/.nvm/versions/node/v22.23.3/bin:$PATH`, for this session only.
**Because:** `nvm use` refuses to run while `~/.npmrc` has `prefix=~/.npm-global`, and
`nvm use --delete-prefix` would remove that line (it's needed for global tools like
Claude Code). The project pins 22.14.0, but 22.23.3 works (one EBADENGINE warning) and
matches CI's "latest 22".
**Alternatives considered:** `nvm install 22.14.0`; `--delete-prefix` (rejected).

## 2026-10-17 — Distribution by ZIP, not git clone (replaces the work-machine sync plan)

**Decided:** teammates get the code as a ZIP (GitHub "Download ZIP" as a read-only
collaborator, or a `git archive` ZIP from the maintainer), upload it to the Chrome
Remote Desktop host, and update by re-downloading. No installers, no git on the host.
`docs/ONBOARDING.md` replaces the planned `docs/WORK_PC_SYNC.md`.
**Because:** simplest for the team; the repo is private.
**Alternatives considered:** a pull-only git clone on the host; oclif installers / `eb update`.

## 2026-10-17 — Format check left failing until commit (option B)

**Decided:** didn't run Prettier on the four new Markdown files during Phase 5.
**Because:** formatting rewrites files, which needs explicit approval; the choice wasn't
made yet.

## 2026-10-17 — Push with Node 22 on the PATH (hooks)

**Decided:** run `git push` as
`PATH=/home/bb_dataops/.nvm/versions/node/v22.23.3/bin:$PATH git push …`. No `~/.bashrc`
edit, no `--no-verify`.
**Because:** the pre-push hook (`npm test`) fails on Node 20 (test file patterns aren't
expanded); this needs no system change.
**Alternatives considered:** PATH line in `~/.bashrc`; apt upgrade to Node 22;
`--no-verify` (rejected).

## 2026-10-17 — .gitignore blocks data and secret files

**Decided:** added `*.csv`, `*.jsonl`, `.env.*`, `*.key`, `*.pem`, `secrets*`,
`credentials.json`. No tracked CSV/JSONL fixtures existed, so no exceptions were needed;
`.env` was already ignored.
**Because:** commands read and write real e-Builder data files.

## 2026-10-17 — CHANGELOG.md added; version unchanged

**Decided:** created `CHANGELOG.md` with one entry for 0.1.0-alpha.43; `package.json`
version not changed.
**Because:** lets teammates match their ZIP copy to a version.

## 2026-10-17 — No npm audit fix; Playwright branch and tags stay local

**Decided:** don't run `npm audit fix`; push only `main`; keep
`feat/playwright-switch` and all `v*` tags local.

## 2026-10-17 — Disable background auto-update (separate commit)

**Decided:** disable the background `eb update --autoupdate` (and make `eb update`
unusable) in a SEPARATE commit after the baseline push. The commit will be named here
once it exists.
**Because:** it checks the original author's S3 bucket
(`eb-cli-updates-934677684919-us-east-2-an`), and the `EB_DISABLE_AUTOUPDATE` environment
variable can't be relied on for teammates (each person would have to set it on their
host).
**Alternatives considered:** asking teammates to set `EB_DISABLE_AUTOUPDATE=1`; pointing
updates at a company-owned bucket.

## 2026-10-17 — Supersedes two earlier entries

**Decided:**

- "Update only via git pull (temporary)" now applies to the **maintainer only** (Claw).
  Teammates update by re-downloading the ZIP (see "Distribution by ZIP, not git clone").
- "Format check left failing until commit (option B)" is superseded: Prettier was run
  on all five new Markdown files (`CLAUDE.md`, `DECISIONS.md`, `STATUS.md`,
  `docs/PROJECT_OVERVIEW.md`, `docs/ONBOARDING.md`) before the commit.

## 2026-10-17 — Playwright branch: corrected rationale

**Decided:** correct the reason given in "Preserve the unmerged Playwright branch
locally". The original author's commit message only says "better compatibility".
There is **no evidence yet** that Playwright helps on Chrome Remote Desktop. The branch
is untested and is kept only so the work isn't lost.

## 2026-10-17 — Auto-update and `eb update` disabled

**Decided:** in the commit "Disable oclif auto-update and eb update command", changed
`package.json` `oclif.plugins` from `["@oclif/plugin-update"]` to `[]`. oclif no
longer loads the plugin, so its background auto-update hook never runs and
`eb update` gives "command update not found".
**Kept unchanged on purpose:** the `@oclif/plugin-update` dependency (no lockfile change),
the S3 `oclif.update` block, `scripts/install.sh`, and the inert preparse hook
`src/hooks/preparse/default-update-channel.ts`. These are flagged in
`docs/PROJECT_OVERVIEW.md` §7.4 for a later decision.
**Because:** smallest, easily reversible change that stops any contact with the original
author's S3 bucket. Teammates update by re-downloading the ZIP.
**Alternatives considered:** uninstalling the dependency; `EB_DISABLE_AUTOUPDATE=1`
(can't be relied on for teammates); a company-owned bucket.
