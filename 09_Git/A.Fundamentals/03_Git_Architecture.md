
# 03 — Git Architecture

---

## 🎯 One-Line Definition

> **Git stores your project as a graph of snapshots — every commit points to its parent, creating a chain of history — and branches are just lightweight pointers to commits that move forward as you commit.**

---

## 🔑 The Four Git Object Types

Everything in Git is stored as one of four object types in `.git/objects/`:

```
┌──────────────────────────────────────────────────────────────┐
│  BLOB         → stores file contents                         │
│  TREE         → stores directory structure (lists blobs)     │
│  COMMIT       → stores a snapshot + metadata                 │
│  TAG          → stores a named reference to a commit         │
└──────────────────────────────────────────────────────────────┘

Every object has a SHA-1 hash (40 hex chars) as its ID:
  a3f8c12b9e...  ← this IS the object's identity
  Change anything inside → completely different hash
  Same content always = same hash (content-addressed storage)
```

---

## 🔑 How a Commit Connects to Files

```
COMMIT a3f8c12
  ├── message: "Add employee validation"
  ├── author:  Alice
  ├── date:    2024-06-15
  ├── parent:  b7e2d01         ← points to previous commit
  └── tree:    f9e3c7a          ← points to the root directory snapshot

TREE f9e3c7a  (root directory)
  ├── blob 4a1b2c3  "Controllers/EmployeeController.cs"
  ├── blob 8d9e0f1  "Models/Employee.cs"
  ├── tree b2c3d4e  "Views/"                ← subdirectory
  └── blob 1e2f3a4  "Program.cs"

BLOB 4a1b2c3  (content of EmployeeController.cs)
  "public class EmployeeController : Controller
   {
     // ... full file content stored here ...
   }"
```

---

## 🔑 The Commit Chain — Git's History

Commits form a linked list — each pointing to its parent:

```
HEAD → main
         │
         ▼
[Commit d4e5f6] "Add export to Excel"
  parent: c3d4e5
         │
         ▼
[Commit c3d4e5] "Fix salary validation"
  parent: b7e2d01
         │
         ▼
[Commit b7e2d01] "Add inline editing"
  parent: a3f8c12
         │
         ▼
[Commit a3f8c12] "Initial Grid setup"
  parent: null  ← root commit, no parent
```

This is an immutable chain. **You never change commits — you only add new ones.**

---

## 🔑 What a Branch Actually Is

```
A branch is just a text file containing a commit hash.
Nothing more.

.git/refs/heads/main     contains: "d4e5f6..."
.git/refs/heads/feature  contains: "c3d4e5..."

That's it. No copy of files. No directory.
Just a 41-byte file pointing to one commit.
```

```
Visualised with two branches:

[a3f8c12] ← [b7e2d01] ← [c3d4e5] ← [d4e5f6]
                                           ↑
                                          main  ← branch pointer
                              ↑
                           feature  ← branch pointer
```

When you commit on `main`, `main` pointer moves forward:

```
[a3f8c12] ← [b7e2d01] ← [c3d4e5] ← [d4e5f6] ← [e5f6g7]
                                                       ↑
                                                      main  ← moved forward
                              ↑
                           feature  ← still here
```

---

## 🔑 HEAD — Where You Are Right Now

`HEAD` is a special pointer that tells Git where you currently are.

```
Normally HEAD points to a branch:
  HEAD → main → d4e5f6

When you commit:
  1. New commit created: e5f6g7 (parent = d4e5f6)
  2. main pointer moves to e5f6g7
  3. HEAD still points to main (which now = e5f6g7)

HEAD → main → e5f6g7  ← everything moved forward
```

**Detached HEAD** — HEAD points directly to a commit, not a branch:

```
git checkout b7e2d01  ← checkout old commit directly

HEAD → b7e2d01  ← detached! not pointing to any branch

If you commit now, the new commit has no branch pointer.
It will be orphaned when you switch branches.
This is fine for exploring, but always create a branch before new work.
```

---

## 🔑 The .git Directory — What's Inside

