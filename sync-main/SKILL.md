---
name: sync-main
description: >-
  Sync the latest primary branch (main/master) into the current working branch with automated conflict resolution,
  type checking, and linting. Works across monorepos and polyrepos. Trigger with /sync-main, "sync main", or "sync branch main".
---

# Sync Main / Primary Branch Workflow

Automates synchronizing the upstream primary branch (`origin/main` or `origin/master`) into the current working git branch. Adaptively detects project setup (monorepo vs polyrepo, package manager, and available quality gate scripts).

## Execution Steps

1. **Verify Git Working Tree**:
   - Run `git status` to ensure working tree is clean or changes are committed/stashed.
   - If there are uncommitted changes, notify user or stash them if appropriate.

2. **Detect Project Environment & Primary Branch**:
   - **Primary branch**: Check whether the default branch is `main` or `master`:
     ```bash
     git rev-parse --verify origin/main 2>/dev/null && echo "main" || echo "master"
     ```
   - **Current branch**: `git rev-parse --abbrev-ref HEAD` (ensure not directly on the primary branch).
   - **Package Manager**:
     - `pnpm-lock.yaml` ➔ `pnpm`
     - `yarn.lock` ➔ `yarn`
     - `bun.lockb` or `bun.lock` ➔ `bun`
     - `package-lock.json` ➔ `npm` (fallback: `npm`)
   - **Architecture & Workspace Type**:
     - Monorepo: `pnpm-workspace.yaml`, `turbo.json`, `lerna.json`, or root `package.json` workspaces.
     - Polyrepo / Single repo: Standalone `package.json`.

3. **Fetch and Merge Primary Branch**:
   - Run `git fetch origin`
   - Merge upstream:
     ```bash
     git merge origin/<primary_branch> -m "Merge branch '<primary_branch>' into <current_branch>"
     ```

4. **Handle Merge Conflicts (if any)**:
   - If conflicts arise, inspect conflicted files with `git status` and `view_file`.
   - Resolve conflicts thoughtfully respecting the project's design system and architectural boundaries (e.g., FSD layers if applicable).
   - Stage resolved files: `git add <resolved_files>`
   - Finalize merge commit: `git commit -m "Merge branch '<primary_branch>' into <current_branch>"`

5. **Verify Quality Gates (Adaptive Execution)**:
   - Inspect root `package.json` `scripts`:
     - **TypeScript**:
       - Monorepo: If project references build mode `-b` is used, run `<pm> --filter "*" exec tsc -b` or `<pm> exec tsc -b`. If a `typecheck` script exists, run `<pm> run typecheck`.
       - Polyrepo: If `typecheck` script exists, run `<pm> run typecheck`. Otherwise run `npx tsc --noEmit` if `tsconfig.json` exists.
     - **Lint**:
       - If `lint` script exists, run `<pm> run lint`.
       - If `lint:fsd` script exists (e.g., Steiger for FSD), run `<pm> run lint:fsd`.
   - If errors occur, fix them before proceeding.

6. **Report to User**:
   - Summarize merged commits from `origin/<primary_branch>`.
   - Confirm TypeScript and Linter status.
   - Indicate whether branch is ready to be pushed or needs additional edits.
