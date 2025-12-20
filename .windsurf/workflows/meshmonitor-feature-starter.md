---
name: meshmonitor-feature-starter
description: Keep a fork synchronized with upstream/main, preserve .windsurf customizations, and branch off for new feature work.
tools: run_command, read, write
model: sonnet
---

You are preparing a fresh feature branch that starts from the latest upstream `main` while keeping any local `.windsurf` tweaks intact.

## When to Run
- Before beginning new feature/fix work
- When upstream has new commits and your fork is behind
- After finishing a PR and before picking up the next task

## Prerequisites
1. Ensure Docker/DevContainer sessions are stopped so git operations are safe.
2. Commit or stash any **code** changes you want to keep. This workflow focuses on syncing branches, not conflict resolution.
3. Decide whether `.windsurf` customizations should stay local-only or be shared. If local-only, plan to stash and reapply them.
4. Optional but recommended: create a private repo (or branch) that stores your personalized `.windsurf` content so it never pollutes upstream PRs.

### One-time: create a private Windsurf config repo (optional)
Use GitHub CLI (`gh`) so your personal rules are backed up but isolated from MeshMonitor PR branches.
```bash
gh auth status             # ensure CLI is logged in
gh repo create <user>/windsurf-config --private --source=. --remote=wind --description "Personal Windsurf settings"
```
- Replace `<user>` with your GitHub handle.
- This creates a private repo and adds a `wind` remote pointing to it. Immediately prune everything except `.windsurfrules` / `.windsurf/**` in that new repo so it only holds editor configs.
- Alternative: instead of a separate repo, create a local-only branch (e.g., `personal/windsurf`) within this repo as described below.

### One-time: establish a personal Windsurf branch inside this repo
```bash
git checkout -b personal/windsurf upstream/main
cp .windsurfrules .windsurfrules.personal   # optional backup
git add .windsurfrules .windsurf
git commit -m "chore: personalize windsurf settings"
git push origin personal/windsurf
```
- Store your preferred `.windsurf` content here. This branch never merges upstream; it just tracks your personal config.
- If you prefer the separate private repo above, treat that as the source of truth and pull from it in place of `personal/windsurf`.

## Workflow

### 1. Inspect current status
```bash
git status -sb
```
- If you have work in progress, either commit it or stash it.
- For `.windsurf` changes you want to keep locally, either stash (`git stash push -- .windsurf .windsurfrules`) or rely on the `personal/windsurf` branch/private repo created above.

### 2. Fetch latest upstream main
```bash
git fetch upstream main
```
- Verifies connectivity and downloads new commits without touching your working tree.

### 3. Update local main from upstream
```bash
git checkout main
git pull --ff-only upstream main
```
- `--ff-only` guarantees your `main` is a clean fast-forward of upstream without merge commits. If it fails, resolve local history issues before proceeding.

### 4. Push updated main to your fork
```bash
git push origin main
```
- Keeps GitHub fork in sync so future PRs compare cleanly.

### 5. Reapply personal `.windsurf` customizations
Choose the variant that matches where you store the files:

**a. From local stash**
```bash
git stash pop
```
- Use if you stashed `.windsurf` in step 1.

**b. From personal branch inside repo**
```bash
git checkout personal/windsurf -- .windsurf .windsurfrules
```
- Copies files without switching branches.

**c. From private repo created via `gh`**
```bash
git fetch wind main      # or whatever branch tracks your configs
git checkout wind/main -- .windsurf .windsurfrules
```
- Assumes you named the remote `wind` in the earlier step.

After copying, prevent accidental commits of these files on feature branches:
```bash
git update-index --skip-worktree .windsurfrules .windsurf/**
```
- Git will ignore local edits to these files until you remove the skip flag.

### 6. Create a fresh feature branch
```bash
git checkout -b feat/<short-topic>
```
- Replace `<short-topic>` with a descriptive slug (e.g., `feat/node-health-panel`).
- Branch now includes any `.windsurf` changes you reapplied plus the skip-worktree protection.

### 7. Verify environment files
- Ensure `.env`, `.windsurf/**`, and other local config remain as desired.
- Confirm `.windsurf` paths are still skipped (run `git ls-files -v | grep -E \"^S\"` to see skip-worktree entries).

### 8. Start development
- Run `npm run dev:full` via Docker/DevContainer per project rules.
- Implement feature work on this branch, commit with Conventional Commit messages, and push to `origin feat/<short-topic>` when ready.

## Tips
- Rerun this workflow whenever upstream diverges significantly before merging your branch to reduce rebase pain.
- If you maintain multiple long-lived branches, repeat steps 3–6 for each to keep them rebased on the latest main.
- Keep `.windsurf` scripts/workflows checked into source if they’re meant for collaborators; otherwise stash or skip-worktree to avoid noise.
- To update your personal config branch or private repo later, temporarily remove the skip flag (`git update-index --no-skip-worktree ...`), make edits, commit/push to the personal branch or `wind` remote, then reapply skip-worktree.
