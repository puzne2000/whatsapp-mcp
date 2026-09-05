# Working with Git (Fork Workflow)

This repo is cloned from [`verygoodplugins/whatsapp-mcp`](https://github.com/verygoodplugins/whatsapp-mcp),
which you don't own. Changes go through your own fork and a pull request back
to the original repo. See [`AGENTS.md`](../AGENTS.md) for PR rules, scope
(`ROADMAP.md`), and CI requirements — this doc only covers the git mechanics.

Remote naming convention used throughout:
- **`origin`** → your fork (you can push here)
- **`upstream`** → the original repo (read-only for you)

## 1) Clone and fork the upstream repository

If you already cloned the upstream repo directly (so `origin` currently
points at `verygoodplugins/whatsapp-mcp`):

```bash
# 1. Fork the repo on GitHub (creates <you>/whatsapp-mcp)
gh repo fork verygoodplugins/whatsapp-mcp

# 2. Re-point your local remotes
git remote rename origin upstream
git remote add origin https://github.com/<your-username>/whatsapp-mcp.git

# 3. Verify
git remote -v
```

If you're starting fresh (no local clone yet), fork first and clone your fork
directly:

```bash
gh repo fork verygoodplugins/whatsapp-mcp --clone
cd whatsapp-mcp
git remote add upstream https://github.com/verygoodplugins/whatsapp-mcp.git
```

## 2) Keep `main` in sync with upstream

Do this before starting any new feature branch, so you're building on top of
the latest upstream code:

```bash
git fetch upstream
git checkout main
git merge upstream/main    # or: git rebase upstream/main
git push origin main
```

- `git fetch upstream` downloads upstream's latest commits into
  `upstream/main` without touching your working files.
- `git merge`/`git rebase` brings those commits into your local `main`.
- `git push origin main` updates your fork on GitHub to match.

## 3) Develop a new feature

```bash
# Branch off an up-to-date main
git checkout main
git checkout -b feat/short-description

# ... make changes, following AGENTS.md (scope, tests, docs) ...

git add <files>
git commit -m "feat: short description"

# Repeat commits as needed while developing
git add <files>
git commit -m "fix: address review feedback"

# Push the branch to your fork (not upstream — you can't push there)
git push -u origin feat/short-description
```

Use a [conventional commit](https://www.conventionalcommits.org/) prefix
matching the PR type: `feat:`, `fix:`, `chore:`, `docs:`, `ci:`, `refactor:`,
`test:`, `perf:`. Run the repo's lint/test commands (see `AGENTS.md` → Local
commands) before pushing.

## 4) Open a PR and get the feature into upstream

```bash
gh pr create --repo verygoodplugins/whatsapp-mcp \
  --title "feat: short description" \
  --body "Closes #<issue-number>

<what changed and why>"
```

This opens a PR from `<your-username>:feat/short-description` into
`verygoodplugins/whatsapp-mcp:main`. From here:

- CI (lint, tests, CodeQL, etc. — see `AGENTS.md` → CI gates) runs automatically.
- If maintainers request changes, keep committing to the same branch and
  push again — the PR updates automatically:
  ```bash
  git add <files>
  git commit -m "fix: address review comment"
  git push origin feat/short-description
  ```
- Once approved and merged upstream, your feature is in
  `verygoodplugins/whatsapp-mcp:main`.

## 5) Before opening the next feature branch

Clean up and re-sync so the next branch starts from a fresh, current `main`:

```bash
# Switch back to main
git checkout main

# Sync with upstream again (repeat step 2 above)
git fetch upstream
git merge upstream/main
git push origin main

# Delete the merged feature branch (local and remote)
git branch -d feat/short-description
git push origin --delete feat/short-description
```

Then start the next feature from step 3, branching off the freshly synced
`main`.
