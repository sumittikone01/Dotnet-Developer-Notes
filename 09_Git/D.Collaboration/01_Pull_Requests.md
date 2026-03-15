
# 01 — Pull Requests

---

## 🎯 One-Line Definition

> **A Pull Request (PR) is a formal request to merge your branch into another — it creates a space for your team to review the code, discuss changes, run automated checks, and approve before anything touches the main branch.**

---

## 🔑 Why Pull Requests Exist

```
WITHOUT pull requests:
  Developer pushes directly to main
  Nobody reviews the code
  Bugs go straight to production
  No discussion about approach
  No audit trail of decisions

WITH pull requests:
  Developer pushes feature branch
  Opens PR: "I want to merge this into main"
  Team reviews every line of code
  Comments, suggestions, discussions
  Automated tests run
  At least one approval required
  Only then: merge into main

PR = code review + collaboration + quality gate
```

---

## 🔑 The Pull Request Workflow

```
1. Create a branch
   git checkout -b feature/employee-export

2. Do your work, commit
   git add . && git commit -m "Add Excel export to employee grid"
   git push -u origin feature/employee-export

3. Open a Pull Request (on GitHub/Azure DevOps/GitLab)
   - Title: "Add Excel export to employee grid"
   - Description: what you changed and why
   - Target branch: main (where you want to merge)
   - Assign reviewers

4. Team reviews
   - Look at the diff (every line changed)
   - Leave comments on specific lines
   - Suggest changes
   - Request changes (you must fix before merge)
   - Approve (they're happy)

5. Address feedback
   git add . && git commit -m "Address review: use streaming export for large files"
   git push  ← new commits show up in the PR automatically

6. All checks pass + approvals received → merge

7. Delete the feature branch (GitHub offers this automatically)
```

---

## 🔑 Writing a Good PR Description

```
BAD PR description:
  Title: "fixes"
  Description: (empty)

GOOD PR description:
  Title: Add Excel export to employee grid

  ## What changed
  - Added Export to Excel button in Grid toolbar
  - Controller action streams the file (handles large datasets)
  - Uses NPOI library for .xlsx format

  ## Why
  Finance team requested this for monthly reporting.
  Closes #142.

  ## How to test
  1. Go to Employee Management page
  2. Click "Export to Excel" button
  3. File should download as employees.xlsx
  4. Verify all columns present and salary formatted correctly

  ## Screenshots
  [screenshot of export button in toolbar]
```

---

## 🔑 Reviewing a Pull Request

As a reviewer, check:

```
CODE QUALITY:
  □ Does the code do what the title/description says?
  □ Is it readable? Can you understand it without asking?
  □ Are there obvious bugs or edge cases missed?
  □ Any security concerns? (SQL injection, exposed secrets?)
  □ Are error cases handled?

STYLE AND STANDARDS:
  □ Follows the team's coding conventions?
  □ No unnecessary commented-out code?
  □ Meaningful variable/method names?

ARCHITECTURE:
  □ Is this the right approach for the problem?
  □ Does it fit the existing patterns (Controller/BAL/DAL)?
  □ Any duplication that should be extracted?

TESTS:
  □ Are there tests for the new functionality?
  □ Do existing tests still pass?
```

---

## 🔑 PR States

```
DRAFT     → Work in progress — not ready for review
            Good for: "I'm sharing my approach, feedback welcome"
            Reviewers know not to do a formal review yet

OPEN      → Ready for review
            Reviewers can approve/request changes

CHANGES   → Reviewer requested changes
REQUESTED   You must address them before merging

APPROVED  → Reviewers are happy, ready to merge

MERGED    → Successfully integrated into target branch
            (usually auto-deletes the feature branch)

CLOSED    → Closed without merging (abandoned)
```

---

## 🔑 Keeping Your PR Branch Up to Date

If main gets new commits while your PR is open:

```bash
# Method 1: Merge main into your feature branch
git checkout feature/employee-export
git fetch origin
git merge origin/main
# Resolve any conflicts
git push

# Method 2: Rebase onto main (cleaner, linear)
git checkout feature/employee-export
git fetch origin
git rebase origin/main
# Resolve any conflicts during rebase
git push --force-with-lease    # ← use --force-with-lease, not --force
                                # safer: fails if someone else pushed
```

---

## 🔑 Merge Strategies for PRs

```
GitHub / Azure DevOps offer options:

  Merge commit (git merge --no-ff)
  → Creates merge commit
  → Preserves all original commits from branch
  → History shows when feature was merged
  → Good for: seeing feature as a unit

  Squash and merge
  → All commits combined into one
  → Clean main history
  → Feature commits lost (but PR still references them)
  → Good for: keeping main history readable

  Rebase and merge
  → Feature commits replayed on main (linear)
  → No merge commit
  → Looks like you committed directly to main
  → Good for: very clean linear history
```

---

## ❓ Interview Questions

**Q: What is a Pull Request?**

> A request to merge a feature branch into a target branch, creating a formal code review process. It lets the team review every changed line, discuss approaches, run automated checks, and require approvals before the code reaches the main branch.

**Q: What is the difference between a Draft PR and an Open PR?**

> A Draft PR signals work in progress — reviewers know it's not ready for formal review. An Open PR is ready for review and approval. Draft is useful for early feedback or sharing approach without blocking the team.

**Q: Why use `git push --force-with-lease` instead of `git push --force`?**

> `--force` overwrites the remote branch regardless of what's there — dangerous if someone else pushed to the same branch. `--force-with-lease` only force-pushes if the remote matches what you last fetched — it fails safely if someone else's work would be overwritten.
>
