
# 03 — Status, Log, Diff

---

## 🎯 One-Line Definition

> **`git status` shows where you are right now, `git log` shows the history of commits, and `git diff` shows exactly what changed line by line — together they answer: "what's going on in this repo?"**

---

## 🔑 `git status` — What's Going On Right Now

```bash
git status
```

Output breakdown:

```
On branch main                     ← current branch
Your branch is up to date with 'origin/main'.  ← vs remote

Changes to be committed:           ← STAGED (will go in next commit)
  (use "git restore --staged <file>..." to unstage)
        modified:   Controllers/EmployeeController.cs
        new file:   Models/EmployeeDto.cs

Changes not staged for commit:     ← MODIFIED but not staged
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes)
        modified:   wwwroot/css/site.css

Untracked files:                   ← Git doesn't know these exist
  (use "git add <file>..." to include in what will be committed)
        appsettings.Development.json
```

Short status — one line per file:

```bash
git status -s
# Output:
# M  Controllers/EmployeeController.cs   ← modified, staged
#  M wwwroot/css/site.css                ← modified, NOT staged
# ?? appsettings.Development.json        ← untracked
# A  Models/EmployeeDto.cs               ← new file, staged

# Left column = staging area status
# Right column = working directory status
# M = modified   A = added   D = deleted   ? = untracked
```

---

## 🔑 `git log` — View Commit History

```bash
# Default log — full details
git log

# Output:
commit a3f8c12b9e4d5f6a7b8c9d0e1f2a3b4c5d6e7f8
Author: Alice Johnson <alice@company.com>
Date:   Mon Jun 15 09:30:22 2024 +0530

    Add salary validation to employee form

    - Added [Range] annotation to Employee model
    - Server returns ModelState errors to Kendo Grid
    - Grid popup shows field-level error messages

commit b7e2d014f5e6a7b8c9d0e1f2a3b4c5d6e7f8a9
Author: Bob Smith <bob@company.com>
Date:   Fri Jun 12 14:15:01 2024 +0530

    Fix employee grid not loading after department filter
```

---

## 🔑 `git log` — Useful Formats

```bash
# ── One line per commit — fastest overview ─────────────────
git log --oneline

# Output:
a3f8c12 Add salary validation to employee form
b7e2d01 Fix employee grid not loading after department filter
c9a1f44 Add inline editing to Kendo Grid
d4e5f62 Setup Kendo UI DataSource transport
e5a3b21 Initial ASP.NET Core project

# ── With branch/merge graph ────────────────────────────────
git log --oneline --graph --all

# Output:
* a3f8c12 (HEAD -> main) Add salary validation
| * f1e2d3c (feature/excel-export) Add Excel export
|/
* b7e2d01 Fix grid loading bug
* c9a1f44 Add inline editing

# ── Limit number of commits ────────────────────────────────
git log -5                   # last 5 commits
git log --oneline -10        # last 10, one line each

# ── Filter by author ───────────────────────────────────────
git log --author="Alice"
git log --author="alice@company.com"

# ── Filter by date ─────────────────────────────────────────
git log --since="2024-06-01"
git log --until="2024-06-15"
git log --since="1 week ago"
git log --since="yesterday"

# ── Filter by commit message content ──────────────────────
git log --grep="Kendo"       # commits mentioning "Kendo"
git log --grep="fix"         # commits with "fix" in message

# ── Show what files changed in each commit ─────────────────
git log --stat               # shows changed files + line counts
git log --name-only          # shows only changed file names

# ── Show changes for a specific file ──────────────────────
git log -- Controllers/EmployeeController.cs
git log --follow -- Controllers/EmployeeController.cs
# --follow = tracks the file even if it was renamed

# ── Formatted output ───────────────────────────────────────
git log --format="%h | %an | %ar | %s"
# Output: a3f8c12 | Alice Johnson | 2 hours ago | Add salary validation
# %h = short hash   %an = author name   %ar = relative date   %s = subject
```

