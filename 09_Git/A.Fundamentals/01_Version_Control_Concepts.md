# 01 — Version Control Concepts

---

## 🎯 One-Line Definition

> **Version control is a system that records every change you make to your code over time — so you can see the full history, go back to any previous state, and collaborate with others without overwriting each other's work.**

---

## 🤔 The Problem Without Version Control

Imagine you're working on an employee management system alone:

```
WITHOUT version control — what actually happens:
────────────────────────────────────────────────────────────
EmployeeController.cs          ← your working file
EmployeeController_backup.cs   ← "just in case"
EmployeeController_v2.cs       ← "new approach"
EmployeeController_final.cs    ← "definitely final"
EmployeeController_final2.cs   ← "ok THIS is final"
EmployeeController_ACTUAL_FINAL_USE_THIS.cs

Problems:
  ✗ Which file is the real one?
  ✗ What changed between v2 and final?
  ✗ You broke something in final — how do you get v2 back?
  ✗ Your colleague edited the same file — whose version wins?
```

---

## 🔑 What Version Control Solves

```
WITH version control (Git):
────────────────────────────────────────────────────────────
EmployeeController.cs          ← ONE file, always current

But behind the scenes Git stores every snapshot:

  Commit 1: "Initial employee grid"
  Commit 2: "Added inline editing"
  Commit 3: "Fixed salary validation"  ← you're here now
  Commit 4: "Added export to Excel"

You can:
  ✓ See exactly what changed in each commit
  ✓ Jump back to Commit 1 if needed
  ✓ See WHO changed WHAT and WHEN
  ✓ Work on features in isolation (branches)
  ✓ Merge everyone's changes together safely
```

---

## 🔑 The 3 Core Benefits

### Benefit 1 — History

Every change is recorded with: what changed, who changed it, when, and why (commit message).

```
git log:
  a3f8c12  Alice   "Added salary validation to employee form"  3 hours ago
  b7e2d01  Bob     "Fixed grid paging bug on page > 3"         Yesterday
  c9a1f44  Alice   "Initial Kendo Grid setup"                  2 days ago
```

### Benefit 2 — Safety (Undo)

Nothing is ever truly lost. Any state can be recovered.

```
"I deleted the whole DAL folder by accident" → git checkout HEAD -- DAL/
"This feature broke everything" → git revert to last working commit
"What did this file look like last week?" → git show HEAD~7:file.cs
```

### Benefit 3 — Collaboration

Multiple developers work simultaneously without overwriting each other.

```
Alice works on:  feature/employee-export  (her own branch)
Bob works on:    feature/salary-report    (his own branch)
Carol works on:  bugfix/grid-pager        (her own branch)

All three work in parallel.
All three merge into main when ready.
No one steps on anyone else.
```

---

## 🔑 Key Terminology You Must Know

```
┌─────────────────────────────────────────────────────────────────┐
│  REPOSITORY (repo)                                               │
│  The folder where Git stores your project + all its history.    │
│  Has a hidden .git/ subfolder that IS the version control.      │
│                                                                  │
│  COMMIT                                                          │
│  A snapshot of your project at a specific point in time.        │
│  Like a save point in a video game. Has a unique ID (hash).     │
│                                                                  │
│  BRANCH                                                          │
│  An independent line of development. A branch is just a         │
│  pointer to a specific commit. Default branch is "main"         │
│  (or "master" in older repos).                                  │
│                                                                  │
│  WORKING DIRECTORY                                               │
│  The actual files you see and edit on your computer.            │
│                                                                  │
│  STAGING AREA (Index)                                            │
│  A preparation zone. You add changes here before committing.    │
│  Lets you choose WHICH changes go into the next commit.         │
│                                                                  │
│  REMOTE                                                          │
│  A copy of the repo stored elsewhere (GitHub, Azure DevOps).    │
│  "origin" is the conventional name for your main remote.        │
│                                                                  │
│  CLONE                                                           │
│  Copying a remote repo to your local machine for the first time.│
│                                                                  │
│  PUSH                                                            │
│  Sending your local commits to the remote repo.                 │
│                                                                  │
│  PULL                                                            │
│  Fetching commits from remote AND merging them into your branch.│
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 The Three Areas of Git

This is the most important mental model in Git:

```
┌──────────────────┐   git add    ┌──────────────────┐   git commit  ┌──────────────┐
│  WORKING          │ ──────────► │  STAGING AREA     │ ────────────► │  REPOSITORY  │
│  DIRECTORY        │             │  (Index)          │               │  (.git/)     │
│                   │             │                   │               │              │
│  Your actual      │             │  Changes chosen   │               │  Permanent   │
│  files on disk    │             │  for next commit  │               │  history of  │
│                   │             │                   │               │  snapshots   │
│  You edit here    │             │  git add picks    │               │              │
└──────────────────┘             └──────────────────┘               └──────────────┘

  ◄──────────────────────────── git checkout ─────────────────────────────────────
  ◄──────────────────────────── git restore  ─────────────────────────────────────
