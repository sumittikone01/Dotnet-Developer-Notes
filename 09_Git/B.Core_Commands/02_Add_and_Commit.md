
# 02 — Add and Commit

---

## 🎯 One-Line Definition

> **`git add` moves changes from your working directory into the staging area — `git commit` takes everything staged and saves it permanently as a snapshot in the repository.**

---

## 🔑 The Two-Step Flow

```
Working Directory         Staging Area            Repository
─────────────────         ────────────            ──────────
You edited files          git add →               git commit →
  EmployeeController.cs   prepare the             permanent snapshot
  Employee.cs              snapshot                saved forever
  site.css
```

Think of it like packing a box before shipping:

* `git add` = put items in the box
* `git commit` = seal and ship the box

---

## 🔑 `git add` — Stage Changes

```bash
# ── Stage a specific file ─────────────────────────────────────
git add EmployeeController.cs

# ── Stage multiple specific files ─────────────────────────────
git add EmployeeController.cs Employee.cs

# ── Stage all files in a folder ───────────────────────────────
git add Controllers/
git add Models/

# ── Stage ALL changes in the repo ─────────────────────────────
git add .
git add -A       # same as . but also stages deletions

# ── Stage interactively — choose chunks within a file ─────────
git add -p EmployeeController.cs
# Shows each changed chunk, asks: stage this? (y/n/s/?)
# y = yes  n = no  s = split into smaller chunks  ? = help

# ── Stage all modified tracked files (not new untracked) ──────
git add -u
```

---

## 🔑 `git commit` — Save the Snapshot

```bash
# ── Commit with message inline ────────────────────────────────
git commit -m "Add salary validation to employee form"

# ── Commit with multi-line message ────────────────────────────
git commit -m "Add salary validation to employee form

- Added [Range] annotation to Employee model
- Server returns validation errors via ModelState
- Kendo Grid popup shows field-level error messages"

# ── Stage AND commit all tracked changes in one step ──────────
git commit -am "Fix paging bug in employee grid"
# -a = auto-stage all modified tracked files (skips git add)
# ONLY works for already-tracked files — new files still need git add

# ── Amend the last commit (before pushing) ────────────────────
git commit --amend -m "Fixed: add salary validation to employee form"
# Replaces the last commit with a new one
# Use when you forgot to include a file or want to fix the message
# NEVER amend commits already pushed to shared remote

# ── Amend and add a forgotten file ────────────────────────────
git add ForgottenFile.cs
git commit --amend --no-edit    # amend without changing the message
```

---

## 🔑 Writing Good Commit Messages

Your commit message is documentation. Future you (and your teammates) will thank you.

```
FORMAT:
  Short summary (50 chars or less)

  Optional longer description explaining WHY (not WHAT —
  the diff shows what, the message explains why).
  Wrap at 72 characters.

EXAMPLES:

❌ BAD commit messages:
  "fix"
  "stuff"
  "wip"
  "asdfgh"
  "changes"
  "updated controller"

✅ GOOD commit messages:
  "Fix employee grid not loading after department filter change"
  "Add anti-forgery token to all Kendo DataSource POST requests"
  "Refactor DAL: extract connection string to appsettings"
  "Fix date off-by-one when sending HireDate to controller"

CONVENTION (many teams use):
  feat: Add Excel export to employee grid
  fix:  Correct salary validation range (10k-500k)
  docs: Update README with deployment steps
  style: Format EmployeeController whitespace
  refactor: Extract employee search to separate method
```

---

## 🔑 What Happens Inside Git on Commit

```
Before commit:
  Staging area has: EmployeeController.cs (modified)

git commit -m "Add validation"

Git does:
  1. Creates a BLOB for EmployeeController.cs content  → hash: 4a1b2c3
  2. Creates a TREE for the directory structure         → hash: f9e3c7a
  3. Creates a COMMIT object:                           → hash: a3f8c12
       tree:    f9e3c7a
       parent:  b7e2d01  (previous commit)
       author:  Alice <alice@company.com>
       date:    Mon Jun 15 09:30:00 2024
       message: "Add validation"
  4. Moves the branch pointer (main) to a3f8c12
  5. Clears the staging area
```

---

## 🔑 Checking What's Staged Before Committing

```bash
# See what's staged vs modified vs untracked
git status

# Output:
# On branch main
# Changes to be committed:        ← STAGED (green)
#   modified:   EmployeeController.cs
#   modified:   Employee.cs
#
# Changes not staged for commit:  ← MODIFIED but NOT staged (red)
#   modified:   site.css
#
# Untracked files:                ← new files Git doesn't know about
#   appsettings.Development.json

# See the EXACT diff of what's staged
git diff --staged

# See the diff of what's modified but not yet staged
git diff
```

---

## 🔑 Unstaging — Remove from Staging Area

```bash
# Unstage a file (keep changes in working directory)
git restore --staged EmployeeController.cs
# OR (older syntax):
git reset HEAD EmployeeController.cs

# Unstage ALL staged files
git restore --staged .
```

---

## 📊 Quick Reference

| Command                         | What It Does                     |
| ------------------------------- | -------------------------------- |
| `git add <file>`              | Stage specific file              |
| `git add .`                   | Stage all changes                |
| `git add -p <file>`           | Stage chunks interactively       |
| `git add -u`                  | Stage all modified tracked files |
| `git commit -m "msg"`         | Commit staged changes            |
| `git commit -am "msg"`        | Stage + commit all tracked       |
| `git commit --amend`          | Replace last commit              |
| `git restore --staged <file>` | Unstage a file                   |
| `git diff --staged`           | See what's about to be committed |

---

## ❓ Interview Questions

**Q: What is the difference between `git add` and `git commit`?**

> `git add` moves changes from the working directory to the staging area — it prepares them for the next commit without saving permanently. `git commit` takes everything in the staging area and saves it as a permanent snapshot in the repository history.

**Q: What does `git commit -am` do differently from `git add . && git commit -m`?**

> `git commit -am` stages and commits all modified *tracked* files in one step. However, it does NOT stage new (untracked) files. `git add .` stages everything including new files, then `git commit -m` commits what's staged.

**Q: When should you use `git commit --amend`?**

> When you want to fix the last commit — either to correct the commit message or to include a file you forgot to stage. Only use it on commits that haven't been pushed to a shared remote — amending rewrites history and will cause problems for others who already pulled that commit.
>
