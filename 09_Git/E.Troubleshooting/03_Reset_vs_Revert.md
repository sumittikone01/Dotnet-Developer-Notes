
# 03 — Reset vs Revert

---

## 🎯 One-Line Definition

> **`git reset` moves the branch pointer backward, rewriting history — `git revert` creates a new commit that undoes a previous one, keeping history intact. Reset is for local cleanup; Revert is safe for shared branches.**

---

## 🔑 The Core Difference — One Rule to Remember

```
┌────────────────────────────────────────────────────────────────┐
│  git reset  → REWRITES history. Moves the pointer back.       │
│              Use on LOCAL commits not yet pushed.             │
│                                                                │
│  git revert → ADDS to history. Creates an undo commit.        │
│              Safe on shared/pushed commits.                   │
└────────────────────────────────────────────────────────────────┘

If you've pushed → use REVERT.
If you haven't pushed → either works (reset is cleaner).
```

---

## 🔑 `git reset` — Move the Branch Pointer Back

### The Three Modes of Reset

```
STARTING STATE:
  Commits: a ← b ← c ← d ← e   (HEAD is at e)
  You want to undo commits d and e.

git reset a3f8c12 (commit b)
  → Branch pointer moves to b
  → What happens to the changes from c, d, e?
     Depends on which mode: --soft, --mixed, --hard
```

### `--soft` — Keep changes staged

```bash
git reset --soft HEAD~2
# Moves branch pointer back 2 commits
# Changes from those commits: KEPT in staging area
# Working directory: unchanged

BEFORE:
  Commits: a─b─c─d─e   HEAD=e
  Staged: nothing
  Working dir: clean

AFTER git reset --soft HEAD~2:
  Commits: a─b─c         HEAD=c (d and e removed from history)
  Staged: all changes from d and e are staged
  Working dir: unchanged

USE WHEN:
  → You want to squash last N commits into one
  → git reset --soft HEAD~3 → git commit -m "one clean commit"
```

### `--mixed` — Keep changes unstaged (DEFAULT)

```bash
git reset HEAD~2
git reset --mixed HEAD~2   # same — mixed is the default

# Moves branch pointer back 2 commits
# Changes from those commits: in working directory (unstaged)
# Staging area: cleared

AFTER git reset --mixed HEAD~2:
  Commits: a─b─c         HEAD=c
  Staged: nothing
  Working dir: has all changes from d and e (modified files)

USE WHEN:
  → You committed the wrong files and want to redo the commit
  → Unstage files and re-stage them differently
```

### `--hard` — Discard changes completely

```bash
git reset --hard HEAD~2

# Moves branch pointer back 2 commits
# Changes from those commits: GONE — permanently discarded
# Staging area: cleared
# Working directory: reset to match commit HEAD~2

AFTER git reset --hard HEAD~2:
  Commits: a─b─c         HEAD=c
  Staged: nothing
  Working dir: CLEAN (matches commit c exactly)
  d and e are gone — no trace

⚠️  WARNING: --hard discards uncommitted changes permanently.
    There is NO undo for --hard (unless you use git reflog quickly).
    USE WITH CAUTION.

USE WHEN:
  → You want to completely discard your last N commits
  → Throw away all experimental changes and start fresh
  → Dangerous but sometimes exactly what's needed
```

---

## 🔑 Common Reset Targets

```bash
# ── Unstage a file (undo git add) ────────────────────────
git reset HEAD EmployeeController.cs
git restore --staged EmployeeController.cs   # modern equivalent

# ── Undo the last commit, keep changes staged ────────────
git reset --soft HEAD~1
# Useful: you committed too early and want to add more

# ── Undo last commit, keep changes unstaged ──────────────
git reset HEAD~1
git reset --mixed HEAD~1   # same

# ── Undo last N commits cleanly ──────────────────────────
git reset --soft HEAD~3    # squash 3 into a new clean commit
git commit -m "Clean single commit"

# ── Completely throw away last commit ─────────────────────
git reset --hard HEAD~1

# ── Reset to match remote exactly (DANGEROUS — loses local work) ─
git reset --hard origin/main
# Useful when: your local branch is messed up and remote is correct
```

---

## 🔑 `git revert` — Create an Undo Commit

Revert creates a NEW commit that is the opposite of a specified commit:

```bash
# Revert the most recent commit
git revert HEAD

# Revert a specific commit
git revert a3f8c12

# Revert without immediately committing (review first)
git revert --no-commit a3f8c12
git revert -n a3f8c12     # short form
# Makes the reverting changes but lets you review before committing

# Revert multiple commits
git revert HEAD~2..HEAD    # revert last 2 commits
```

What revert looks like in history:

