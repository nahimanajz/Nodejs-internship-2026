# Phase 1 — GitHub, ES6 & library API (in-memory)

> **How to use this guide:** Read a section, close it, then write the code yourself.
> If you copy-paste, you are only practising typing — not programming.
> Your mentor will ask you to explain every line in your PR.

---

## Part 1 — GitHub basics

### What is Git and GitHub?

Git tracks changes to your code over time, like a save history you can rewind.
GitHub hosts your repositories online so your mentor can read and review your code.

The four things you do every day:

| Action | Git command |
|--------|-------------|
| Save a checkpoint | `git commit -m "message"` |
| Upload to GitHub | `git push origin branch-name` |
| Download latest changes | `git pull` |
| Create a feature branch | `git checkout -b feature/what-you-are-building` |

**Your first task:** Research how to configure your name and email globally in Git, then verify it worked. Create a GitHub account if you do not have one.

### The daily workflow

```
create branch → write code → commit often → push → open PR → wait for review → fix feedback → push again
```

You do **not** open a new PR after your mentor leaves feedback. Push new commits to the same branch — the PR updates automatically.

### Commit message convention

Every commit must follow: `type: short description in present tense`

| Type | When to use |
|------|-------------|
| `feat:` | new feature |
| `fix:` | bug fix |
| `refactor:` | restructuring without changing behavior |
| `docs:` | README or comments only |
| `chore:` | config, .gitignore, package.json |

Bad: `git commit -m "stuff"` — your mentor will ask you to rewrite this.
Good: `git commit -m "feat: add POST /api/books endpoint"`

### Branch naming

```
feature/book-routes
feature/user-register
fix/loan-available-copies-check
refactor/move-loan-logic-to-service
```

---

## Part 2 — ES6 rules for this project

This project is your ES6 practice ground. Your mentor will request changes on any PR that uses old-style JavaScript.

### Non-negotiable rules

| Rule | Old way — not allowed | New way — required |
|------|-----------------------|-------------------|
| Declaring variables | `var name = "Alice"` | `const name = "Alice"` |
| Reassignable variable | `var count = 0` | `let count = 0` |
| Exporting a function | `function getAll() {}` | `const getAll = () => {}` |
| String with a variable | `"Hello " + name` | `` `Hello ${name}` `` |
| Extracting from an object | `const title = book.title` | `const { title } = book` |
| Async operations | `.then().catch()` | `async / await` with `try / catch` |

### Why `async/await` from the start?

Every data operation becomes asynchronous when you add a real database in Phase 2. If you write `async/await` now, the controllers and services you build in Phase 1 will still work in Phase 2 without changes.

Research: how do you make a function that returns a plain value behave like an async function? You will need this for your Phase 1 repositories.

### ES6 modules

Use ES6 import/export syntax throughout. To enable it in Node.js, add `"type": "module"` to `package.json`. Then:

- Importing: `import express from 'express'`
- Named import: `import { findAll, findById } from '../repositories/bookRepository.js'`
- Default export: `export default router`
- Named export: `export { findAll, findById }`

Note that with ES modules, file extensions are required in import paths: `'../utils/AppError.js'` not `'../utils/AppError'`.

---

## Part 3 — Folder structure

```
library-api/
├── src/
│   ├── config/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── repositories/
│   ├── routes/
│   ├── services/
│   ├── store/
│   ├── utils/
│   └── app.js
├── .env
├── .env.example
├── .gitignore
├── index.js
└── README.md
```

### What lives in each folder

**`config/`**
Environment and app-wide configuration only. No business logic. No data. Just settings.

**`controllers/`**
One file per resource: `bookController.js`, `userController.js`, `loanController.js`.
A controller reads `req`, calls a service, and sends `res`. That is its entire job.
If you find yourself writing data logic inside a controller, stop — it belongs in the service.

**`middleware/`**
Functions that intercept requests before they reach a controller.
Two things live here in Phase 1: validation and the central error handler.

**`models/`**
Describes the shape of your data. In Phase 1 these are plain JSDoc comments in otherwise empty files.
In Phase 2 they become Sequelize model definitions. Keep the files — their presence makes Phase 2 a small step.

