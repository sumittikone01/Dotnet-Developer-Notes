
# 01 — Creating and Switching Branches

---

## 🎯 One-Line Definition

> **A branch is an independent line of development — create one to work on a feature or bug fix without touching `main`, then merge it back when done.**

---

## 🔑 Why Branches Exist

```
WITHOUT branches (everyone commits to main):
  Alice commits broken code → everyone's main is broken
  Bob's work gets mixed with Alice's → messy, hard to review
  Can't have two features in progress simultaneously
  Can't keep production code stable

WITH branches:
  main           → always stable, production-ready
  feature/export → Alice working on Excel export
  feature/report → Bob working on salary report
  bugfix/pager   → Carol fixing the grid pager

  Everyone works independently.
  main stays clean until features are reviewed and merged.
```

---

## 🔑 Creating a Branch

```bash
# ── Create a branch (stays on current branch) ─────────────
git branch feature/excel-export

# ── Create AND switch to it immediately ───────────────────
git checkout -b feature/excel-export
git switch -c feature/excel-export     # modern syntax (Git 2.23+)

# ── Create from a specific commit/branch ──────────────────
git checkout -b bugfix/pager main      # branch off main
git checkout -b hotfix/login v1.0.0   # branch off a tag
git checkout -b feature/x a3f8c12     # branch off specific commit

# ── See all local branches ────────────────────────────────
git branch
# Output:
#   feature/excel-export
# * main                     ← * = current branch
#   bugfix/pager

# ── See all branches (local + remote tracking) ────────────
git branch -a

# ── See branches with last commit info ────────────────────
git branch -v
# Output:
#   feature/excel-export  a3f8c12 Add export button
# * main                  b7e2d01 Fix pager bug
```

---

## 🔑 Switching Branches

```bash
# ── Switch to an existing branch ──────────────────────────
git checkout main
git switch main              # modern syntax

git checkout feature/export
git switch feature/export

# ── Switch to previous branch ─────────────────────────────
git checkout -           # goes back to whatever you were on before
git switch -             # same

# ── What happens when you switch:
#   1. Git reads the destination branch's commit
#   2. Updates all files in your working directory to match
#   3. Moves HEAD to point to the new branch
#   4. Your staged/committed changes stay in history
#   5. UNSTAGED changes follow you (with a warning if conflict)
```

---

## 🔑 Branch Naming Conventions

```
Standard patterns used by teams:

  feature/employee-export-excel    ← new feature
  feature/kendo-grid-popup-edit
  bugfix/grid-pager-count-wrong    ← bug fix
  bugfix/date-off-by-one
  hotfix/login-crash-production    ← urgent production fix
  release/1.2.0                    ← release preparation
  chore/update-dependencies        ← maintenance
  docs/update-readme

Rules:
  → lowercase only
  → hyphens not spaces or underscores
  → descriptive: what does this branch do?
  → no special characters
  → keep it short but clear
```

---

## 🔑 Deleting Branches

```bash
# ── Delete a merged branch (safe — won't delete unmerged) ─
git branch -d feature/excel-export

# ── Force delete (even if unmerged) ──────────────────────
git branch -D feature/abandoned-idea
# Use -D when you want to discard work on a branch

# ── Delete a remote branch ───────────────────────────────
git push origin --delete feature/excel-export
# OR:
git push origin :feature/excel-export

# ── Clean up remote tracking branches for deleted remotes ─
git fetch --prune
```

---

## 🔑 Renaming a Branch

```bash
# Rename current branch
git branch -m new-name

# Rename a specific branch
git branch -m old-name new-name

# If already pushed, update remote too:
git push origin --delete old-name
git push -u origin new-name
```

---

## 📊 Quick Reference

| Command                             | What It Does                    |
| ----------------------------------- | ------------------------------- |
| `git branch <name>`               | Create branch (stay on current) |
| `git checkout -b <name>`          | Create + switch to new branch   |
| `git switch -c <name>`            | Same (modern syntax)            |
| `git checkout <name>`             | Switch to existing branch       |
| `git switch <name>`               | Same (modern syntax)            |
| `git checkout -`                  | Switch to previous branch       |
| `git branch`                      | List local branches             |
| `git branch -a`                   | List all branches               |
| `git branch -v`                   | List with last commit           |
| `git branch -d <name>`            | Delete merged branch            |
| `git branch -D <name>`            | Force delete branch             |
| `git push origin --delete <name>` | Delete remote branch            |

---

## ❓ Interview Questions

**Q: What is a Git branch?**

> A lightweight movable pointer to a specific commit. Creating a branch lets you diverge from the main line of development and work independently. Branches in Git are just 41-byte files — instant and free to create.

**Q: What is the difference between `git branch <name>` and `git checkout -b <name>`?**

> `git branch <name>` creates the branch but stays on the current branch. `git checkout -b <name>` (or `git switch -c <name>`) creates the branch AND immediately switches to it — the most common way to start working on something new.

**Q: What happens to your working directory when you switch branches?**

> Git replaces all tracked files in your working directory to match the state of the branch you switch to. Uncommitted changes that don't conflict follow you. Changes that would conflict with files in the destination branch prevent the switch — Git asks you to commit or stash first.
>
