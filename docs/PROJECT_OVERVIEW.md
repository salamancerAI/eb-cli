# eb-cli — Project Overview

Written 2026-10-17 from a read-only review of the code at version
`0.1.0-alpha.43` (last upstream commit 2026-08-28). Nothing was changed while
writing this.

## What the tool is, in one paragraph

`eb` is a command-line tool that drives a real (usually invisible, "headless")
Chrome browser to click through the e-Builder website for you. You give it a
CSV file ("delete these 200 users", "set these budget item flags"), and it
logs in with a saved session and repeats the clicks for each row. It is built
with **oclif** (a framework for CLI tools), **TypeScript** (JavaScript with
type checking, compiled to plain JavaScript in `dist/`), and **Puppeteer** (a
library that remote-controls Chrome).

---

## 1. Command inventory

Every command lives in its own file under `src/commands/`. The folder is the
"topic" and the file is the command: `src/commands/users/delete.ts` → `eb users delete`.

| Command                                      | File                                        | What it does                                                                                                                                         | Changes e-Builder? | `--dry-run`? |
| -------------------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | ------------ |
| `eb login` (alias `eb session create`)       | `src/commands/login.ts`                     | Log in (username + password via Trimble ID) and save the session locally. Also has `--sso-url` for SSO login — **not used by us** (see Constraints). | No                 | n/a          |
| `eb logout` (alias `eb session delete`)      | `src/commands/logout.ts`                    | Log out one session, all of a user's sessions, or `--all`; deletes them from the local store.                                                        | No                 | n/a          |
| `eb session list`                            | `src/commands/session/list.ts`              | List saved sessions (no browser).                                                                                                                    | No                 | n/a          |
| `eb session test` (alias `eb session clean`) | `src/commands/session/test.ts`              | Check each saved session, refresh valid ones, delete expired ones.                                                                                   | No                 | n/a          |
| `eb apilogs download`                        | `src/commands/apilogs/download.ts`          | Download the API logs table (N pages) to a `.jsonl` file.                                                                                            | No (read-only)     | n/a          |
| `eb users sso-report`                        | `src/commands/users/sso-report.ts`          | Write a CSV showing whether SSO is required for every user.                                                                                          | No (read-only)     | n/a          |
| `eb budgetitems delete FILE`                 | `src/commands/budgetitems/delete.ts`        | Delete budget items listed in a CSV.                                                                                                                 | **Yes**            | Yes          |
| `eb budgetitems set FILE`                    | `src/commands/budgetitems/set.ts`           | Set budget item properties (allow charges, approval required, description).                                                                          | **Yes**            | Yes          |
| `eb process setnullfield FILE`               | `src/commands/process/setnullfield.ts`      | Blank out one or more fields on process instances.                                                                                                   | **Yes**            | Yes          |
| `eb users delete FILE`                       | `src/commands/users/delete.ts`              | Delete users from the account.                                                                                                                       | **Yes**            | Yes          |
| `eb users remove-project FILE`               | `src/commands/users/remove-project.ts`      | Remove users from projects.                                                                                                                          | **Yes**            | Yes          |
| `eb users remove-project-role FILE`          | `src/commands/users/remove-project-role.ts` | Remove users from project roles.                                                                                                                     | **Yes**            | Yes          |
| `eb users setfield FILE`                     | `src/commands/users/setfield.ts`            | Set built-in or custom user fields (CSV column name = field label).                                                                                  | **Yes**            | Yes          |
| `eb users setsso FILE`                       | `src/commands/users/setsso.ts`              | Turn the "SSO required" flag on/off per user. (This is an admin setting on users — it does not make _us_ log in with SSO.)                           | **Yes**            | Yes          |
| `eb update [CHANNEL]`                        | from `@oclif/plugin-update`                 | Self-update the installed CLI from an S3 bucket. **Do not use — see Risks.**                                                                         | No                 | n/a          |

Common flags on every e-Builder batch command (from `src/lib/base-session-command.ts:17-27`):
`-i/--session-id`, `-u/--username`, `-s/--show-browser`. Batch commands also
have `--dry-run`, `-v/--verbose`, `-o/--output-csv` (results file) and `--json`.

CSV column names are forgiving: `User Name`, `user_name` and `userName` all
match (aliases in `src/lib/csv.ts`).

**Docs gap:** the README only documents 12 commands. It is missing
`process setnullfield`, `users setfield`, `users setsso`, `users sso-report`,
`users remove-project`, and `users remove-project-role`.

---

## 2. How sessions and credentials are stored

- **Where:** a small SQLite database file at **`~/.eb-cli/eb.db`** — in your
  home folder, **outside the repo** (`src/lib/db.ts:25-39`). The location can be
  changed with the `EB_CLI_DB_PATH` environment variable.