```

**Real example:**

```
You edit 3 files: EmployeeController.cs, Employee.cs, site.css

git add EmployeeController.cs Employee.cs   ← stage only the C# files
git commit -m "Add employee validation"     ← commit only those 2 files

site.css changes stay in Working Directory  ← not committed yet
```

---

## 🔑 What a Commit Actually Is

A commit is NOT a diff — it is a **complete snapshot** of all tracked files.

```
Commit a3f8c12:
  ├── EmployeeController.cs   (complete content at this point)
  ├── Employee.cs              (complete content at this point)
  ├── Program.cs               (complete content at this point)
  └── ... all other files ...  (complete content at this point)

  Metadata:
    Author:  Alice <alice@company.com>
    Date:    Mon Jun 15 09:30:00 2024
    Message: "Fixed salary validation bug"
    Parent:  b7e2d01  (the previous commit)
```

Git is space-efficient — unchanged files are stored as references, not duplicated.

---

## 🔑 File States in Git

A file in your working directory can be in one of these states:

```
UNTRACKED     → Git doesn't know about this file yet
                (new file you haven't run git add on)

TRACKED:
  UNMODIFIED  → tracked, and not changed since last commit
  MODIFIED    → tracked, and changed since last commit (not staged)
  STAGED      → modified and added to staging area (ready to commit)
  COMMITTED   → in the repository history

Flow:
  New file  →  git add  →  STAGED  →  git commit  →  COMMITTED
  Edit file →             MODIFIED → git add →  STAGED  →  git commit  →  COMMITTED
```

---

## 🔑 Why Every Developer Must Know This

```
In your daily .NET work:

  Without Git:
    "I broke the Employee DAL" → you're in trouble
    "Bob and I both edited EmployeeController.cs" → someone's work is lost
    "Which version is in production?" → no way to know

  With Git:
    "I broke the Employee DAL" → git checkout HEAD -- DAL/ → fixed
    "Bob and I both edited EmployeeController.cs" → git merge → both changes kept
    "Which version is in production?" → git log → see every deployment
    "Review my changes before merge" → pull request → team reviews code
```

---

## ❓ Interview Questions

**Q: What is version control and why is it used?**

> A system that tracks every change to code over time. Used to maintain history, enable safe rollback to any previous state, and allow multiple developers to collaborate without overwriting each other's work.

**Q: What is the difference between the Working Directory, Staging Area, and Repository in Git?**

> Working Directory is where you edit files. Staging Area (index) is where you prepare specific changes for the next commit using `git add`. Repository is the permanent history of all commits stored in `.git/`. Changes move: Working → Staging → Repository.

**Q: What is a commit?**

> A snapshot of all tracked files at a specific point in time. Every commit has a unique hash ID, stores the author, date, message, and a reference to the parent commit, forming a chain of history.

**Q: What is the difference between a local and remote repository?**

> A local repository lives on your computer in the `.git/` folder. A remote repository (like on GitHub or Azure DevOps) is a shared copy accessible by the whole team. `push` sends your commits to remote; `pull` brings remote commits to your local repo.
>