**`repositories/`**
The only layer that reads or writes data. In Phase 1 this means your in-memory arrays.
In Phase 2 you rewrite only these files. Everything above remains unchanged.

**`routes/`**
Maps a URL path and HTTP method to a controller function. Contains no logic whatsoever.

**`services/`**
All business rules live here. "Can this book be borrowed?" is a business rule.
Services call repositories. They never touch `req`, `res`, or the store directly.

**`store/`**
Phase 1 only. Your in-memory arrays and Maps. This folder disappears in Phase 2.

**`utils/`**
Small helpers used across the app. Your custom error class lives here.

### The data flow rule — memorise this

```
Route  →  Controller  →  Service  →  Repository  →  Store
```

Each layer may only call the layer directly below it.
A controller must never import a repository.
A route must never import a service.

If your mentor sees an import that skips a layer, the PR will not be approved.

---

## Part 4 — Project rubric

Complete every item before asking your mentor to review Phase 1.
Your mentor will go through this checklist line by line.

### GitHub & workflow
- [ ] Project is on GitHub in a public repository
- [ ] Every feature was built on its own branch and merged via a PR
- [ ] Commit messages follow the `type: description` convention throughout
- [ ] `.env` is in `.gitignore` and has never been committed
- [ ] `.env.example` is committed and lists every variable name (values left blank)
- [ ] `README.md` explains installation steps and how to run the server

### ES6
- [ ] `"type": "module"` is in `package.json`
- [ ] Zero uses of `var` anywhere in the project
- [ ] Zero `.then()` chains — `async/await` is used everywhere
- [ ] All imports use ES6 `import` syntax
- [ ] All exports use ES6 `export` syntax
- [ ] Destructuring is used in controllers when extracting fields from `req.body`
- [ ] Template literals are used for all string interpolation

### Architecture
- [ ] Folder structure matches the layout above exactly
- [ ] No business logic in route files or controller files
- [ ] No data access (array operations) in controller or service files
- [ ] No `req` or `res` objects passed into services or repositories
- [ ] The in-memory store is only imported by repository files

### In-memory store & models
- [ ] A `store/inMemoryStore.js` file exists with seed data for all three resources
- [ ] Model files exist for Book, User, and Loan with JSDoc comments describing all fields and types
- [ ] Model files export nothing (they are documentation stubs for Phase 2)

### API
- [ ] All 11 endpoints are implemented (see Part 5)
- [ ] All endpoints return JSON
- [ ] All responses follow the `{ success, data }` or `{ success, message }` shape described in Part 5

### Validation
- [ ] All `POST` and `PUT` endpoints validate the request body using Joi
- [ ] Validation errors return `400` with a descriptive `message`
- [ ] Missing required fields return `400`, not a `500` crash

### Error handling
- [ ] A custom `AppError` class exists in `utils/` with `message` and `statusCode`
- [ ] All `async` controller functions wrap their code in `try/catch` and call `next(err)` on failure
- [ ] A central error handler middleware is the last `app.use()` call in `app.js`
- [ ] Requesting a non-existent resource returns `404`, not `500`

### Postman
- [ ] A Postman collection JSON file is committed to the repo root
- [ ] The collection uses a `{{base_url}}` environment variable
- [ ] Every endpoint has at least one happy-path test and one error-case test

---

## Part 5 — Endpoints

### URL naming rules

The HTTP method is the verb. The URL is the noun. Never mix them.

| Rule | Wrong | Correct |
|------|-------|---------|
| Use plural nouns | `/book` | `/books` |
| Lowercase, hyphens only | `/getBooks`, `/Get-Books` | `/books` |
| No verbs in the URL | `/createBook`, `/deleteUser` | Method + `/books` |
| Nested for sub-resources | `/getUserLoans` | `/loans/user/:userId` |
| Partial update via action | `/returnBook` | `PATCH /loans/:id/return` |
| All routes prefixed | `/books` | `/api/books` |

---

### Books

#### `GET /api/books`

Returns all books. Must support optional query params `?genre=Programming` and `?available=true`.

