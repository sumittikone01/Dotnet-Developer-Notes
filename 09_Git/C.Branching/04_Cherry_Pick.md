
# 04 — Cherry Pick

---

## 🎯 One-Line Definition

> **`git cherry-pick` copies a specific commit from one branch and applies it to your current branch — you pick exactly the commit you want, leaving the rest behind.**

---

## 🔑 The Problem Cherry-Pick Solves

```
SCENARIO:
  You have a feature branch with 5 commits.
  Commit 3 is a critical bug fix needed in production RIGHT NOW.
  You don't want commits 1, 2, 4, 5 in production yet.

  feature/export: a─b─[bugfix]─d─e
                           ↑
                           just this one!

  main:           a─b

git cherry-pick <hash-of-bugfix>

  main:           a─b─bugfix'
  (bugfix' = same changes, new commit hash, new parent = b)

  feature/export unchanged: a─b─bugfix─d─e
```

---

## 🔑 Basic Cherry-Pick

```bash
# Find the commit hash you want
git log --oneline feature/excel-export
# a3f8c12 Fix anti-forgery token in DataSource transport   ← want this
# b7e2d01 Add export button (incomplete)
# c9a1f44 Wip: Excel download
# d4e5f62 Start Excel export feature

# Switch to the target branch
git checkout main

# Apply the specific commit
git cherry-pick a3f8c12

# Done! The fix is now in main.
# a3f8c12 is copied as a new commit with a different hash.
```

---

## 🔑 Cherry-Pick Multiple Commits

```bash
# Pick several specific commits (not necessarily adjacent)
git cherry-pick a3f8c12 b7e2d01 c9a1f44

# Pick a range of commits (a..b picks commits AFTER a up to and including b)
git cherry-pick a3f8c12..d4e5f62
# Picks: b7e2d01, c9a1f44, d4e5f62 (not a3f8c12 itself)

# Pick a range INCLUDING the first commit (use ^)
git cherry-pick a3f8c12^..d4e5f62
# Picks: a3f8c12, b7e2d01, c9a1f44, d4e5f62
```

---

## 🔑 Cherry-Pick Options

```bash
# ── Apply changes but don't commit yet ────────────────────
git cherry-pick a3f8c12 --no-commit
git cherry-pick a3f8c12 -n    # short form
# Stages the changes — you can review and modify before committing
# Useful when you want to combine with other changes

# ── Edit the commit message ───────────────────────────────
git cherry-pick a3f8c12 --edit
git cherry-pick a3f8c12 -e    # short form
# Opens editor to modify the commit message before applying

# ── Add reference to original commit in message ───────────
git cherry-pick a3f8c12 -x
# Adds "(cherry picked from commit a3f8c12)" to the commit message
# Useful for tracking where the fix came from
```

---

## 🔑 Handling Cherry-Pick Conflicts

```bash
git cherry-pick a3f8c12
# CONFLICT (content): Merge conflict in EmployeeController.cs

# Fix the conflict...
git add EmployeeController.cs

# Continue
git cherry-pick --continue

# OR abort if you change your mind
git cherry-pick --abort
```

---

## 🔑 Real-World Scenarios

```
Scenario 1 — Hotfix from feature to main:
  feature/payments has a critical security fix buried in it
  git cherry-pick <security-fix-hash> onto main
  Deploy main to production immediately

Scenario 2 — Fix applied to wrong branch:
  You committed a bug fix to feature/export by mistake
  It belongs on main
  git cherry-pick <fix-hash> onto main
  Then git revert on feature branch or reset it

Scenario 3 — Backport fix to old release branch:
  main has a bug fix
  You maintain release/1.x for old clients
  git cherry-pick <fix-hash> onto release/1.x
  Both versions get the fix
```

---

## 📊 Quick Reference

| Command                        | What It Does                       |
| ------------------------------ | ---------------------------------- |
| `git cherry-pick <hash>`     | Copy one commit to current branch  |
| `git cherry-pick <h1> <h2>`  | Copy multiple specific commits     |
| `git cherry-pick <h1>..<h2>` | Copy a range (excluding h1)        |
| `git cherry-pick -n <hash>`  | Apply changes without committing   |
| `git cherry-pick -x <hash>`  | Include original hash in message   |
| `git cherry-pick --continue` | Continue after conflict resolution |
| `git cherry-pick --abort`    | Cancel cherry-pick in progress     |

---

## ❓ Interview Questions

**Q: What does `git cherry-pick` do?**

> It copies a specific commit from anywhere in the repository and applies those same changes as a new commit on your current branch. The original commit remains untouched — cherry-pick creates a copy with a new hash.

**Q: When would you use cherry-pick instead of merge?**

> When you only want specific commits, not an entire branch. Common scenarios: applying a hotfix from a feature branch to main without the incomplete feature, backporting a fix to an older release branch, or recovering a commit accidentally made on the wrong branch.

**Q: What is the downside of cherry-pick?**

> It creates a duplicate commit — the same change exists in two places with different hashes. When the original feature branch eventually merges, the cherry-picked changes may show as conflicts since Git sees them as different commits. Use cherry-pick for urgent targeted fixes, not as a regular workflow.
>