- **What:** one row per session: username, environment (us1…ca), account name,
  and the **browser login cookies** as JSON text (`src/lib/db.ts:58-65`).
- **Passwords are not saved.** They're typed at a hidden prompt during `eb login`
  and used once. (Avoid `eb login -p <password>`: that would put the password in
  your shell history.)
- **The cookies are not encrypted.** Anyone who copies `eb.db` can act as you in
  e-Builder until the session expires. Treat that file like a password. Never
  copy it between machines, into the repo, or into chat.
- **Is it kept out of git?** Yes, twice over: it lives outside the repo folder,
  and `.gitignore` also has `*.db`.

**Gap in `.gitignore` (not changed yet; to decide together):** commands read and
write **CSV and JSONL files containing real user and project data**. If you run
`eb` from inside the repo folder, those files land in the repo folder. Only
`test*.csv` is ignored today, so an `eb users delete users.csv -o results.csv`
run here would leave `users.csv` / `results.csv` / `api-logs-*.jsonl` that a
careless `git add .` would commit. Proposed additions:

```
# e-Builder data files (inputs/outputs) — never commit
*.csv
*.jsonl
# secrets
.env.*
*.key
*.pem
secrets*
credentials.json
```

(Simpler alternative: always run `eb` from a separate work folder, never from
the repo.)

---

## 3. How `--dry-run` works (the pattern to copy)

Two layers:

**A. The command file** (e.g. `src/commands/users/delete.ts`)

1. Declares the flag (`:49`):
   `'dry-run': Flags.boolean({ description: 'Dry run (no actual deletion)' }),`
2. Passes it down to the browser function for each CSV row (`:197-204`):
   `await eb.deleteUser({ env, cookies, browser, page, user, dryRun: flags['dry-run'] });`
3. Reports it in the summary and `--json` output (`dryRun: Boolean(flags['dry-run'])`).

**B. The browser function** in `src/lib/eb-puppetmaster/` (e.g. `user.ts:421-481`)

- Accepts `dryRun` in its options (`dryRun = false` by default).
- Does **all the navigation and lookups for real**, then **skips only the final
  "commit" click** (Confirm / Save / Remove), wrapped in `if (!dryRun) { … }`.

So dry-run means a **rehearsal on the live site**: it logs in, finds each
record, and proves the selectors work, but doesn't make the change.

**⚠️ Important nuance, which the new-command pattern should improve on:**

- `users delete` dry-run **clicks the real Delete button**, then clicks
  **Cancel** on the confirmation dialog (`user.ts:470-481`).
- `budgetitems delete` dry-run **clicks the real Delete button** and then just
  doesn't click Confirm (`budgetitem.ts:182-201`).
- Neither should change anything, but they rely on e-Builder's confirmation
  dialog being there. If e-Builder ever removed that dialog, a "dry run" would
  delete for real.
- The safer rule for **new** commands: in dry-run mode, never click any button
  that starts a change. Stop before it and log "would delete X".

---

## 4. How to add a new command (plain-language walkthrough)

Say you want `eb projects archive FILE`.

1. **Make the command file:** `src/commands/projects/archive.ts`. The folder name
   (`projects`) becomes the topic; the file name (`archive`) becomes the command.
   oclif finds it automatically. No registration step.
2. **Start from a copy of the closest existing command**, e.g.
   `src/commands/users/remove-project.ts`, the shortest CSV batch command.
   It already has the session handling, CSV parsing, progress bar, Ctrl+C handling,
   results CSV and `--json`.
3. **Keep these parts unchanged:**
   - `extends BaseSessionCommand` and `...BaseSessionCommand.baseFlags`: these
     give you `--session-id/--username/--show-browser` and the login fallback.
   - `'dry-run': Flags.boolean(...)`. **This is required for every new command
     (project constraint).**
   - `this.getSession(flags)` and `this.getSessionCookies(...)`
   - The cookie refresh every 10 rows, and the `finally { closeBrowser() }`.
4. **Put the browser clicking in a new function** in
   `src/lib/eb-puppetmaster/<something>.ts` that takes
   `{ env, cookies, browser, page, ..., dryRun }`. Export it from
   `src/lib/eb-puppetmaster/index.ts`. In dry-run, stop **before** any button that
   changes data (see the nuance in section 3).
5. **Add CSV column aliases** to `src/lib/csv.ts` if the CSV has new columns.
6. **Add a test** under `src/test/commands/` and/or `src/test/puppetmaster/`,
   copying a neighbouring test file.
7. **Check it:** `npm run build`, `npm run lint`, `npm test`, then
   `./bin/run.js projects archive --help`.
8. **Try it for real only with `--dry-run` first**, on a 1–2 row CSV.
9. **Update the README** commands section (or regenerate it with `npx oclif readme`).