```
.git/
├── HEAD              ← "ref: refs/heads/main" (where you are)
├── config            ← repo-specific settings (remote URLs, user)
├── index             ← the staging area (binary file)
├── objects/          ← ALL objects: blobs, trees, commits, tags
│   ├── a3/           ← first 2 chars of hash = subfolder
│   │   └── f8c12b... ← rest of hash = filename
│   ├── b7/
│   │   └── e2d01...
│   └── ...
└── refs/
    ├── heads/        ← local branches
    │   ├── main      ← contains "d4e5f6..." (hash of tip commit)
    │   └── feature   ← contains "c3d4e5..."
    ├── remotes/      ← remote tracking branches
    │   └── origin/
    │       ├── main  ← last known state of remote main
    │       └── feature
    └── tags/         ← tag references
```

---

## 🔑 Remote Tracking Branches

When you clone or fetch, Git creates remote tracking branches:

```
Local branches:          Remote tracking branches:
  main                     origin/main   (last known state of remote)
  feature                  origin/feature

origin/main is NOT the same as main.
  origin/main = "what main looked like on remote last time I fetched"
  main        = "my local branch, may have diverged"

git fetch → updates origin/main to match remote
git merge origin/main → merges remote state into your local main
git pull = git fetch + git merge (in one step)
```

---

## 🔑 The Staging Area (Index) — Why It Exists

Most version control systems don't have a staging area. Git has one on purpose:

```
You edited 5 files:
  EmployeeController.cs   → big feature addition
  Employee.cs              → model change for the feature
  site.css                 → quick unrelated styling fix
  appsettings.json         → connection string (don't commit this!)
  README.md                → doc update

Without staging:
  Commit everything or nothing — messy, one giant commit

With staging (git add):
  git add EmployeeController.cs Employee.cs
  git commit -m "Add employee export feature"

  git add site.css
  git commit -m "Fix navigation styling"

  git add README.md
  git commit -m "Update setup documentation"

  # appsettings.json → add to .gitignore, never stage
```

Staging lets you create **clean, focused commits** even from messy work sessions.

---

## 🔑 The Full Architecture Picture

```
┌──────────────────────────────────────────────────────────────────────┐
│  YOUR MACHINE                                                         │
│                                                                        │
│  Working Directory     Staging Area        .git Repository            │
│  ─────────────────     ────────────        ───────────────            │
│                                                                        │
│  [modified files]  →   [staged files]  →   [commits chain]            │
│                    add                commit                           │
│  [modified files]  ←─────────────────────  [checkout/restore]        │
│                                                                        │
│  .git/HEAD → refs/heads/main → commit hash → tree → blobs             │
│                                                                        │
└──────────────────────────────────────────────────────────────────────┘
                                    │ push
                                    ▼ pull/fetch
┌──────────────────────────────────────────────────────────────────────┐
│  REMOTE (GitHub / Azure DevOps)                                       │
│                                                                        │
│  [same commit objects, trees, blobs — complete mirror]                │
│  refs/heads/main → commit hash                                        │
└──────────────────────────────────────────────────────────────────────┘
```

---

## ❓ Interview Questions

**Q: What are the four Git object types?**

> Blob (file contents), Tree (directory structure), Commit (snapshot + metadata including parent pointer), Tag (named reference to a commit). All stored in `.git/objects/` addressed by SHA-1 hash.

**Q: What is a branch in Git at the file system level?**

> A 41-byte text file in `.git/refs/heads/` containing the SHA-1 hash of the commit it points to. That's all. No file copies, no directory. This is why creating branches in Git is instant and free.

**Q: What is HEAD in Git?**

> A special pointer stored in `.git/HEAD` that tells Git where you currently are. Normally it points to a branch name (e.g., `ref: refs/heads/main`). "Detached HEAD" means it points directly to a commit hash instead of a branch.

**Q: Why does Git have a staging area when most VCS don't?**

> The staging area lets you craft precise, focused commits from messy work sessions. You can edit many files but choose exactly which changes go into each commit, keeping the project history clean and meaningful.

**Q: What is the difference between `origin/main` and `main`?**

> `origin/main` is a remote tracking branch — a read-only local snapshot of what `main` looked like on the remote the last time you fetched. `main` is your local branch that you commit to. They can diverge and are synced with `fetch`/`pull`/`push`.
>