**Success — `200`**
```json
{
  "success": true,
  "data": [
    {
      "id": "a1b2c3",
      "title": "Clean Code",
      "author": "Robert C. Martin",
      "isbn": "9780132350884",
      "genre": "Programming",
      "totalCopies": 3,
      "availableCopies": 2,
      "createdAt": "2024-01-15T08:00:00.000Z"
    }
  ]
}
```

---

#### `GET /api/books/:id`

Returns one book by its ID.

**Success — `200`**
```json
{
  "success": true,
  "data": {
    "id": "a1b2c3",
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "isbn": "9780132350884",
    "genre": "Programming",
    "totalCopies": 3,
    "availableCopies": 2,
    "createdAt": "2024-01-15T08:00:00.000Z"
  }
}
```

**Not found — `404`**
```json
{
  "success": false,
  "message": "Book not found"
}
```

---

#### `POST /api/books`

Creates a new book. `availableCopies` must be set to the same value as `totalCopies` automatically — the client never sends it.

**Request body**
```json
{
  "title": "The Pragmatic Programmer",
  "author": "David Thomas",
  "isbn": "9780135957059",
  "genre": "Programming",
  "totalCopies": 2
}
```

**Success — `201`**
```json
{
  "success": true,
  "data": {
    "id": "d4e5f6",
    "title": "The Pragmatic Programmer",
    "author": "David Thomas",
    "isbn": "9780135957059",
    "genre": "Programming",
    "totalCopies": 2,
    "availableCopies": 2,
    "createdAt": "2024-01-16T09:30:00.000Z"
  }
}
```

**Validation error — `400`**
```json
{
  "success": false,
  "message": "\"title\" is required. \"totalCopies\" must be a number"
}
```

---

#### `PUT /api/books/:id`

Updates an existing book. At least one field is required. The client cannot change `availableCopies` directly.

**Request body** (all fields optional, at least one required)
```json
{
  "totalCopies": 5,
  "genre": "Software Engineering"
}
```

**Success — `200`**
```json
{
  "success": true,
  "data": {
    "id": "a1b2c3",
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "isbn": "9780132350884",
    "genre": "Software Engineering",
    "totalCopies": 5,
    "availableCopies": 2,
    "createdAt": "2024-01-15T08:00:00.000Z"
  }
}
```

**Not found — `404`**
```json
{
  "success": false,
  "message": "Book not found"
}
```

---

#### `DELETE /api/books/:id`

Deletes a book permanently.

**Success — `204`**
*(no body)*

**Not found — `404`**
```json
{
  "success": false,
  "message": "Book not found"
}
```

---

### Users

#### `POST /api/users/register`

Registers a new user. Research `bcrypt` — you must hash the password before storing it. Never store a plain-text password.

**Request body**
```json
{
  "name": "Alice Uwimana",
  "email": "alice@example.com",
  "password": "securepassword123"
}
```

**Success — `201`**
```json
{
  "success": true,
  "data": {
    "id": "u7h8i9",
    "name": "Alice Uwimana",
    "email": "alice@example.com",
    "role": "member",
    "createdAt": "2024-01-16T10:00:00.000Z"
  }
}
```

> `passwordHash` is never returned in any response — not here, not anywhere. If your mentor sees it in a response, the PR will not pass.

**Email already taken — `409`**
```json
{
  "success": false,
  "message": "An account with this email already exists"
}
```

**Validation error — `400`**
```json
{
  "success": false,
  "message": "\"email\" must be a valid email address"
}
```

---

#### `POST /api/users/login`

Validates credentials. Research how `bcrypt.compare()` works.

**Request body**
```json
{
  "email": "alice@example.com",
  "password": "securepassword123"
}
```

**Success — `200`**
```json
{
  "success": true,
  "data": {
    "id": "u7h8i9",
    "name": "Alice Uwimana",
    "email": "alice@example.com",
    "role": "member"
  }
}
```

**Wrong credentials — `401`**
```json
{
  "success": false,
  "message": "Invalid email or password"
}
```

