
# 02 — Merging

---

## 🎯 One-Line Definition

> **Merging combines the work from one branch into another — Git figures out what each branch added and combines them, creating a merge commit when the histories have diverged.**

---

## 🔑 The Two Merge Types

### Fast-Forward Merge (no merge commit)

Happens when the target branch has no new commits since you branched off — Git just moves the pointer forward:

```
Before:
  main:    a─b         ← no new commits on main
  feature: a─b─c─d    ← feature has new commits

git checkout main
git merge feature

After (fast-forward):
  main:    a─b─c─d    ← pointer just moved forward
  No merge commit created
```

### True Merge (creates a merge commit)

Happens when both branches have new commits since diverging:

```
Before:
  main:    a─b─x─y    ← main has new commits (x, y)
  feature: a─b─c─d    ← feature also has new commits (c, d)

git checkout main
git merge feature

After (merge commit M):
  main:    a─b─x─y─M  ← M is the merge commit
                 ╱
  feature: a─b─c─d
  M has TWO parents: y and d
```

---

## 🔑 How to Merge

```bash
# Standard workflow:
# 1. Finish your feature
git checkout feature/excel-export
git add . && git commit -m "Complete Excel export feature"

# 2. Switch to the target branch
git checkout main

# 3. Pull latest (don't merge outdated main)
git pull

# 4. Merge
git merge feature/excel-export

# 5. Push
git push
```

---

## 🔑 Merge Options

```bash
# ── Standard merge ─────────────────────────────────────────
git merge feature/excel-export
# Fast-forward if possible, merge commit if branches diverged

# ── Force a merge commit (even if fast-forward is possible) ─
git merge --no-ff feature/excel-export
# Creates a merge commit always
# Useful for keeping feature history grouped together
# Many teams use this so you can see "this was feature X"

# ── Squash merge — combine all feature commits into one ────
git merge --squash feature/excel-export
git commit -m "Add Excel export feature"
# All feature commits become ONE clean commit on main
# Feature branch history is not preserved in main
# Feature branch NOT marked as merged — delete it manually

# ── Abort a merge in progress ──────────────────────────────
git merge --abort
# Use when you started a merge but want to cancel
# (only works before the merge is committed)
```

---

## 🔑 Merge Conflicts — When and Why

Conflicts happen when BOTH branches modified the same lines of the same file:

```
main branch has EmployeeController.cs:
  public JsonResult Read([DataSourceRequest] DataSourceRequest request)
  {
      var result = _db.Employees.Where(e => e.IsActive).ToDataSourceResult(request);
      return Json(result);
  }

feature branch has same method (different change):
  public JsonResult Read([DataSourceRequest] DataSourceRequest request)
  {
      var result = _db.Employees.OrderBy(e => e.Name).ToDataSourceResult(request);
      return Json(result);
  }

Git doesn't know which LINQ change you want → CONFLICT
```

---

## 🔑 Resolving Merge Conflicts

```bash
git merge feature/excel-export
# CONFLICT (content): Merge conflict in Controllers/EmployeeController.cs
# Automatic merge failed; fix conflicts and then commit the result.

git status
# Both modified: Controllers/EmployeeController.cs
```

Open the file — Git marks conflicts:

```csharp
<<<<<<< HEAD
    var result = _db.Employees.Where(e => e.IsActive).ToDataSourceResult(request);
=======
    var result = _db.Employees.OrderBy(e => e.Name).ToDataSourceResult(request);
>>>>>>> feature/excel-export
```

```
<<<<<<< HEAD     = your current branch (main) version
=======          = separator
>>>>>>> feature  = the branch being merged in
```

You must manually edit to the correct result:

```csharp
// Resolved: keep BOTH changes
var result = _db.Employees
                .Where(e => e.IsActive)
                .OrderBy(e => e.Name)
                .ToDataSourceResult(request);
```

Then:

```bash
# After editing all conflict files:
git add Controllers/EmployeeController.cs
git commit   # Git creates the merge commit with your resolution
# OR: git commit -m "Merge feature/excel-export — keep both active filter and name sort"
```

---

## 🔑 Preventing/Minimising Conflicts

```bash
# 1. Keep feature branches short-lived (hours/days, not weeks)
# 2. Pull main into your feature branch regularly:
git checkout feature/excel-export
git merge main    # bring main's changes into your feature
# Resolve any conflicts here (smaller, easier)
# When you merge back: no conflicts (already resolved)

# 3. Communicate with team about which files each person is changing
# 4. Write small, focused commits (easier to resolve)
```

---

## 📊 Merge Types Quick Reference

| Type          | Command                         | Creates Merge Commit | Use For                              |
| ------------- | ------------------------------- | -------------------- | ------------------------------------ |
| Fast-forward  | `git merge <branch>`          | ❌ No                | Short-lived, no divergence           |
| Regular merge | `git merge <branch>`          | ✅ Yes (if diverged) | Standard merge                       |
| No-FF merge   | `git merge --no-ff <branch>`  | ✅ Always            | Keep feature grouped in history      |
| Squash merge  | `git merge --squash <branch>` | ✅ One clean commit  | Clean main history, many WIP commits |

---

## ❓ Interview Questions

**Q: What is a fast-forward merge?**

> When the target branch has no new commits since the feature branch was created, Git simply moves the branch pointer forward to the feature's latest commit. No merge commit is created — the history looks linear.

**Q: What causes a merge conflict?**

> When both branches modified the same lines in the same file. Git can automatically merge changes to different parts of a file but can't decide which version of the same line to keep — that requires human judgment.

**Q: What is the difference between `git merge --no-ff` and a regular merge?**

> A regular merge does a fast-forward if possible (no merge commit, linear history). `--no-ff` always creates a merge commit even when fast-forward is possible. This preserves a clear record in history that "these commits were a feature" — makes the feature visible as a group in `git log --graph`.
>
