
# 03 — Rebasing

---

## 🎯 One-Line Definition

> **Rebase moves your branch's commits to start from a different base commit — instead of merging (which creates a merge commit), it replays your commits on top of another branch, creating a perfectly linear history.**

---

## 🔑 Merge vs Rebase — The Same Goal, Different History

```
STARTING POINT (both main and feature have new commits):
  main:    a─b─x─y
  feature: a─b─c─d

AFTER git merge main (into feature):
  feature: a─b─c─d─M
                ╲ ╱
                 x─y (from main)
  History has a merge commit — you can see the branches

AFTER git rebase main (feature onto main):
  feature: a─b─x─y─c'─d'
  History looks LINEAR — as if you built on top of main the whole time
  c' and d' are NEW commits (same changes, different parent)
```

---

## 🔑 How to Rebase

```bash
# ── Rebase your feature onto main (most common use) ────────
git checkout feature/excel-export
git rebase main
# Replays your feature commits on top of main's latest commit

# ── Then merge (now it's a fast-forward) ───────────────────
git checkout main
git merge feature/excel-export   # clean fast-forward, no merge commit

# ── Rebase a specific branch ───────────────────────────────
git rebase main feature/excel-export
# No need to checkout first — rebase feature onto main
```

What rebase does step by step:

```
1. Finds where feature and main diverged (commit b)
2. Saves your feature commits (c, d) as patches
3. Moves feature pointer to tip of main (y)
4. Replays each saved commit on top:
   - Apply c → creates c' (same changes, new parent = y)
   - Apply d → creates d' (same changes, new parent = c')
5. Moves feature to point at d'
```

---

## 🔑 Interactive Rebase — Rewrite History

Interactive rebase (`-i`) lets you edit, squash, reorder, or delete commits before merging:

```bash
# Rewrite last 3 commits interactively
git rebase -i HEAD~3

# Rewrite all commits since branching from main
git rebase -i main
```

Git opens your editor with:

```
pick a3f8c12 Add export button to grid toolbar
pick b7e2d01 Export button wip
pick c9a1f44 Export button fix typo
pick d4e5f62 Add Excel download logic
pick e5a3b21 Excel download fix

# Commands:
# p, pick   = use commit as-is
# r, reword = use commit but edit the message
# e, edit   = use commit but stop for amending
# s, squash = combine with previous commit (meld into above)
# f, fixup  = like squash but discard this commit's message
# d, drop   = remove commit entirely
```

Common operation — squash WIP commits into one clean commit:

```
pick a3f8c12 Add export button to grid toolbar
s    b7e2d01 Export button wip               ← squash into above
f    c9a1f44 Export button fix typo           ← fixup (discard msg)
pick d4e5f62 Add Excel download logic
f    e5a3b21 Excel download fix               ← fixup

# Result: 2 clean commits instead of 5 messy ones
# a3f8c12 Add export button to grid toolbar
# d4e5f62 Add Excel download logic
```

---

## 🔑 The Golden Rule of Rebase

```
┌──────────────────────────────────────────────────────────────┐
│  NEVER rebase commits that have been pushed to a            │
│  SHARED remote branch.                                      │
│                                                              │
│  WHY: Rebase creates NEW commits (c', d') with new hashes.  │
│  The old commits (c, d) still exist on the remote.          │
│  Your teammates have c and d. You now have c' and d'.       │
│  When they pull: Git sees diverged history → chaos.        │
│                                                              │
│  SAFE to rebase:                                            │
│  ✅ Your own LOCAL feature branch (not pushed yet)          │
│  ✅ A branch only YOU are working on                        │
│                                                              │
│  NEVER rebase:                                              │
│  ❌ main branch                                             │
│  ❌ Any shared branch others have pulled                    │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔑 Handling Rebase Conflicts

```bash
git rebase main
# CONFLICT (content): Merge conflict in EmployeeController.cs

# Fix the conflict in the file...
git add EmployeeController.cs

# Continue the rebase (apply the next commit)
git rebase --continue

# If there are more conflicting commits, repeat:
# fix → git add → git rebase --continue

# Abort the rebase (go back to where you started)
git rebase --abort
```

---

## 🔑 Merge vs Rebase — When to Use Which

```
USE MERGE when:
  ✅ Integrating a feature into main (final merge)
  ✅ You want to preserve the exact history of when/where branches diverged
  ✅ Working on a shared branch others may have pulled
  ✅ Team convention uses merge

USE REBASE when:
  ✅ Updating your local feature branch with latest main changes
  ✅ Cleaning up messy WIP commits before a pull request (interactive)
  ✅ You want a linear, readable history
  ✅ Your branch is local only (not yet pushed or pushed only by you)
```

---

## ❓ Interview Questions

**Q: What is the difference between merge and rebase?**

> Both integrate changes from one branch into another. Merge creates a merge commit and preserves the true history of when branches diverged. Rebase rewrites history by replaying commits on a new base, creating a linear history with no merge commit.

**Q: What is the Golden Rule of Rebase?**

> Never rebase commits that have already been pushed to a shared remote branch. Rebase creates new commits with new hashes — if others have pulled the old commits, their history diverges from yours, causing serious problems.

**Q: What is interactive rebase used for?**

> `git rebase -i` lets you rewrite commit history before merging — squash multiple WIP commits into one clean commit, reword messages, reorder commits, or drop commits entirely. Used to create a clean, professional commit history before a pull request.
>
