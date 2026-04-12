# Coding standards, folder structure & packages

> Read this document before writing a single file. Every decision about where code lives and how it is written is defined here. Your mentor will use this as the benchmark for every PR review.

---

## Part 1 — ES6 rules

This project is your ES6 training ground. Your mentor will request changes on any PR that breaks these rules. They are not optional.

### Variable declarations

| Situation | Required | Forbidden |
|-----------|----------|-----------|
| Value that never changes | `const name = "Alice"` | `var name = "Alice"` |
| Value that will be reassigned | `let count = 0` | `var count = 0` |
| Any context | — | `var` in any form |

### Functions and syntax

| Rule | Forbidden | Required |
|------|-----------|----------|
| Declaring a function to export | `function getAll() {}` | `const getAll = () => {}` |
| Building a string with a variable | `"Hello " + name` | `` `Hello ${name}` `` |
| Extracting fields from an object | `const title = book.title` | `const { title } = book` |
| Extracting from `req.body` in controllers | `req.body.title, req.body.author` | `const { title, author } = req.body` |

### Async code

| Rule | Forbidden | Required |
|------|-----------|----------|
| Handling promises | `.then().catch()` chains | `async/await` with `try/catch` |
| Controller functions | Synchronous `(req, res) => {}` | `async (req, res, next) => {}` |
| Uncaught async errors | Unhandled rejection | `try/catch` calling `next(err)` |

**Why `async/await` from the start?** Every data operation becomes asynchronous when you connect a real database in Phase 2. Writing `async/await` now means your controllers and services work in Phase 2 without changes. Research: how do you make a function that returns a plain value behave like an async function? You will need this for your Phase 1 repositories.

### ES6 modules

Add `"type": "module"` to `package.json`. This enables ES6 `import`/`export` syntax across the whole project.

| Action | Syntax |
|--------|--------|
| Import a default export | `import express from 'express'` |
| Import named exports | `import { findAll, findById } from '../repositories/bookRepository.js'` |
| Export a default | `export default router` |
| Export named values | `export { findAll, findById }` |

File extensions are required in import paths when using ES modules. Write `'../utils/AppError.js'` not `'../utils/AppError'`.

---

