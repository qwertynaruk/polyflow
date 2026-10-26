---
name: sync-pr
description: >-
  Sync the latest primary branch (main/master) into the current working branch, run quality gates, push, and create or update a Pull Request. Works across monorepos and polyrepos. Trigger with /sync-pr, /pr, "sync and pr", or "sync branch main and create pr".
---

# Sync and Pull Request Workflow

Automates synchronizing the upstream primary branch into the current working branch, running verification checks, pushing to remote, and creating or updating a Pull Request. Adaptively detects package managers, monorepo/polyrepo workspaces, and PR templates.

## Execution Steps

1. **Verify Git Working Tree & Changes**:
   - Check status with `git status`.
   - Ensure intentional changes are committed before starting the sync. If uncommitted changes exist, ask user or commit with a descriptive message.

2. **Detect Project Environment & Primary Branch**:
   - **Primary branch**: Detect `main` or `master`:
     ```bash
     git rev-parse --verify origin/main 2>/dev/null && echo "main" || echo "master"
     ```
   - **Current branch**: `git rev-parse --abbrev-ref HEAD` (ensure not directly on primary branch).
   - **Package Manager**:
     - `pnpm-lock.yaml` ➔ `pnpm`
     - `yarn.lock` ➔ `yarn`
     - `bun.lockb` or `bun.lock` ➔ `bun`
     - `package-lock.json` ➔ `npm` (fallback: `npm`)
   - **Workspace Architecture**:
     - Monorepo: `pnpm-workspace.yaml`, `turbo.json`, `lerna.json`, or root `package.json` workspaces.
     - Polyrepo / Single repo: Standalone `package.json`.

3. **Sync with Primary Branch**:
   - Run `git fetch origin`
   - Merge upstream:
     ```bash
     git merge origin/<primary_branch> -m "Merge branch '<primary_branch>' into <current_branch>"
     ```
   - If there are conflicts:
     - Inspect conflicts with `git status` and `view_file`.
     - Resolve conflicts adhering strictly to project architectural boundaries (and doctrine if present).
     - Stage resolved files: `git add <resolved_files>`
     - Complete the merge: `git commit -m "Merge branch '<primary_branch>' into <current_branch>"`

4. **Verify Quality Gates (Adaptive Execution)**:
   - Inspect root `package.json` `scripts`:
     - **TypeScript**:
       - Monorepo: Run project references build mode `-b` (`<pm> --filter "*" exec tsc -b` or `<pm> exec tsc -b`) or `<pm> run typecheck` if available.
       - Polyrepo: Run `<pm> run typecheck` if present, or `npx tsc --noEmit`.
     - **Lint**:
       - Run `<pm> run lint` if present.
       - Run `<pm> run lint:fsd` if present.
     - **Unit Tests**:
       - If `test` script exists and is not an unconfigured placeholder: run `<pm> test --run` or `<pm> test`.
     - **Build Verification**:
       - If `build` script exists: run `<pm> run build`.
   - If any gate fails, fix issues before pushing.

5. **Push to Remote**:
   - Push current branch to remote:
     ```bash
     git push -u origin <current_branch>
     ```

6. **Create or Update Pull Request**:
   - Check if a PR already exists for this branch:
     ```bash
     gh pr list --head <current_branch>
     ```
   - **If a PR exists**:
     - The push automatically updates the open PR.
     - Retrieve PR URL: `gh pr view --json url -q .url`
     - Report the updated PR to the user.
   - **If no PR exists**:
     - Inspect commit logs: `git log origin/<primary_branch>..<current_branch> --oneline`
     - Draft a concise Conventional PR title (e.g. `feat(...): ...` or `fix(...): ...`).
     - **PR Body Template**:
       - Check if `.github/pull_request_template.md` exists:
         - If found, format PR body following its structure.
         - If not found, use standard clean structure:
           - `## Summary of Changes`
           - `## Type of Change`
           - `## Testing & Verification`
     - Create the PR:
       ```bash
       gh pr create --base <primary_branch> --head <current_branch> --title "<title>" --body "<body>"
       ```

7. **Report to User**:
   - Provide the PR URL.
   - Summarize merged commits from primary branch and newly pushed commits.
   - Confirm quality gate results (TypeScript, Lint, Tests, Build).