---

## 🔑 `git diff` — See Exactly What Changed

```bash
# ── Working directory: unstaged changes ────────────────────
git diff
# Shows changes NOT yet staged (modified but not added)

# ── Staging area: staged but not committed ─────────────────
git diff --staged
git diff --cached    # same thing
# Shows changes that WILL go into next commit

# ── Specific file ──────────────────────────────────────────
git diff EmployeeController.cs
git diff --staged EmployeeController.cs

# ── Between two commits ─────────────────────────────────────
git diff a3f8c12 b7e2d01
git diff HEAD~2 HEAD          # current vs 2 commits ago
git diff HEAD~1               # current vs previous commit

# ── Between branches ────────────────────────────────────────
git diff main feature/excel-export
git diff main..feature/excel-export   # same

# ── Summary: just file names, not content ──────────────────
git diff --name-only
git diff --name-status        # shows M/A/D before filename
```

---

## 🔑 Reading a Diff

```diff
diff --git a/Controllers/EmployeeController.cs b/Controllers/EmployeeController.cs
index 4a1b2c3..8d9e0f1 100644
--- a/Controllers/EmployeeController.cs      ← "a" = before
+++ b/Controllers/EmployeeController.cs      ← "b" = after

@@ -45,8 +45,15 @@ public class EmployeeController : Controller
 // context lines (unchanged, shown for reference)
         var employees = _db.Employees.AsQueryable();

-        return Json(employees.ToDataSourceResult(request));
+        if (ModelState.IsValid)
+        {
+            _db.Employees.Add(employee);
+            _db.SaveChanges();
+        }
+        return Json(new[] { employee }.ToDataSourceResult(request, ModelState));

 // context lines
```

```
Lines starting with:
  (nothing) = unchanged context line
  -          = REMOVED (was in old, gone in new)
  +          = ADDED (not in old, now in new)

@@ -45,8 +45,15 @@
   -45,8  = "in the OLD file, starting at line 45, showing 8 lines"
   +45,15 = "in the NEW file, starting at line 45, showing 15 lines"
```

---

## 🔑 Viewing a Specific Commit

```bash
# Show what changed in a specific commit
git show a3f8c12

# Show the contents of a file at a specific commit
git show a3f8c12:Controllers/EmployeeController.cs

# Show the last commit
git show HEAD

# Show the commit 2 back
git show HEAD~2
```

---

## 📊 Quick Reference

| Command                             | What It Shows                              |
| ----------------------------------- | ------------------------------------------ |
| `git status`                      | Current state — staged/modified/untracked |
| `git status -s`                   | Same but one line per file                 |
| `git log`                         | Full commit history                        |
| `git log --oneline`               | One line per commit                        |
| `git log --oneline --graph --all` | Visual branch graph                        |
| `git log --author="Name"`         | Commits by one author                      |
| `git log --grep="text"`           | Commits with text in message               |
| `git log -- <file>`               | History of one file                        |
| `git diff`                        | Unstaged changes                           |
| `git diff --staged`               | Staged changes                             |
| `git diff HEAD~1`                 | Vs previous commit                         |
| `git diff main..feature`          | Branch differences                         |
| `git show <hash>`                 | Full details of one commit                 |

---

## ❓ Interview Questions

**Q: What is the difference between `git diff` and `git diff --staged`?**

> `git diff` shows changes in your working directory that are NOT yet staged — what you've modified but haven't added. `git diff --staged` shows changes that ARE staged and will go into the next commit.

**Q: How do you see a one-line history of all commits including branches?**

> `git log --oneline --graph --all` — `--oneline` compresses each commit to one line, `--graph` draws the branch/merge lines, `--all` includes all branches not just the current one.

**Q: How do you see what a specific old commit changed?**

> `git show <hash>` shows the commit metadata and all the diffs for that commit. Use `git show <hash>:<filepath>` to see the complete contents of a specific file at that commit.
>