## Part 2 — Folder structure

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
├── postman_collection.json
└── README.md
```

### What each folder is for

| Folder | Single responsibility | Files inside |
|--------|-----------------------|-------------|
| `config/` | App-wide settings only. No logic, no data. | `env.js` or similar |
| `controllers/` | Read `req`, call a service, send `res`. Nothing else. | `bookController.js`, `userController.js`, `loanController.js` |
| `middleware/` | Functions that run on every request before the controller. | `validate.js`, `errorHandler.js` |
| `models/` | Describes the shape of each data type. Phase 1: JSDoc stubs. Phase 2: Sequelize definitions. | `Book.js`, `User.js`, `Loan.js` |
| `repositories/` | The only place that reads or writes data. Phase 1: array operations. Phase 2: ORM calls. | `bookRepository.js`, `userRepository.js`, `loanRepository.js` |
| `routes/` | Maps a URL + HTTP method to a controller function. No logic. | `bookRoutes.js`, `userRoutes.js`, `loanRoutes.js` |
| `services/` | All business rules. "Can this book be borrowed?" belongs here. | `bookService.js`, `userService.js`, `loanService.js` |
| `store/` | Phase 1 only. The in-memory arrays and Maps. Deleted in Phase 2. | `inMemoryStore.js` |
| `utils/` | Small helpers shared across the app. | `AppError.js` |

### Data flow — memorise this

```
Route  →  Controller  →  Service  →  Repository  →  Store
```

Each layer may only call the layer directly below it. No skipping.

| Layer | May import from | Must never import from |
|-------|----------------|----------------------|
| `routes/` | `controllers/` | services, repositories, store |
| `controllers/` | `services/` | repositories, store, `req`/`res` passed down |
| `services/` | `repositories/` | store directly, anything HTTP-related |
| `repositories/` | `store/` | — |

Your mentor will read every `import` statement. An import that skips a layer means the PR is not approved until it is fixed.

---

## Part 3 — Models (data shapes)

In Phase 1, model files are documentation — they describe what your data looks like so that Phase 2 model definitions become a direct translation, not a guessing game.

Each model file must contain a JSDoc comment with every field, its type, and any constraints. Export an empty object as a placeholder.

**Fields to document:**

| Model | Fields |
|-------|--------|
| `Book` | `id`, `title`, `author`, `isbn`, `genre`, `totalCopies`, `availableCopies`, `createdAt` |
| `User` | `id`, `name`, `email`, `passwordHash`, `role` (`'member'` or `'admin'`), `createdAt` |
| `Loan` | `id`, `bookId`, `userId`, `borrowedAt`, `dueDate`, `returnedAt` (nullable) |

Research what a JSDoc `@typedef` comment looks like. That is the format to use.

---

## Part 4 — In-memory store

`src/store/inMemoryStore.js` is your Phase 1 database. It is a plain JavaScript file that exports arrays containing seed data.

Rules:
- It is only ever imported by repository files. Nothing else touches it.
- It must have at least 2–3 seed records for each resource so your GET endpoints return something on first run without needing to POST first.
- When the server restarts, all data resets. That is expected and acceptable in Phase 1.

---

## Part 5 — Validation

All `POST` and `PUT` endpoints must validate the request body before the controller logic runs. Validation belongs in middleware using **Joi**.

The pattern is a middleware factory: a function that takes a Joi schema and returns a middleware function. Research how to write this. Your controller should never see an invalid request body.

Rules for validation responses:
- Invalid body → `400` with a `message` listing every field that failed
- Missing a required field → `400`, never a `500` crash
- Extra fields not in the schema → strip them silently, do not error

---

## Part 6 — Error handling

### AppError class

Create `src/utils/AppError.js`. This is a custom error class that extends the built-in `Error`. It must carry two properties:

- `message` — the human-readable description
- `statusCode` — the HTTP status code to send

Research: how do you extend a built-in class in JavaScript using ES6 `class` syntax?

### Central error handler

Create `src/middleware/errorHandler.js`. This is a special Express middleware that receives errors from `next(err)` calls anywhere in the app. It must be the **last** `app.use()` call in `app.js`.

It handles two cases:

| Error type | What to respond with |
|------------|---------------------|
| An `AppError` (operational) | Use `err.statusCode` and `err.message` |
| Any other error (unexpected) | Respond with `500` and a generic `"Something went wrong"` message — never expose internal details |

Every `async` controller function must wrap its body in `try/catch` and call `next(err)` in the catch block. If you forget, unhandled promise rejections will crash the server.

---

## Part 7 — Packages

Install these packages and read their documentation before using them. The links go to the official documentation.

### Runtime dependencies

| Package | Purpose | Install | Documentation |
|---------|---------|---------|--------------|
| `express` | HTTP server and routing framework | `npm install express` | [expressjs.com/en/4x/api.html](https://expressjs.com/en/4x/api.html) |
| `dotenv` | Loads `.env` file into `process.env` | `npm install dotenv` | [npmjs.com/package/dotenv](https://www.npmjs.com/package/dotenv) |
| `joi` | Request body validation | `npm install joi` | [joi.dev/api](https://joi.dev/api) |
| `bcrypt` | Password hashing and comparison | `npm install bcrypt` | [npmjs.com/package/bcrypt](https://www.npmjs.com/package/bcrypt) |
| `uuid` | Generates unique IDs for in-memory records | `npm install uuid` | [npmjs.com/package/uuid](https://www.npmjs.com/package/uuid) |

### Development dependencies

| Package | Purpose | Install | Documentation |
|---------|---------|---------|--------------|
| `nodemon` | Restarts the server automatically when files change | `npm install --save-dev nodemon` | [nodemon.io](https://nodemon.io) |

### `package.json` scripts to add

```json
"scripts": {
  "start": "node index.js",
  "dev": "nodemon index.js"
}
```

Run the server in development with `npm run dev`. Your server restarts automatically every time you save a file.

---

## Part 8 — Project rubric

Your mentor checks every item in this table before the Phase 1 review begins.

### ES6

| Requirement | Pass | Fail |
|-------------|------|------|
| Variable declarations | `const` and `let` only | Any `var` anywhere |
| Async style | `async/await` with `try/catch` | Any `.then()` or `.catch()` |
| Module syntax | `import` / `export` throughout | Any `require()` or `module.exports` |
| `package.json` | `"type": "module"` present | Missing |
| Destructuring | Used in controllers for `req.body` | Fields accessed individually |
| String interpolation | Template literals only | String concatenation |

### Architecture

| Layer | Allowed to call | Must never call |
|-------|----------------|-----------------|
| `routes/` | `controllers/` only | services, repositories, store |
| `controllers/` | `services/` only | repositories, store |
| `services/` | `repositories/` only | store directly, `req`, `res` |
| `repositories/` | `store/` only | — |

### Files and data

| File / folder | Requirement |
|---------------|-------------|
| `store/inMemoryStore.js` | Exists with seed data for all three resources |
| `models/Book.js` | JSDoc `@typedef` describing all fields; exports empty object |
| `models/User.js` | Same |
| `models/Loan.js` | Same |
| `store/` imported only by | Repository files — nowhere else |

### API, validation and errors

| Area | Requirement |
|------|-------------|
| Endpoints | All 11 implemented and matching the API reference |
| Response shape | `{ success, data }` on success or `{ success, message }` on failure — always |
| POST / PUT validation | All use Joi; invalid bodies return `400` with a readable message |
| Missing required field | Returns `400`, never `500` |
| Resource not found | Returns `404`, never `500` |
| `AppError` class | Exists in `utils/` with `message` and `statusCode` |
| Error handler | Last `app.use()` in `app.js`; all async controllers call `next(err)` |
| Password in response | Never — instant PR rejection |

---

## Part 9 — Questions your mentor will ask during code review

Be ready to answer every one of these out loud for any file in your PR. If you cannot, read the code again and find the answer before requesting a review.

- "Why is this logic in the service and not the controller?"
- "What happens if I send an empty body to this POST endpoint right now?"
- "Why does `availableCopies` change when a loan is created?"
- "If I wanted to swap the array store for a real database, which files would you change?"
- "Why are the repository functions `async` if the store is just an array?"
- "Where is the password being compared during login, and which function does that comparison?"
- "What does `next(err)` do and why do you call it instead of `res.json()`?"
- "Why does the login endpoint return the same error message whether the email or the password is wrong?"
