# GitHub guide

> **Your first task before writing any code:** Complete every section in Part 1 and have a working GitHub repository with a README before you touch Express or Node.

---

## Part 1 — One-time setup

### Create a GitHub account

Go to [github.com](https://github.com) and create a free account if you do not have one. Use a professional username — this portfolio will be visible to future employers.

### Install Git

Download Git from [git-scm.com](https://git-scm.com) and install it. After installation, open your terminal and research how to:

- Set your global username
- Set your global email
- Verify both settings were saved

These two settings appear on every commit you ever make. Get them right once and you never touch them again.

### Create your repository

1. Go to [github.com/new](https://github.com/new)
2. Name it `library-api`
3. Set visibility to **Public** — your mentor needs to access it without an invitation
4. Check **Add a README file**
5. Click **Create repository**

Then clone it to your machine:

```bash
git clone https://github.com/your-username/library-api.git
cd library-api
```

---

## Part 2 — The README.md

Create or update `README.md` before writing any API code. This is the front door of your project. Anyone who opens the repository — a mentor, a recruiter, a future colleague — should understand what it does and how to run it within 60 seconds.

Your README must answer these four questions:

| Question | What to write |
|----------|--------------|
| What is this project? | One or two sentences: what the API does and what problem it solves |
| What features does it have? | A short list — manage books, register users, borrow and return books |
| How do I run it? | Steps: clone → `npm install` → copy `.env.example` to `.env` → `npm run dev` |
| How do I test the endpoints? | Where to find the Postman collection and how to import it |

A README with only a title and no other content will be sent back for revision before any code review begins.

---

## Part 3 — Daily workflow

Every feature you build follows this same cycle without exception:

```
create a branch → write code → commit small checkpoints → push → open PR → wait for review → fix feedback → push again
```

### Branching

Never commit directly to `main`. Create a branch for every feature or fix:

```bash
git checkout -b feature/book-routes
```

Name your branches so anyone can understand what they contain at a glance.

| Type | Example |
|------|---------|
| New feature | `feature/book-routes` |
| New feature | `feature/user-register` |
| Bug fix | `fix/loan-available-copies-check` |
| Refactor | `refactor/move-loan-logic-to-service` |

### Committing

Commit often — after each small working piece, not at the end of the day. Small commits are easier to review and easier to undo if something breaks.

Every commit message must follow this pattern:

```
type: short description in present tense
```

| Type | When to use |
|------|-------------|
| `feat:` | adding a new feature |
| `fix:` | fixing a bug |
| `refactor:` | restructuring code without changing behavior |
| `docs:` | updating README or comments only |
| `chore:` | config files, `.gitignore`, `package.json` changes |

**Bad commits your mentor will ask you to rewrite:**
```
git commit -m "stuff"
git commit -m "fix"
git commit -m "wip"
git commit -m "done"
```

**Good commits:**
```
git commit -m "feat: add GET /api/books endpoint"
git commit -m "feat: add Joi validation to POST /api/books"
git commit -m "fix: return 404 when book id does not exist"
git commit -m "refactor: move borrow logic from controller to service"
```

### Pushing

After committing, upload your branch to GitHub:

```bash
git push origin feature/book-routes
```

### Opening a pull request

1. Go to your repository on GitHub
2. Click the **Compare & pull request** button that appears after pushing
3. Write a short description of what the PR does
4. Submit it — your mentor will receive a notification

### After your mentor leaves feedback

Do **not** close the PR and open a new one. Fix the issues in your code, commit the changes, and push again to the same branch:

```bash
git add .
git commit -m "refactor: move validation out of controller per review"
git push origin feature/book-routes
```

The PR updates automatically. Your mentor sees the new commits appear inside the same review thread.

---

## Part 4 — Files that must always be in `.gitignore`

Create a `.gitignore` file in the project root before your first commit. It must contain at minimum:

```
node_modules/
.env
```

`node_modules/` is never committed because it is generated locally by `npm install`. Committing it bloats the repository and causes conflicts.

`.env` is never committed because it contains secrets — database passwords, API keys, tokens. If it is ever committed and pushed to a public repository, treat every value inside it as compromised and rotate them immediately.

### `.env.example`

Create a `.env.example` file and commit it. This file shows every variable name the project needs, but with the actual values left blank or as placeholders:

```
PORT=
#leave this part for phase two, where we will use actual database like Postgress or Mysql
DB_HOST=
DB_NAME=
DB_USER=
DB_PASSWORD=
```

Anyone who clones the repository copies this file to `.env` and fills in their own values. This is the standard pattern for every professional Node.js project.

---

## Part 5 — Postman

Postman is the tool you use to send HTTP requests to your API while it is running locally. Download it from [postman.com](https://www.postman.com).

### Setting up an environment

Instead of hardcoding `http://localhost:3000` into every request, create a Postman environment:

1. Click **Environments** in the left sidebar
2. Create a new environment called `Library API - Local`
3. Add a variable: name `base_url`, initial value `http://localhost:3000`
4. Save it and select it as the active environment

Now every request URL uses `{{base_url}}/api/books` instead of `http://localhost:3000/api/books`. When you deploy in Phase 2 you just update one variable.

### What to test for every endpoint

For every endpoint you build, create at least two requests in Postman:

| Test type | What it checks |
|-----------|---------------|
| Happy path | Valid input, expect the correct success code and response shape |
| Error case | Invalid or missing data, expect the correct error code and message |

Some endpoints need more than two. The borrow endpoint, for example, needs a test for "book not found", "user not found", and "no copies available" separately.

### Exporting the collection

When Phase 1 is complete, export your Postman collection:

1. Right-click your collection in the sidebar
2. Click **Export**
3. Choose Collection v2.1
4. Save the file as `postman_collection.json`
5. Place it in the project root and commit it

Your mentor will import this file and run your tests.

---

## Part 6 — GitHub checklist for Phase 1 review

Your mentor will check every item in this table before reviewing any code.

| Requirement | What your mentor checks |
|-------------|------------------------|
| Public repository | Can open the URL without logging in |
| README present | Answers the four questions from Part 2 |
| Branch per feature | Every PR came from a named branch, never from `main` |
| Commit messages | All follow `type: description` — no vague messages |
| `.env` never committed | Not present anywhere in the git history |
| `.env.example` committed | Lists all variable names with blank values |
| `postman_collection.json` committed | In the project root, importable |
| Postman environment variable | All request URLs use `{{base_url}}` |