Code conventions (from `.github/copilot-instructions.md`): relative imports end in `.js`
even in `.ts` files; use `import type` for types-only imports; strict TypeScript.

---

## 5. Build, lint, tests, git hooks

- `npm run build`: compile `src/` → `dist/` (`tsc`)
- `npm run lint`: ESLint over `src/`
- `npm test`: build, then run the Node test runner over `dist/test/**`. There are 18 test files.
  (`.github/copilot-instructions.md:14` says there are no tests. That line is out of date.)
- `npm run format:check`: Prettier formatting check
- `npm run dev -- <command>`: run straight from TypeScript source, no build needed
- **Git hooks:** `npm ci` runs `prepare` → `simple-git-hooks`, which installs
  hooks into `.git/hooks`. After that, **every `git commit` runs lint-staged**
  (ESLint plus Prettier, which may reformat staged files), and **every `git push` runs
  the full `npm test`** (`package.json:100-103`). If tests fail, the push is
  blocked. This is good, but surprising if you don't know about it.
- **Measured on Claw (2026-10-17, Node 22.23.3):** 101 tests, all pass, about 4.3 minutes.
  So every `git push` takes about 4–5 minutes.
- **Under Node 20 the pre-push hook fails:** `Could not find
'/home/bb_dataops/bb_spec/EB-CLI/dist/test/*.test.js'`. Node 20's test runner
  doesn't expand `*` patterns; that arrived in Node 21+. The pre-commit hook (ESLint +
  Prettier) works fine on Node 20.
- **Node version:** `package.json` pins Node **22.14.0** (`engines`, `:21-23`, and
  `oclif.node`, `:56`). Claw currently has v20.20.2.

## 6. Chrome / Puppeteer on Linux

- Puppeteer downloads its own Chrome into `~/.cache/puppeteer` the first time a
  browser-backed command runs, or during `npm ci` via Puppeteer's install script.
  That's roughly 150–200 MB.
- If Chrome won't start, `browser-manager.ts:185-215` retries after installing
  Chrome. It installs **system libraries** only if you're root or have set
  `EB_PUPPETEER_INSTALL_DEPS=1`. Otherwise you install them yourself with `apt`.
- The browser always launches with `--no-sandbox` (`browser-manager.ts:151`), which
  is common in WSL and containers.
- The `--sso-url` login path uses `channel: 'chrome'`, meaning a system-installed
  Google Chrome rather than Puppeteer's copy (`browser-manager.ts:166`). We don't use SSO, so this
  doesn't matter to us.

---

## 7. Risks

### 7.1 License: **unresolved**

- There is **no LICENSE file** in the repo.
- `package.json:20` says `"license": "ISC"`. That's npm's default value, which
  `npm init` fills in automatically, so it isn't a real license decision.
- Default copyright law applies, so the author (Ryan Haack) holds the rights unless an agreement says otherwise.
  You've said the colleague handed it over with permission. **Get that permission
  in writing** (an email is enough): a transfer to the company, or a license
  grant. Then add a LICENSE file (or a "Proprietary — [Company]" notice) and fix
  `package.json:20`.
- `package.json:40` has macOS identifier `com.avicado.eb-cli`. Is "avicado" your
  company, or the author's? That decides whether to keep it.

### 7.2 Updates come from the original author's S3 bucket: **highest-priority risk**

- `eb update`, and oclif's background auto-update in installed (tarball/installer)
  builds, download new versions from
  **`eb-cli-updates-934677684919-us-east-2-an`**, an AWS S3 bucket in AWS account
  `934677684919` that belongs to the original author's setup (`package.json:57-66`).
- So **anyone on your team who installed `eb` with the installer or a native
  package will get whatever is published to that bucket**: code you didn't
  write and can't review, running with their e-Builder sessions. That's
  fine while you trust it, but now that the tool is your company's, the update
  channel should be yours or switched off.
- **Confirmed on Claw (2026-10-17):** running only `./bin/run.js --help` printed
  `[ENOENT] Error: spawn eb ENOENT`. The plugin's startup hook
  (`node_modules/@oclif/plugin-update/dist/hooks/init.js:80`) tries to launch
  `eb update --autoupdate` **in the background on ordinary commands**, at most once
  per debounce window (1 or 14 days). It failed here only because `eb` isn't on the
  PATH. On a machine where `eb` _is_ on the PATH, it will try to auto-update from the
  author's bucket without being asked. Whether that succeeds for a ZIP/npm-built copy
  is untested. The plugin honours `EB_DISABLE_AUTOUPDATE=1` (`init.js:18`) as an
  off switch. Flagged only; nothing changed.
- Until we decide: **update only by re-downloading the repo. Never `eb update`.**
- Options to decide later: (a) remove `@oclif/plugin-update` and the `update` block
  so updates are only `git pull`. That's simplest and $0. (b) Point to a bucket your company owns.

