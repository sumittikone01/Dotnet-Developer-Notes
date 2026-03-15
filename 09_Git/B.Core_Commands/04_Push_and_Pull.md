
# 04 — Push and Pull

---

## 🎯 One-Line Definition

> **`git push` sends your local commits to the remote repository — `git pull` fetches remote commits and merges them into your current branch.**

---

## 🔑 The Remote Relationship

```
YOUR MACHINE                          REMOTE (GitHub/Azure DevOps)
─────────────                         ────────────────────────────
local/main      ──── git push ──────► origin/main (remote)
                ◄─── git pull ─────── origin/main (remote)

"origin" = the conventional name for your primary remote
           (set automatically when you clone)
```

---

## 🔑 `git push` — Send Your Commits to Remote

```bash
# ── Standard push — push current branch to its remote ─────
git push

# ── First push of a new branch (sets the upstream) ────────
git push -u origin feature/excel-export
# -u = --set-upstream
# After this: just "git push" works (no need to specify branch)

# ── Push a specific branch ────────────────────────────────
git push origin main
git push origin feature/excel-export

# ── Push ALL branches ──────────────────────────────────────
git push --all origin

# ── Push tags ─────────────────────────────────────────────
git push --tags           # push all local tags to remote
git push origin v1.0.0    # push specific tag
```

---

## 🔑 `git pull` — Get Remote Commits + Merge

```bash
# ── Pull current branch (fetch + merge) ───────────────────
git pull

# ── Pull a specific branch ────────────────────────────────
git pull origin main
git pull origin feature/excel-export

# ── Pull with rebase instead of merge ─────────────────────
git pull --rebase
# Instead of creating a merge commit, replays your commits
# on top of the fetched commits — cleaner linear history
# Many teams use this as standard: git config pull.rebase true

# ── Set default pull strategy ─────────────────────────────
git config --global pull.rebase false   # always merge (default)
git config --global pull.rebase true    # always rebase
```

---

## 🔑 What Happens Under the Hood

```
git pull = git fetch + git merge

STEP 1 — git fetch:
  Downloads new commits from remote
  Updates origin/main pointer locally
  Does NOT touch your local main branch

  Before:  local main = a3f8c12
           origin/main = a3f8c12   (same)

  Teammate pushes b7e2d01, c9a1f44...

  After fetch:
           local main = a3f8c12   (unchanged)
           origin/main = c9a1f44  (updated — reflects remote)

STEP 2 — git merge origin/main:
  Merges origin/main into your local main
  If no conflicts: fast-forward or merge commit
  If conflicts: you must resolve them manually

  After merge:
           local main = c9a1f44   (caught up)
           origin/main = c9a1f44  (same)
```

---

## 🔑 Fast-Forward vs Merge Commit

```
FAST-FORWARD (no divergence — cleanest):
  Remote: a─b─c─d
  Local:  a─b          ← you're behind, no local commits
  After pull: a─b─c─d  ← just moves pointer forward, no merge commit

MERGE COMMIT (diverged — both have new commits):
  Remote: a─b─c
  Local:  a─b─x─y      ← you also made commits since branching
  After pull: a─b─c─x─y─M  ← M is a merge commit joining both lines
```

---

## 🔑 Push Rejected — When and Why

```bash
# Common error:
$ git push
! [rejected] main -> main (non-fast-forward)
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. Integrate the remote changes before pushing.

WHY: Your teammate pushed commits you don't have yet.
     Git won't overwrite their work.

FIX:
  git pull          # get their commits first
  # resolve any conflicts if needed
  git push          # now your push succeeds
```

```
Timeline:
  Start: local=a─b,  remote=a─b
  You commit c,d:    local=a─b─c─d,  remote=a─b
  Teammate pushes x: local=a─b─c─d,  remote=a─b─x
  You try git push → REJECTED (remote has x you don't have)
  git pull → local=a─b─c─d─x─M (merge), remote=a─b─x
  git push → SUCCESS ✅
```

---

## 🔑 Tracking Branches — Check Ahead/Behind

```bash
# See how your branch compares to remote
git status
# Output includes:
# Your branch is ahead of 'origin/main' by 2 commits.
#   (use "git push" to publish your local commits)

# OR:
# Your branch is behind 'origin/main' by 3 commits, and can be fast-forwarded.
#   (use "git pull" to update your local branch)

# OR:
# Your branch and 'origin/main' have diverged,
# and have 2 and 3 different commits each, respectively.

# See exact commit difference
git log origin/main..main      # commits you have that remote doesn't
git log main..origin/main      # commits remote has that you don't
```

---

## 📊 Quick Reference

| Command                         | What It Does                             |
| ------------------------------- | ---------------------------------------- |
| `git push`                    | Push current branch to its remote        |
| `git push -u origin <branch>` | Push + set upstream (first push)         |
| `git push origin <branch>`    | Push specific branch                     |
| `git pull`                    | Fetch + merge remote into current branch |
| `git pull --rebase`           | Fetch + rebase (no merge commit)         |
| `git pull origin <branch>`    | Pull specific branch                     |

---

## ❓ Interview Questions

**Q: What is the difference between `git push` and `git pull`?**

> `git push` sends your local commits to the remote repository. `git pull` downloads remote commits and merges them into your current local branch. Push = outgoing, Pull = incoming.

**Q: Why would `git push` be rejected?**

> The remote has commits you don't have locally. Git won't overwrite teammates' work. Fix: run `git pull` first to integrate the remote changes, resolve any conflicts, then `git push` again.

**Q: What does `git push -u origin feature/my-branch` do?**

> Pushes the branch to remote AND sets the upstream tracking link. After this, `git push` and `git pull` with no arguments know to use `origin/feature/my-branch`. The `-u` flag only needs to be used once on the first push of a new branch.
>
