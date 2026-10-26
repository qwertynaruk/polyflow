# 🌊 PolyFlow

> **Adaptive Git & Delivery Workflows for AI Coding Agents**  
> _Seamlessly automate branch synchronization, quality gates, and pull requests across Monorepos and Polyrepos._

[![Architecture](https://img.shields.io/badge/Architecture-Monorepo%20%26%20Polyrepo-blue)](#-adaptive-architecture-monorepo--polyrepo-support)
[![Compatibility](https://img.shields.io/badge/Compatible-Antigravity%20%7C%20Claude%20Code%20%7C%20Cursor%20%7C%20Windsurf-success)](#%EF%B8%8F-cross-ide--ai-tool-compatibility)
[![Git Workflows](https://img.shields.io/badge/Workflows-Sync%20%7C%20Quality%20Gates%20%7C%20PR-orange)](#-available-skills)

---

## 💡 About PolyFlow

**PolyFlow** is a standardized, **IDE-agnostic** agent workflow toolkit designed to eliminate friction in daily Git operations.

- **Why "Poly"?** — Built to thrive across diverse repository topologies (**Poly**repos, **Mono**repos, and multi-package workspaces) without requiring separate configurations.
- **Why "Flow"?** — Creates a continuous, automated delivery flow: from upstream branch synchronization and automated conflict resolution to quality gates and pull request creation.

Compatible with **Google Antigravity IDE, Claude Code, Cursor, Windsurf, GitHub Copilot, Cline / Roo Code**, and human developers via terminal.

---

## 📦 Available Skills

| Skill           | Trigger Commands / Prompts                                         | Description                                                                                                                                                                         |
| --------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`sync-main`** | `/sync-main`, `sync main`, `sync branch main`                      | Syncs the upstream primary branch (`main` / `master`) into your working branch with automated conflict resolution and quality verification.                                         |
| **`sync-pr`**   | `/sync-pr`, `/pr`, `sync and pr`, `sync branch main and create pr` | Full delivery workflow: syncs primary branch, executes quality gates (typecheck, lint, test, build), pushes to remote, and creates or updates a Pull Request via GitHub CLI (`gh`). |

---

## 🛠️ Cross-IDE & AI Tool Compatibility

Because all skills are written in **Open Standard Markdown (`SKILL.md`) with YAML Frontmatter**, any modern AI coding assistant can interpret and execute them:

| AI Assistant / IDE             | How to Use                                                                                                     |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **Google Antigravity IDE**     | Automatically discovered in `.agents/skills/`. Trigger directly with `/sync-main` or `/sync-pr` in the chat.   |
| **Claude Code**                | In chat, prompt: `"follow workflow in .agents/skills/sync-pr/SKILL.md"` or add references in your `CLAUDE.md`. |
| **Cursor IDE**                 | Type `@.agents/skills/sync-pr/SKILL.md` in Composer/Chat, or symlink into `.cursor/rules/`.                    |
| **Windsurf (Cascade)**         | Reference `@.agents/skills/sync-pr/SKILL.md` in Cascade or copy instructions into `.windsurfrules`.            |
| **Cline / Roo Code / Copilot** | Prompt the agent to read and execute `.agents/skills/sync-pr/SKILL.md`.                                        |
| **Human Developer**            | Read `SKILL.md` as standard SOP runbooks and execute the shell commands directly in your terminal.             |

---

## ⚡ Adaptive Architecture: Monorepo & Polyrepo Support

Both `sync-main` and `sync-pr` are designed to be **Universal and Adaptive**. When triggered, PolyFlow inspects the active project environment and automatically adjusts its execution:

### 1. Primary Branch Detection

- Automatically detects whether the remote primary branch is `origin/main` or `origin/master`.

### 2. Package Manager Detection

- Inspects lockfiles at repository root:
  - `pnpm-lock.yaml` ➔ `pnpm`
  - `yarn.lock` ➔ `yarn`
  - `bun.lockb` or `bun.lock` ➔ `bun`
  - `package-lock.json` ➔ `npm` (fallback: `npm`)

### 3. Workspace Type & Quality Gates

- **Monorepo** (`turbo.json`, `pnpm-workspace.yaml`, or root `workspaces`):
  - **Type Check**: Uses project references build mode (`<pm> --filter "*" exec tsc -b` or `<pm> exec tsc -b`) or root `typecheck` script.
  - **Lint**: Runs `<pm> run lint` and `<pm> run lint:fsd` (if Steiger/FSD is configured).
  - **Unit Tests**: Runs `<pm> test --run` or `<pm> test`.
  - **Build**: Runs `<pm> run build`.
- **Polyrepo (Single project)**:
  - **Type Check**: Runs `<pm> run typecheck` or `npx tsc --noEmit`.
  - **Lint**: Runs `<pm> run lint`.
  - **Unit Tests**: Runs `<pm> test` (if defined in `package.json`).
  - **Build**: Runs `<pm> run build` (if defined in `package.json`).

### 4. Pull Request Formatting

- Detects `.github/pull_request_template.md` if present in the repo and populates all relevant checklists.
- Falls back to a standard Conventional PR template (Summary, Type of Change, Verification).

---

## 🚀 How to Use

### 1. Synchronize with Main Branch

Ask your AI assistant:

```text
/sync-main
```

Or in natural language:

> "Sync the latest main branch into my current branch"
> "ช่วย sync main ให้หน่อย"

**Workflow executed by PolyFlow:**

1. Checks for uncommitted work.
2. Fetches `origin` and merges `origin/main` (or `origin/master`).
3. Handles merge conflicts thoughtfully respecting architectural patterns.
4. Executes project typecheck and linter.
5. Reports status.

### 2. Sync, Verify Quality Gates, and Create/Update PR

Ask your AI assistant:

```text
/sync-pr
```

Or in natural language:

> "Sync main, verify quality gates, and create a pull request"
> "/pr"

**Workflow executed by PolyFlow:**

1. Commits any staged/pending work if requested.
2. Fetches and merges latest `origin/main` (or `master`).
3. Runs all quality gates:
   - TypeScript check
   - Lint & Architecture rules
   - Unit tests
   - Production bundle build
4. Pushes current branch to remote (`git push -u origin <branch>`).
5. Checks if a PR already exists:
   - **Existing PR**: Automatically updated by the push.
   - **New PR**: Generates Conventional title & structured description, then creates PR via `gh pr create`.
6. Returns the clickable PR URL to the user.

---

## 🌐 Sharing Across the Team & Other Repositories

You can share PolyFlow with teammates and use it across all repositories using two approaches:

### Approach A: Global Installation (Recommended for Individual Devs)

To use PolyFlow across **ALL** repositories on your computer without copying files into every repo:

1. Push this folder to a shared GitHub repository (e.g., `https://github.com/your-org/polyflow.git`).
2. Developers clone it once into their machine's global customization directory:
   - **Google Antigravity IDE**:
     ```bash
     git clone https://github.com/your-org/polyflow.git ~/.gemini/config/plugins/polyflow
     ```
   - **Claude Code / Generic AI tools**:
     Clone into your personal dotfiles or skills folder (e.g. `~/.config/polyflow/`) and link as needed.
3. **Done!** Whenever you open any project (monorepo or single repo), your AI assistant can use `/sync-main` and `/sync-pr`.
4. To update: run `git pull` in the cloned directory.

---

### Approach B: Project-level Repository Customization (`.agents/`)

If you want a specific repository to have PolyFlow pre-configured for anyone on the team who clones it:

1. Place the skills in `<repo-root>/.agents/skills/`:
   ```text
   <repo-root>/
     └── .agents/
         ├── README.md
         └── skills/
             ├── sync-main/
             │   └── SKILL.md
             └── sync-pr/
                 └── SKILL.md
   ```
2. Commit and push `.agents/` to git.
3. Any developer opening the repository with Antigravity, Claude Code, Cursor, or Windsurf will immediately have access to these workflows.

---

## 📋 Requirements

- **Git** installed and authenticated (`git push` access).
- **GitHub CLI (`gh`)** installed and authenticated (`gh auth login`) for automated PR creation.