### 7.3 The release workflow (`.github/workflows/release.yml`)

- It runs when a **tag starting with `v` is pushed** (`release.yml:3-6`). The README's
  "runs when a release is published" (`README.md:28`) is out of date.
- It doesn't hard-code the bucket. It reads repo variable `OCLIF_S3_BUCKET` and
  secrets `AWS_ACCESS_KEY_ID/SECRET` (`release.yml:41,72-73,99,152-153`). Your repo has
  none of these, so it would **fail at "Validate required config"** and publish
  nothing to S3. But the `create-release` job (`:12-22`) needs no secrets, so it **would**
  create a GitHub Release on your repo.
- **There are 41 version tags (`v0.1.0-alpha.*`) in the local clone that have not
  been pushed.** Don't run `git push --tags` or `git push --follow-tags`: it could start
  dozens of release runs at once.
- The `oclif upload` step reads the bucket name from `package.json` (the haackr
  bucket). If someone ever set AWS secrets on your repo, it would try to upload
  there and fail without that account's credentials.
- `ci.yml` runs build, tests, lint and format check on every push to `main`. That's
  harmless and useful, and it most likely already ran once on the first push.

### 7.4 Every reference to the original author's accounts/infrastructure

| File:line                                     | What                                                                                    | Notes                                                        |
| --------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `package.json:59`                             | `"bucket": "eb-cli-updates-934677684919-us-east-2-an"`                                  | Author's S3 bucket (update source)                           |
| `package.json:60`                             | `"host": "https://eb-cli-updates-934677684919-us-east-2-an.s3.us-east-2.amazonaws.com"` | Same                                                         |
| `package.json:57-66`                          | whole `oclif.update.s3` block                                                           | Drives `eb update`                                           |
| `package.json:33`                             | `"@oclif/plugin-update"`                                                                | The plugin that does the updating                            |
| `package.json:40`                             | `"identifier": "com.avicado.eb-cli"`                                                    | Whose domain is avicado?                                     |
| `scripts/install.sh:6-7`                      | `curl … eb-cli-updates-934677684919… /install.sh \| bash`                               | Install instructions point to author's bucket                |
| `scripts/install.sh:10`                       | `S3_HOST="https://eb-cli-updates-934677684919-…"`                                       | Installer downloads from author's bucket                     |
| `.github/workflows/release.yml:41,99,165,167` | `OCLIF_S3_BUCKET` var / `aws s3 cp … s3://$OCLIF_S3_BUCKET/…`                           | Not hard-coded, but uploads to whatever bucket is configured |
| `.github/workflows/release.yml:72-73,152-153` | `secrets.AWS_ACCESS_KEY_ID/SECRET`                                                      | Needs AWS keys we don't have                                 |
| `README.md:21-52`                             | "Release Automation" section (S3, channels)                                             | Describes author's release setup                             |
| `README.md:81`                                | default path `/Users/ryan/dev/eb-cli/…`                                                 | Author's Mac path baked into generated docs                  |
| `README.md:6`                                 | `npm install -g eb-cli`                                                                 | Generic oclif text; we install from git, not npm             |
| git history (all 148 commits)                 | author `Ryan Haack <ryan@haackr.com>`                                                   | Kept on purpose, since it documents how the tool was built   |

**What is NOT there (you asked me to check):** the word `haackr` appears **nowhere
in the files**. `package.json` has **no** `repository`, `bugs` or `homepage` fields,
and the README has no GitHub links to haackr. The only `haackr` is the commit
author email in git history.

### 7.5 Constraint conflicts and other technical risks

- **SSO:** `eb login --sso-url` exists (`src/commands/login.ts:27-31`,
  `auth.ts:332+`). Your constraint is no SSO, so just don't use that flag. We could
  remove or hide it later if you want.
- **Chrome Remote Desktop:** commands run headless by default, so nothing needs
  a screen. `--show-browser` opens a visible Chrome window. That works inside a
  CRD desktop session, but not over plain SSH.
- **Dry-run on the delete commands clicks the real Delete button** (section 3).
- **Unmerged `feat/playwright-switch` branch** (local only): one commit, 35 files,
  that swaps Puppeteer for Playwright. Upstream never finished it. Review it before
  using it.
- **Screen-scraping is brittle.** The automation depends on e-Builder's page
  structure (element IDs like `#ctl00_ctl00_ContentPlaceHolder1_…`). An e-Builder UI
  update can break commands without warning. Always dry-run first after an
  e-Builder release.
- **The login flow depends on Trimble ID pages** (`auth.ts:14-18`). If your account
  has MFA, headless login may need `--show-browser`.
- **Node pinned to 22.14.0**, and Claw has v20.
- **The package is marked alpha** (`0.1.0-alpha.43`).
