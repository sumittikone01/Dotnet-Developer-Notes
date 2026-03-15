
# 01 — Init, Clone, Config

---

## 🎯 One-Line Definition

> **`init` creates a new Git repo from scratch, `clone` downloads an existing one, and `config` sets your identity so every commit knows who made it.**

---

## 🔑 `git init` — Start a New Repo

Turns any folder into a Git repository:

```bash
# Create a new project folder and init
mkdir MyEmployeeApp
cd MyEmployeeApp
git init

# Output:
# Initialized empty Git repository in /MyEmployeeApp/.git/

# What happened:
#   .git/ folder created — this IS the repository
#   One hidden folder = entire version control system
#   Your files are NOT yet tracked (nothing staged, nothing committed)
```

```bash
# OR: init AND create the folder in one step
git init MyEmployeeApp
# Creates MyEmployeeApp/ and initializes Git inside it
```

After init — your first commit:

```bash
git add .                         # stage all files
git commit -m "Initial commit"    # first snapshot
```

---

## 🔑 `git clone` — Download an Existing Repo

Gets a complete copy of a remote repository — all commits, all branches, all history:

```bash
# Basic clone (creates folder named after the repo)
git clone https://github.com/company/employee-app.git
# Creates: employee-app/ with full history inside

# Clone into a specific folder name
git clone https://github.com/company/employee-app.git MyApp
# Creates: MyApp/ instead

# Clone via SSH (preferred in professional settings)
git clone git@github.com:company/employee-app.git

# What clone does:
#   1. Creates local directory
#   2. Initializes .git/ inside it
#   3. Downloads ALL objects (commits, trees, blobs)
#   4. Sets up "origin" remote pointing to the URL
#   5. Checks out the default branch (usually main)
```

---

## 🔑 `git config` — Set Your Identity

Git records your name and email on every commit. **Set these before your first commit.**

```bash
# ── Global config (applies to ALL repos on your machine) ─────
git config --global user.name  "Alice Johnson"
git config --global user.email "alice@company.com"

# ── Local config (overrides global for this repo only) ───────
git config --local user.name  "Alice"
git config --local user.email "alice@personal.com"

# Use local when your work and personal repos need different emails
```

---

## 🔑 Essential Config Settings

```bash
# Set your default editor (for commit messages when needed)
git config --global core.editor "code --wait"      # VS Code
git config --global core.editor "notepad"           # Notepad
git config --global core.editor "vim"               # Vim

# Set default branch name (modern standard is "main")
git config --global init.defaultBranch main

# Make Git remember credentials (Windows)
git config --global credential.helper manager-core

# Better diff output (shows word-level changes)
git config --global diff.wordRegex .

# Always push to same branch name on remote
git config --global push.default current

# Set line ending handling (Windows — important for teams)
git config --global core.autocrlf true
# (Mac/Linux: core.autocrlf input)
```

---

## 🔑 Viewing and Editing Config

```bash
# View all config settings
git config --list

# View global settings only
git config --global --list

# View a specific setting
git config user.name          # shows: Alice Johnson
git config user.email         # shows: alice@company.com

# Open global config file in editor
git config --global --edit

# Config files location:
#   Global: ~/.gitconfig  (C:\Users\YourName\.gitconfig on Windows)
#   Local:  .git/config   (inside the repo)
#   Local overrides Global
```

---

## 🔑 `.gitignore` — Tell Git What to Ignore

After init or clone, create a `.gitignore` to exclude files:

```
# .gitignore for ASP.NET Core project

# Build outputs
bin/
obj/

# User-specific Visual Studio files
.vs/
*.user
*.suo

# Sensitive config (NEVER commit connection strings or secrets)
appsettings.Development.json
appsettings.Production.json
*.env

# NuGet packages (restored automatically)
packages/

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/
```

```bash
# Check what git sees (ignoring gitignore rules)
git status

# Check if a specific file is ignored
git check-ignore -v appsettings.Development.json
# Output: .gitignore:7:appsettings.Development.json  ← ignored at line 7
```

---

## 📊 Quick Reference

| Command                                | What It Does                       |
| -------------------------------------- | ---------------------------------- |
| `git init`                           | Create new repo in current folder  |
| `git init <name>`                    | Create folder and init repo inside |
| `git clone <url>`                    | Download complete repo from URL    |
| `git clone <url> <name>`             | Clone into a specific folder name  |
| `git config --global user.name "X"`  | Set your name (all repos)          |
| `git config --global user.email "X"` | Set your email (all repos)         |
| `git config --list`                  | Show all current settings          |
| `git config --global --edit`         | Open global config in editor       |

---

## ❓ Interview Questions

**Q: What does `git init` do?**

> Creates a `.git/` folder in the current directory. This folder contains everything Git needs — the object store, branch references, config, and staging area. The directory becomes a Git repository.

**Q: What is the difference between `git init` and `git clone`?**

> `git init` creates a new empty repository from scratch. `git clone` creates a local copy of an existing remote repository, downloading all commits, branches, and history.

**Q: Why do you need to set `user.name` and `user.email` in Git config?**

> Git records the author's name and email on every commit. Without this config, commits either fail or show wrong authorship. Use `--global` to set it once for all repos on your machine, or `--local` to override per-repo.
>
