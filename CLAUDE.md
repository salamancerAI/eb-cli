# Project context for Claude Code

## What this project is

The tool is a command line interface with many different scripts to automate heavy eBuilder tasks.

**Current phase:** Iteration and expansion
**Success looks like:** Automated tasks for eBuilder that I can use in my day to day job.

## Who's building it

Solo builder with a systems-analyst background (construction technology),
not a software engineer. Prefers things explained step by step rather than
assumed. Comfortable running commands as long as they are explained first.

When giving instructions:

- Give copy-paste commands, one step at a time, and say what each does.
- Explain what a command will change before running anything non-trivial.
- If something errors, show the exact error and the likely cause before
  trying fixes. Don't guess-and-check with git or the filesystem.

## How to work in this repo

1. **Read `STATUS.md` first.** It says what is true right now and what's next.
2. **Log decisions in `DECISIONS.md`** (date, decided, because, alternatives)
   instead of making them silently.
3. **Update `STATUS.md`** at the end of any working session: where things
   stand, next action, known gaps, blockers.
4. **Ask before:**
   - installing new dependencies
   - changing folder structure
   - running anything that touches the network, a git remote, or another machine
   - deleting or overwriting files
5. **Keep it simple.** Prefer the smallest change that works. Do not add
   frameworks, abstractions, or features that weren't asked for.
6. **Never commit secrets** (API keys, tokens, passwords). They belong in
   `.env` files that `.gitignore` already excludes.
7. **Scope discipline.** If a request drifts into "Out of scope" below, say so
   and ask before proceeding.

## Folder layout

This repo uses the standard oclif CLI layout. Do not restructure it.

- `bin/` — entry points: `run.js` (built CLI) and `dev.js` (runs TypeScript
  source directly); `.cmd` versions for Windows
- `src/commands/` — one file per CLI command; folder name = topic
  (e.g. `src/commands/users/delete.ts` → `eb users delete`)
- `src/lib/` — shared code: session base command, CSV, local database, login helper
- `src/lib/eb-puppetmaster/` — Puppeteer browser automation against e-Builder
- `src/hooks/` — oclif hooks (runs before command parsing)
- `src/test/` — tests (mirrors `commands/`, `lib/`, `puppetmaster/`)
- `scripts/` — `install.sh` installer script
- `.github/workflows/` — `ci.yml` (checks) and `release.yml` (publishing)
- `docs/` — project docs (overview, work-PC sync, notes)
- `dist/` — compiled output from the build (git-ignored, not committed)
- `README.md`, `package.json`, `tsconfig.json`, `eslint.config.js`,
  `.eslintrc.json`, `.prettierrc` — project config
- `STATUS.md`, `DECISIONS.md`, `GIT_GUIDE.md`, `CLAUDE.md` — project memory

## Constraints

- **No SSO login to e-Builder.** The tool signs in with e-Builder
  username/password sessions only.
- **Must run on Chrome Remote Desktop.** The work device is a Chromebook;
  `eb` is used through a Chrome Remote Desktop session.
- **Every NEW script/command must include a `--dry-run` mode** that shows
  what would change without changing anything in e-Builder.
- Daily tool for work (e-Builder analyst).
- $0 budget.
- Licensing: not yet resolved (no LICENSE file in the original repo).
  See `docs/PROJECT_OVERVIEW.md` → Risks.

## Out of scope right now

- [Thing you're deliberately not doing yet]
- [Thing you're deliberately not doing yet]

---

## Machines

- **Claw** — Windows 11 Home running **WSL2 Ubuntu** (user `bb_dataops`).
  Runs Claude Code and holds the development copy of this repo at
  `~/bb_spec/EB-CLI`. All development, commits and pushes happen here.
  Reached from the laptop by SSH (VS Code Remote-SSH for editing).
  Node 22 comes from nvm (`~/.nvm`); `/usr/bin/node` is an older v20.
- **Chrome Remote Desktop host** — where the team actually runs `eb`
  (reached from Chromebooks). It does **not** clone the repo: teammates
  download a ZIP from GitHub (or get one from `git archive`) and upload it
  there. Updates = re-download + re-upload. See `docs/ONBOARDING.md`.
- Saved e-Builder sessions (`~/.eb-cli/eb.db`) stay on the machine that
  created them. Never copy them between machines or into the repo.

---

## Git & GitHub

- Remote: `github.com/salamancerAI/eb-cli` (private). An independent copy of
  the original `haackr/eb-cli`, with no link to it. Not a fork, and not created
  from `project-template`.
- Everyday loop on Claw: `git add <specific files>` → `git commit -m "short description"` →
  `git push`. No blind `git add .`. Explain each step when asking me to run it.
- Never force-push, rewrite history, or `git add` ignored files without asking.
- Never `git push --tags`: the old `v*` tags would trigger `release.yml`.
- See `GIT_GUIDE.md` for the full walkthrough.

## Definition of done for a work session

- Code or changes run and were tested at least once.
- `STATUS.md` updated (where it stands, next action, known gaps).
- Non-trivial choices logged in `DECISIONS.md`.
- Changes committed (and pushed, if I asked).
