# Pull Request Guide — POST /api/books

> **What is a pull request?**
> When you finish building a feature, you don't push code directly into the main codebase.
> Instead, you open a Pull Request (PR) — a formal way of saying *"here is my work, please review it before we merge it."*
> Your mentor reads your code, leaves comments, and only merges it when it is ready.
> This is how every professional engineering team works.

---

## Before You Open a PR

Run through this yourself before touching GitHub. Catching mistakes here saves you a review cycle.

- [ ] `POST /api/books` works in Postman — test every case listed in this guide
- [ ] You are on branch `feature/create-book`, **not** on `main`
- [ ] All changes are committed (run `git status` — it should say "nothing to commit")
- [ ] No `console.log` left in your code
- [ ] Your branch is pushed to GitHub

---

## Step 1 — Create your branch

> **Why a branch?**
> `main` is the stable version of the project. You never work directly on it.
> A branch is your own safe copy where you can build and break things freely.
> When your work is reviewed and approved, it gets merged back into `main`.

```bash
git checkout main                        # switch to main
git pull origin main                     # get the latest changes from GitHub
git checkout -b feature/create-book      # create your branch and switch to it
```

You are now on `feature/create-book`. Everything you commit stays here until it is merged.

---

## Step 2 — Commit as you go

> **Why small commits?**
> One big commit at the end (`"done"`) tells your reviewer nothing about how you thought through the problem.
> Small commits show your reasoning step by step — and if something breaks, you can pinpoint exactly where.

Build the endpoint in steps and commit each one:

```bash
git add .
git commit -m "feat: add POST /api/books route and handler"

# once the auto-set logic works:
git commit -m "feat: set availableCopies equal to totalCopies on creation"

# once validation is in:
git commit -m "feat: return 400 when required fields are missing"
```

### Commit message format

```
<type>: <what you did in plain English>
```

| Type | When to use | Example |
|------|-------------|---------|
| `feat:` | You added new functionality | `feat: add POST /api/books route` |
| `fix:` | You fixed a bug | `fix: availableCopies not auto-set` |
| `chore:` | Setup or config, no feature logic | `chore: install uuid package` |

**Bad** → `git commit -m "done"` or `"fix stuff"` or `"asdf"`

**Good** → `git commit -m "feat: return 400 when title is missing"`

Your commit messages are part of the permanent history of the project. Write them as if someone reading them six months from now needs to understand what changed and why.

---

## Step 3 — Push your branch

> **Why push?**
> Your commits only exist on your local machine until you push.
> Pushing uploads your branch to GitHub so your mentor can see and review it.

```bash
git push origin feature/create-book
```

---

## Step 4 — Open the Pull Request on GitHub

1. Go to the repository on GitHub
2. You will see a yellow banner: **"Compare & pull request"** — click it
3. Confirm the settings:
   - **Base branch:** `main` ← this is where your code will be merged into
   - **Compare branch:** `feature/create-book` ← this is your work
4. Fill in the PR description (template below)
5. In the right sidebar, under **Reviewers**, assign your mentor
6. Click **Create pull request**

> **Do NOT click "Merge pull request."**
> Merging is your reviewer's responsibility after they approve.
> Merging your own PR defeats the purpose of a review.

---

## PR Description — copy this and fill it in

> **Why write a description?**
> Your reviewer was not sitting next to you while you built this.
> The description is your chance to explain what you built, how to verify it works, and confirm you checked the important things.
> A good description gets your PR merged faster.

```markdown
## What this PR does
Adds POST /api/books endpoint. A librarian can add a new book to the catalogue.
availableCopies is automatically set equal to totalCopies — the client never sends it.

## How to test it

Start the server: `npm run dev`

### Case 1 — happy path (expect 201)
POST http://localhost:3000/api/books
Body:
{
  "title": "The Pragmatic Programmer",
  "author": "David Thomas",
  "isbn": "9780135957059",
  "genre": "Programming",
  "totalCopies": 2
}
Expected: 201, availableCopies === 2 in the response

### Case 2 — missing required field (expect 400)
Same request but remove "title"
Expected: 400 with { "success": false, "message": "\"title\" is required" }

### Case 3 — client tries to set availableCopies (should be ignored)
Add "availableCopies": 99 to the body
Expected: 201, but availableCopies in the response still equals totalCopies (2), not 99

## Checklist
- [ ] URL is POST /api/books (plural, prefixed with /api, no verb in the path)
- [ ] availableCopies is auto-set — never accepted from the client
- [ ] Returns { "success": true, "data": { ...book } } on 201
- [ ] Returns { "success": false, "message": "..." } on 400
- [ ] Tested all 3 cases in Postman
- [ ] No console.log in the code
```

---

## Step 5 — Responding to review comments

> **Reviews are not personal.**
> Every professional developer gets their code reviewed and receives comments.
> The goal is better code, not criticism of you as a person.
> Senior engineers get review comments too.

When your mentor leaves feedback on a line of your code:

1. Read the comment carefully — understand *why* they are flagging it, not just *what* to change
2. Make the fix on the **same branch** — do not open a new branch
3. Commit with a message that references what you fixed:

```bash
git add .
git commit -m "fix: address review — reject availableCopies from request body"
git push origin feature/create-book
```

4. Reply to the comment on GitHub confirming you addressed it
5. The PR updates automatically — your mentor will see the new commit

---

## What your reviewer will check for this endpoint

Understanding what reviewers look for helps you catch issues yourself before they do.

| What they check | Why it matters |
|-----------------|----------------|
| `availableCopies` is never from the client | This is an explicit rule in the spec — the value is always derived from `totalCopies` |
| A unique `id` is generated | Every book needs an identity. Use `crypto.randomUUID()` or the `uuid` package |
| `createdAt` is set by the server | Timestamps should never come from the client — the client can send anything |
| Status code is `201`, not `200` | `201 Created` specifically means a resource was created. `200 OK` means something else succeeded. They are not interchangeable |
| `400` message is descriptive | `"title is required"` tells the client exactly what to fix. `"bad request"` tells them nothing |

---

## Quick reference — the full flow

```
1.  git checkout main && git pull origin main
2.  git checkout -b feature/create-book
3.  build → git commit -m "feat: add POST /api/books route"
4.  build → git commit -m "feat: auto-set availableCopies on creation"
5.  build → git commit -m "feat: return 400 on missing fields"
6.  git push origin feature/create-book
7.  Open PR on GitHub → fill in description → assign mentor as reviewer
8.  Receive review → fix comments → push → mentor merges
```

---

*When in doubt, ask your mentor before pushing. A question takes 2 minutes. A broken `main` takes much longer to fix.*
