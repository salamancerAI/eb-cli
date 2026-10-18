# Onboarding: getting eb-cli

The code lives in a **private** GitHub repo: `github.com/salamancerAI/eb-cli`.
You always run `eb` on the **Chrome Remote Desktop host**, never on the
Chromebook itself.

## 1. Get the code

Pick one:

**A. GitHub access (read-only)**

1. Ask the maintainer to add your GitHub account to `salamancerAI/eb-cli` as a
   collaborator with **Read** access.
2. Accept the invite from your email or GitHub notifications.
3. Open `github.com/salamancerAI/eb-cli` → green **Code** button → **Download ZIP**.

**B. ZIP from the maintainer**
The maintainer makes a ZIP of the current version on their machine:

```
git archive --format=zip -o eb-cli.zip HEAD
```

and sends you `eb-cli.zip`. This ZIP contains only the committed code. It
never includes anyone's sessions, `node_modules/` or build output.

**Then:** upload the ZIP to the Chrome Remote Desktop host (the file transfer
button in the Chrome Remote Desktop side panel) and unzip it. The ZIP has no
`node_modules/` or `dist/`, so on the host, from inside the unzipped folder,
run (needs Node 22):

```
npm ci
npm run build
node bin/run.js --help
```

> Check your employer's rules before using a personal GitHub account
> (option A) on work machines.

## 2. Updating

When the maintainer publishes changes, **download a fresh ZIP and re-upload it**
the same way as step 1. Replace your old folder, then reinstall and rebuild.

To check which copy you have: run `node bin/run.js --version` and compare it to the top
entry in `CHANGELOG.md` (date, version, what changed).

Update only this way. Don't use any built-in self-update command.

## 3. Sessions and credentials stay with you

- `eb login` saves your e-Builder session in **your own home folder**
  (`~/.eb-cli/eb.db`) on the host, not in the eb-cli folder.
- That file lets anyone act as you in e-Builder. **Never zip it, commit it,
  email it, or share it.** Never type your password into a file or a chat.
- Keep your CSV input/output files outside the eb-cli folder, so they never end
  up in a ZIP.
- When you're done or leaving, run `eb logout --all`.
