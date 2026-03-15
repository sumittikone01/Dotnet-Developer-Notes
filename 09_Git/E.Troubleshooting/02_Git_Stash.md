
# 02 — Git Stash

---

## 🎯 One-Line Definition

> **`git stash` temporarily shelves your uncommitted changes so you can switch branches or pull updates cleanly, then brings them back when you're ready to continue.**

---

## 🔑 The Problem Stash Solves

```
SCENARIO:
  You're halfway through a feature on feature/export.
  Your manager calls: "Critical bug in production — fix NOW."
  You need to switch to main to fix the bug.
  But you have half-finished, unstaged changes.

  git checkout main → ERROR: "Your local changes would be overwritten"

  You can't commit half-done work.
  You don't want to lose your work.
  You need to switch branches right now.

  SOLUTION: git stash
  → Saves your changes temporarily
  → Working directory is clean
  → Switch to main, fix the bug
  → Come back, apply stash, continue where you left off
```

---

## 🔑 Basic Stash Commands

```bash
# ── Save current changes to stash ─────────────────────────
git stash
git stash save "half-done employee export feature"   # with a description

# ── List all stashes ───────────────────────────────────────
git stash list
# stash@{0}: On feature/export: half-done employee export feature
# stash@{1}: WIP on main: fix validation bug
# stash@{2}: WIP on feature/report: wip report filters

# ── Apply the most recent stash (keeps it in stash list) ──
git stash apply

# ── Apply AND remove from stash list ──────────────────────
git stash pop

# ── Apply a specific stash ────────────────────────────────
git stash apply stash@{2}
git stash pop stash@{1}

# ── Remove a specific stash ───────────────────────────────
git stash drop stash@{0}

# ── Remove ALL stashes ────────────────────────────────────
git stash clear
```

---

## 🔑 The Full Workflow

```bash
# Situation: halfway through feature, need to fix urgent bug

# Step 1: Stash current work
git stash save "WIP: Excel export — halfway through download action"

# Step 2: Switch to main and fix
git checkout main
git pull
git checkout -b hotfix/login-null-ref
# ...fix the bug...
git add . && git commit -m "Fix null reference in LoginController"
git checkout main
git merge hotfix/login-null-ref
git push

# Step 3: Go back to feature
git checkout feature/excel-export

# Step 4: Restore stashed work
git stash pop    # applies latest stash and removes it from stash list

# Continue where you left off ✅
```

---

## 🔑 Stash Options

```bash
# Stash including untracked files (new files not yet git add'd)
git stash -u
git stash --include-untracked

# Stash ONLY specific files
git stash push -m "stash description" -- Controllers/EmployeeController.cs Models/Employee.cs

# Show what's in the most recent stash
git stash show
git stash show -p    # show with full diff

# Show what's in a specific stash
git stash show stash@{1} -p

# Apply stash to a DIFFERENT branch
git checkout other-branch
git stash apply stash@{0}
```

---

## 🔑 `apply` vs `pop` — Which to Use

```
git stash apply:
  → Restores the stash
  → KEEPS the stash in the stash list
  → Use when: you might need to apply the same stash again
               OR you want to apply it to multiple branches

git stash pop:
  → Restores the stash
  → REMOVES it from the stash list
  → Use when: you're done with the stash and it's a one-time restore
  → Most common choice for normal use
```

---

## ❓ Interview Questions

**Q: What does `git stash` do?**

> It saves your uncommitted changes (both staged and unstaged) to a temporary stack and reverts your working directory to a clean state. You can then switch branches, pull updates, or do other work, then restore the stashed changes later with `git stash pop` or `git stash apply`.

**Q: What is the difference between `git stash pop` and `git stash apply`?**

> Both restore the stash. `pop` removes it from the stash list after applying (use this for normal one-time restore). `apply` restores it but keeps it in the list — useful when you want to apply the same changes to multiple branches.

**Q: Does `git stash` save untracked files?**

> By default, no — it only saves tracked files. Use `git stash -u` or `git stash --include-untracked` to include new files that haven't been `git add`'d yet.
>
