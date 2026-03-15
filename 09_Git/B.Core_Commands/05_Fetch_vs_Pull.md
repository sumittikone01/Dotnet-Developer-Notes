
# 05 — Fetch vs Pull

---

## 🎯 One-Line Definition

> **`git fetch` downloads remote commits into your local repo but does NOT touch your working branch — `git pull` does fetch AND immediately merges the result into your current branch.**

---

## 🔑 The Core Difference — One Visual

```
BEFORE (you're behind by 2 commits):
  local  main:      a─b
  remote main:      a─b─c─d

AFTER git fetch:
  local  main:      a─b           ← YOUR branch unchanged
  origin/main:      a─b─c─d      ← tracking branch updated
  Working directory: unchanged

AFTER git pull (= fetch + merge):
  local  main:      a─b─c─d      ← YOUR branch moved forward
  origin/main:      a─b─c─d      ← same
  Working directory: updated with c and d's changes
```

---

## 🔑 `git fetch` — Download Without Merging

```bash
# Fetch all branches from origin
git fetch

# Fetch a specific remote
git fetch origin

# Fetch a specific branch
git fetch origin main
git fetch origin feature/excel-export

# Fetch all remotes (if you have multiple)
git fetch --all

# Fetch and prune deleted remote branches
git fetch --prune
git fetch -p      # short form
# Removes local tracking branches for remote branches that no longer exist
```

After fetch — you inspect before merging:

```bash
git fetch

# See what came in
git log HEAD..origin/main --oneline
# Shows: commits on remote that you don't have yet

# See the diff
git diff HEAD origin/main

# Now decide to merge
git merge origin/main
# OR rebase
git rebase origin/main
```

---

## 🔑 When to Use Fetch Instead of Pull

```
USE git fetch when:
  ✅ You want to SEE remote changes before integrating them
  ✅ You're not ready to merge right now
  ✅ You want to inspect: "what did my team push?"
  ✅ You're on a commit or detached HEAD (merge would be confusing)
  ✅ You want to cherry-pick specific commits from remote

USE git pull when:
  ✅ You want to update your branch immediately (most common)
  ✅ You know the remote is clean (no surprises expected)
  ✅ Quick sync before starting work in the morning
```

---

## 🔑 Fetch + Inspect + Merge — The Safe Pattern

```bash
# Morning routine — safe way to start the day

# Step 1: Download what the team pushed overnight
git fetch

# Step 2: See what changed
git log HEAD..origin/main --oneline
# a3f8c12 Alice: Add employee export button
# b7e2d01 Bob:   Fix salary validation range

# Step 3: See the actual code diff
git diff HEAD origin/main

# Step 4: Decide — merge or rebase
git merge origin/main       # merge (creates merge commit if diverged)
# OR
git rebase origin/main      # rebase (replays your commits on top, cleaner)
```

---

## 🔑 Remote Tracking Branches After Fetch

```bash
# See all branches — local and remote tracking
git branch -a

# Output:
  * main                          ← current local branch (*)
    feature/excel-export          ← other local branch
    remotes/origin/main           ← remote tracking branch
    remotes/origin/feature/export ← remote tracking branch

# Remote tracking branches are read-only views of the remote
# They update ONLY when you fetch
```

---

## 🔑 Pruning Stale Remote Tracking Branches

When a remote branch is deleted (e.g., after a pull request is merged):

```bash
# Without pruning — stale branches accumulate
git branch -a
# remotes/origin/feature/old-thing    ← deleted on remote, still shows locally
# remotes/origin/feature/another-old  ← same

# Prune: remove local tracking branches for deleted remote branches
git fetch --prune
git fetch -p

# OR set this globally so every fetch auto-prunes
git config --global fetch.prune true
```

---

## 📊 Fetch vs Pull Quick Reference

|                           | `git fetch`             | `git pull`        |
| ------------------------- | ------------------------- | ------------------- |
| Downloads remote commits  | ✅ Yes                    | ✅ Yes              |
| Updates local branch      | ❌ No                     | ✅ Yes              |
| Changes working directory | ❌ No                     | ✅ Yes              |
| Risk of conflicts         | ❌ None                   | ✅ Possible         |
| Good for:                 | Inspecting before merging | Quick sync          |
| Equivalent to             | `fetch`only             | `fetch`+`merge` |

---

## ❓ Interview Questions

**Q: What is the difference between `git fetch` and `git pull`?**

> `git fetch` downloads commits from the remote and updates remote tracking branches (like `origin/main`) but leaves your local branch unchanged. `git pull` does the same download AND immediately merges the remote changes into your current local branch. `git pull` = `git fetch` + `git merge`.

**Q: When would you use `git fetch` over `git pull`?**

> When you want to see what changed on the remote before deciding whether to merge. Useful in the morning to check what teammates pushed overnight, or when you want to inspect remote changes and cherry-pick specific commits rather than merging everything.

**Q: What does `git fetch --prune` do?**

> It removes local remote-tracking branches for remote branches that have been deleted. For example, if a feature branch was merged and deleted on GitHub, `fetch --prune` cleans up the stale `remotes/origin/feature/xxx` reference locally.
>