```
BEFORE:
  a ← b ← c ← d ← e   (e is a buggy commit)

git revert e   (revert commit e)

AFTER:
  a ← b ← c ← d ← e ← e'
  e' = "Revert 'Add broken export feature'"
  e' contains the OPPOSITE changes of e
  e itself is still there — history preserved
```

---

## 🔑 Revert in Practice

```bash
# Scenario: bad commit pushed to shared branch, need to undo

# Find the bad commit
git log --oneline
# a3f8c12 Add broken salary export (← this one broke production)
# b7e2d01 Fix pager count
# c9a1f44 Add grid filtering

# Revert it
git revert a3f8c12
# Git opens editor with message: "Revert 'Add broken salary export'"
# Save and close → creates revert commit

git push   # push the revert commit to remote
# Now remote has the fix — bad commit is neutralised
```

---

## 🔑 Reset vs Revert — Side by Side

```
SCENARIO: Commit d is bad. You want to undo it.

History:  a ─ b ─ c ─ d ─ e   (d is the bad commit)

OPTION 1: git reset --hard c  (if e was only a local commit)
  Result:   a ─ b ─ c          (d and e erased)
  d is gone. e is gone.
  ✅ Clean history
  ❌ e is lost too (if it had good work)
  ❌ ONLY if d and e haven't been pushed

OPTION 2: git revert d
  Result:   a ─ b ─ c ─ d ─ e ─ d'
  d' undoes d's changes. d still exists.
  ✅ Safe for shared/pushed branches
  ✅ e is preserved
  ✅ Full audit trail
  ❌ Slightly messier history
```

---

## 🔑 `git reflog` — Your Safety Net

`git reflog` records every movement of HEAD — even after `git reset --hard`. It's your undo for the undo:

```bash
# If you reset --hard and regret it:
git reflog
# Output:
# e5f6g7 HEAD@{0}: reset: moving to HEAD~2
# a3f8c12 HEAD@{1}: commit: Add salary validation   ← this is what you want
# b7e2d01 HEAD@{2}: commit: Fix pager bug

# Recover the lost commits:
git reset --hard a3f8c12    # go back to where you were before the reset

# OR create a new branch at the lost commit:
git checkout -b recovery/my-lost-work a3f8c12
```

> Reflog entries expire after ~90 days. Act quickly after an accidental hard reset.

---

## 📊 Reset vs Revert — Decision Guide

| Situation                                     | Command                                    |
| --------------------------------------------- | ------------------------------------------ |
| Unstage a file                                | `git restore --staged <file>`            |
| Undo last commit, keep work staged            | `git reset --soft HEAD~1`                |
| Undo last commit, keep work unstaged          | `git reset HEAD~1`                       |
| Discard last commit completely (LOCAL only)   | `git reset --hard HEAD~1`                |
| Undo a commit already pushed to shared branch | `git revert <hash>`                      |
| Squash last 3 commits into one                | `git reset --soft HEAD~3`+`git commit` |
| Production hotfix — undo a pushed bad commit | `git revert <hash>`+`git push`         |

---

## ⚠️ The Golden Rules

```
RULE 1: Never git reset --hard on a shared branch
  → Destroys commits others may have pulled
  → Use git revert instead

RULE 2: Never git reset on pushed commits (unless you're alone on the branch)
  → If someone pulled, their history diverges from yours
  → Causes force-push issues and lost work

RULE 3: Always check git status and git log before reset --hard
  → Once done, recovery requires reflog (time-limited)

RULE 4: Use git reflog if you made a mistake
  → It records every HEAD movement for ~90 days
  → Recovers commits thought to be lost
```

---

## ❓ Interview Questions

**Q: What is the difference between `git reset` and `git revert`?**

> `git reset` moves the branch pointer backward, effectively rewriting history — the removed commits are gone from the branch. It's only safe for local commits not yet pushed. `git revert` creates a new commit that undoes a previous one, preserving the full history — it's safe for shared branches because it only adds to history, never removes.

**Q: What is the difference between `git reset --soft`, `--mixed`, and `--hard`?**

> All three move the branch pointer back. `--soft` keeps the undone changes in the staging area. `--mixed` (default) puts them back in the working directory unstaged. `--hard` discards them entirely — the working directory is reset to match the target commit. `--hard` is destructive and should be used with caution.

**Q: How do you recover from an accidental `git reset --hard`?**

> Use `git reflog` — it records every movement of HEAD for ~90 days. Find the hash of the commit you lost, then `git reset --hard <hash>` to go back to it, or `git checkout -b recovery/branch <hash>` to create a new branch at that point.

**Q: When should you use `git revert` instead of `git reset`?**

> Always when the commit has been pushed to a shared branch. `git revert` is the only safe way to undo pushed commits — it adds a new "undo" commit without rewriting history. Other team members who already pulled won't have conflicts.
>