> Use the same message whether the email does not exist or the password is wrong. Never tell the client which one failed — that is a security leak.

---

#### `GET /api/users/:id`

Returns a user profile. Password hash never included.

**Success — `200`**
```json
{
  "success": true,
  "data": {
    "id": "u7h8i9",
    "name": "Alice Uwimana",
    "email": "alice@example.com",
    "role": "member",
    "createdAt": "2024-01-16T10:00:00.000Z"
  }
}
```

**Not found — `404`**
```json
{
  "success": false,
  "message": "User not found"
}
```

---

### Loans

#### `POST /api/loans`

Borrows a book. Before creating the loan, the service must:
1. Confirm the book exists
2. Confirm the user exists
3. Confirm `availableCopies > 0`
4. Decrement `availableCopies` by 1

**Request body**
```json
{
  "bookId": "a1b2c3",
  "userId": "u7h8i9",
  "dueDate": "2024-02-16"
}
```

**Success — `201`**
```json
{
  "success": true,
  "data": {
    "id": "l1m2n3",
    "bookId": "a1b2c3",
    "userId": "u7h8i9",
    "borrowedAt": "2024-01-16T11:00:00.000Z",
    "dueDate": "2024-02-16T00:00:00.000Z",
    "returnedAt": null
  }
}
```

**No copies available — `409`**
```json
{
  "success": false,
  "message": "No copies of this book are currently available"
}
```

**Book not found — `404`**
```json
{
  "success": false,
  "message": "Book not found"
}
```

---

#### `PATCH /api/loans/:id/return`

Returns a borrowed book. The service must:
1. Confirm the loan exists
2. Confirm `returnedAt` is still `null` — if not, reject
3. Set `returnedAt` to the current timestamp
4. Increment `availableCopies` on the book by 1

> Why `PATCH` and not `PUT`? Because you are updating one field on the resource, not replacing the whole thing. Understand the difference before your review.

**No request body required.**

**Success — `200`**
```json
{
  "success": true,
  "data": {
    "id": "l1m2n3",
    "bookId": "a1b2c3",
    "userId": "u7h8i9",
    "borrowedAt": "2024-01-16T11:00:00.000Z",
    "dueDate": "2024-02-16T00:00:00.000Z",
    "returnedAt": "2024-01-20T14:30:00.000Z"
  }
}
```

**Already returned — `409`**
```json
{
  "success": false,
  "message": "This loan has already been returned"
}
```

**Not found — `404`**
```json
{
  "success": false,
  "message": "Loan not found"
}
```

---

#### `GET /api/loans/user/:userId`

Returns the full borrowing history for a user, with book details nested inside each loan.

In Phase 1 you must look up the book manually in the service and attach it. In Phase 2 this happens automatically with Sequelize `include`. Notice the connection.

**Success — `200`**
```json
{
  "success": true,
  "data": [
    {
      "id": "l1m2n3",
      "borrowedAt": "2024-01-16T11:00:00.000Z",
      "dueDate": "2024-02-16T00:00:00.000Z",
      "returnedAt": "2024-01-20T14:30:00.000Z",
      "book": {
        "id": "a1b2c3",
        "title": "Clean Code",
        "author": "Robert C. Martin"
      }
    }
  ]
}
```

**User has no loans — `200`** (empty array, not `404`)
```json
{
  "success": true,
  "data": []
}
```

**User not found — `404`**
```json
{
  "success": false,
  "message": "User not found"
}
```

---

## Part 6 — Questions your mentor will ask during code review

Be ready to answer every one of these verbally, for any file in your PR:

- "Why is this logic in the service and not the controller?"
- "What happens if I send an empty body to this endpoint right now?"
- "Why does `availableCopies` change when a loan is created?"
- "If I wanted to swap the array store for a database, which files would you change?"
- "Why are the repository functions `async` if the store is just an array?"
- "Where is the password being compared in the login flow, and which function does it?"
- "What does `next(err)` do and why do you call it instead of `res.json()`?"
- "Why does the login endpoint return the same error message whether the email or the password is wrong?"

If you cannot answer these, you are not ready to move to Phase 2.
