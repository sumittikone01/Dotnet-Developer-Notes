
# 01 — Resolving Merge Conflicts

---

## 🎯 One-Line Definition

> **A merge conflict happens when two branches changed the same lines — Git marks the conflict in the file, you manually pick the correct version, stage the file, and complete the merge.**

---

## 🔑 When Conflicts Happen

```
Conflicts ONLY happen when:
  ✅ Both branches modified the SAME LINE(S) in the SAME FILE

Conflicts do NOT happen when:
  ❌ Different files were changed (auto-merged)
  ❌ Same file but different lines (auto-merged)
  ❌ One branch added a new file (no conflict)
  ❌ One branch deleted a file the other didn't touch
```

---

## 🔑 Anatomy of a Conflict Marker

```csharp
<<<<<<< HEAD
    var result = _db.Employees.Where(e => e.IsActive)
                               .ToDataSourceResult(request);
=======
    var result = _db.Employees.OrderBy(e => e.Name)
                               .ToDataSourceResult(request);
>>>>>>> feature/excel-export
```

```
<<<<<<< HEAD         = start of YOUR version (current branch)
=======              = separator
>>>>>>> feature/...  = start of THEIR version (branch being merged)
```

Everything between `<<<<<<<` and `=======` is what YOU have.
Everything between `=======` and `>>>>>>>` is what THEY have.
**You delete all three markers and write the correct final version.**

---

## 🔑 Step-by-Step: Resolving a Conflict

```bash
# Step 1: Start merge
git checkout main
git merge feature/excel-export

# Output:
CONFLICT (content): Merge conflict in Controllers/EmployeeController.cs
Automatic merge failed; fix conflicts and then commit the result.

# Step 2: See which files have conflicts
git status
# Both modified: Controllers/EmployeeController.cs

# Step 3: Open the file and find the markers
# Edit to the correct final version:

# BEFORE (conflict markers in file):
#   <<<<<<< HEAD
#       .Where(e => e.IsActive)
#   =======
#       .OrderBy(e => e.Name)
#   >>>>>>> feature/excel-export

# AFTER (your resolved version — keep BOTH changes):
    var result = _db.Employees
                    .Where(e => e.IsActive)    // from main
                    .OrderBy(e => e.Name)       // from feature
                    .ToDataSourceResult(request);

# Step 4: Stage the resolved file
git add Controllers/EmployeeController.cs

# Step 5: Check all conflicts are resolved
git status
# All conflicts fixed but you are still merging.
# (use "git commit" to conclude merge)

# Step 6: Commit to complete the merge
git commit
# OR with a message:
git commit -m "Merge feature/excel-export — keep active filter + name sort"
```

---

## 🔑 The Three Outcomes When Resolving

```
OUTCOME 1: Keep YOUR version only (HEAD)
  Delete everything from ======= to >>>>>>> feature
  Delete the conflict markers
  Result: only your code remains

OUTCOME 2: Keep THEIR version only (incoming)
  Delete everything from <<<<<<< HEAD to =======
  Delete the conflict markers
  Result: only their code remains

OUTCOME 3: Keep BOTH (most common in code)
  Combine both versions logically
  Delete all three marker lines
  Result: both changes coexist

OUTCOME 4: Write something new entirely
  Delete both versions and all markers
  Write the correct solution from scratch
  Happens when neither version is right individually
```

---

## 🔑 Using VS Code to Resolve Conflicts

VS Code shows conflicts with helpful buttons:

```
When you open a file with conflicts in VS Code:

  [Accept Current Change]  → keep HEAD version
  [Accept Incoming Change] → keep theirs
  [Accept Both Changes]    → keep both (appended)
  [Compare Changes]        → side-by-side diff

These buttons appear right above each conflict marker.
Much easier than editing markers manually.
```

---

## 🔑 Preventing Conflicts — Best Practices

```
1. Keep feature branches short-lived
   Long-running branches = more divergence = more conflicts

2. Pull/merge main into your branch regularly
   git checkout feature/mine
   git merge main    ← do this every day or two
   Resolve small conflicts early rather than big ones at the end

3. Communicate with teammates
   "I'm refactoring EmployeeController today"
   → others know to avoid that file

4. Make small, focused commits
   Smaller changes = smaller conflict areas

5. Avoid large refactors on shared files
   Or do them in a dedicated PR that merges first
```

---

## ❓ Interview Questions

**Q: What causes a merge conflict?**

> When two branches both modified the same lines in the same file. Git can auto-merge changes to different parts of a file, but when both branches changed the exact same lines, it can't decide which version to keep — you must resolve it manually.

**Q: How do you complete a merge after resolving conflicts?**

> Edit the file to the correct final version, removing all conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`). Then `git add <file>` to mark it resolved, and `git commit` to complete the merge.

**Q: How can you minimise merge conflicts on a team?**

> Keep feature branches short-lived (merge frequently), regularly merge `main` into your feature branch to resolve small conflicts early, and coordinate with teammates about which files each person is changing to avoid parallel edits to the same files.
>
