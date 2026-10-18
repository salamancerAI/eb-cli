# Git & GitHub walkthrough

Copy-paste these in order. Run them from inside the project folder (the one
this file is in) unless noted otherwise.

## First time only: turning this template into a real GitHub template

Do this once, on the template folder itself, before you start copying it for
individual projects.

1. Create a **new repository** on github.com — name it something like
   `project-template`. Leave it empty (no README, no .gitignore — you already
   have those here).
2. In the repo's **Settings** tab, check the box **"Template repository"**.
   This is what makes "Use this template" show up later — it's the whole
   point of doing this step.
3. Back in this folder on your machine, run:
   ```
   git init
   git add .
   git commit -m "Initial project template"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/project-template.git
   git push -u origin main
   ```
4. Refresh the GitHub page — you should see all these files there.

## Starting a new project from the template (this is the repeatable part)

1. On github.com, go to your `project-template` repo and click the green
   **"Use this template" → "Create a new repository"** button.
2. Name the new repo after the actual project (e.g. `pod-designs-2026`).
   Choose **Private** unless you have a reason to make it public.
3. Clone it to your machine:
   ```
   git clone https://github.com/YOUR-USERNAME/pod-designs-2026.git
   cd pod-designs-2026
   ```
4. Fill in `README.md`, `STATUS.md`, and `CLAUDE.md` for this specific
   project. Delete the example entry in `DECISIONS.md`.
5. Commit the fill-in:
   ```
   git add .
   git commit -m "Fill in project details"
   git push
   ```

## The everyday loop, once you're working

Run this whenever you've made progress you don't want to lose:

```
git add .
git commit -m "short description of what changed"
git push
```

`git add .` stages everything that changed (except what `.gitignore` excludes).
`git commit` saves a snapshot locally with a message describing it.
`git push` sends that snapshot to GitHub.

## A few things worth knowing

- **Nothing is sent to GitHub until you `git push`.** `git commit` alone is
  local only — safe to do often.
- **The `.gitignore` in this template already excludes secrets/API keys and
  the `output/` folder.** Don't fight it by force-adding a secrets file —
  if you need to track something ignored, that's a sign to check the
  `.gitignore` first, not skip it.
- **If a command errors and you're not sure why**, paste the exact error into
  a chat with Claude (or ask Claude Code directly if you're using it) —
  don't guess-and-check with git, it's easy to get into a confusing state.
- **Undo an uncommitted mistake:** `git restore <file>` reverts a file to its
  last committed version.
