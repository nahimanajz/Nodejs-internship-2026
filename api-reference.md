# Library API — endpoint reference

> **How to use this document:** This is your contract. Every endpoint you build must match the URL, method, request body, and response shapes shown here exactly. Test each one in Postman before marking it done.

---

## What this project is

The Library Management API is a backend service that allows a library to manage its book collection and track borrowing activity. It supports three resources: **books**, **users**, and **loans**.

A librarian can add, update, and remove books from the catalogue. A member can register an account, log in, borrow an available book, and return it when done. Every borrow and return is recorded as a loan.

### Resources at a glance

| Resource | What it represents |
|----------|--------------------|
| `Book` | A title in the library catalogue, with a count of total and available copies |
| `User` | A registered member or admin who can borrow books |
| `Loan` | A record of one user borrowing one book, including borrow date, due date, and return date |

---

## URL naming rules

The HTTP method is the verb. The URL is the noun. Never mix them.

| Rule | Wrong | Correct |
|------|-------|---------|
| Use plural nouns | `/book` | `/books` |
| Lowercase only | `/getBooks`, `/Get-Books` | `/books` |
| No verbs in the URL | `/createBook`, `/deleteUser` | method + `/books` |
| Nested for sub-resources | `/getUserLoans` | `/loans/user/:userId` |
| Partial update via action | `/returnBook` | `PATCH /loans/:id/return` |
| All routes prefixed with `/api` | `/books` | `/api/books` |

---

## Response shape

Every response from this API follows one of two shapes. Never return a raw string or a bare object.

**Success**
```json
{ "success": true, "data": {} }
```

**Failure**
```json
{ "success": false, "message": "Human-readable explanation" }
```

---

## Books

### `GET /api/books`

Returns all books in the catalogue. Supports two optional query parameters:

- `?genre=Programming` — filter by genre
- `?available=true` — return only books with `availableCopies > 0`

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

### `GET /api/books/:id`

Returns a single book by its ID.

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

### `POST /api/books`

Adds a new book to the catalogue. The client sends `totalCopies` — your service must automatically set `availableCopies` to the same value. The client never sends `availableCopies`.

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

### `PUT /api/books/:id`

Updates an existing book. All fields are optional but at least one must be provided. The client cannot change `availableCopies` directly — it is managed by the loan logic.

**Request body** (at least one field required)
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

**Validation error — `400`**
```json
{
  "success": false,
  "message": "Request body must contain at least one field to update"
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

### `DELETE /api/books/:id`

Permanently removes a book from the catalogue.

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

## Users

### `POST /api/users/register`

Creates a new user account. Before storing the user, your service must hash the password using `bcrypt`. Never store a plain-text password.

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

> `passwordHash` is never returned in any response — not here, not anywhere. If your mentor sees it in a response the PR will not pass.

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

### `POST /api/users/login`

Checks credentials against the stored hash. Research `bcrypt.compare()` to understand how this works without ever decrypting the stored hash.

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

> Return the same message whether the email does not exist or the password is wrong. Never tell the client which one failed — that is a security leak.

---

### `GET /api/users/:id`

Returns a user profile. The password hash is never included in any response.

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

## Loans

### `POST /api/loans`

Records a user borrowing a book. Before creating the loan your service must perform these checks in order:

1. Confirm the book exists — if not, return `404`
2. Confirm the user exists — if not, return `404`
3. Confirm `availableCopies > 0` — if not, return `409`
4. Decrement `availableCopies` by 1
5. Create and return the loan

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

**Book or user not found — `404`**
```json
{
  "success": false,
  "message": "Book not found"
}
```

---

### `PATCH /api/loans/:id/return`

Records that a borrowed book has been returned. Your service must:

1. Confirm the loan exists — if not, return `404`
2. Check that `returnedAt` is still `null` — if it already has a value, the book was already returned, return `409`
3. Set `returnedAt` to the current timestamp
4. Increment `availableCopies` on the associated book by 1

> **Why `PATCH` and not `PUT`?** `PUT` replaces an entire resource. `PATCH` applies a partial change. Returning a book updates one field on the loan — that is a `PATCH`. Understand this distinction before your review.

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

### `GET /api/loans/user/:userId`

Returns the full borrowing history for a user. Each loan entry must include a nested `book` object with the book details — not just a `bookId`.

In Phase 1 you look up the book manually in your service and attach it to each loan. In Phase 2 this happens in a single database query using Sequelize `include`. Notice that pattern now.

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

## All endpoints at a glance

| Method | URL | What it does | Success code |
|--------|-----|--------------|-------------|
| GET | `/api/books` | List all books (filterable) | 200 |
| GET | `/api/books/:id` | Get one book | 200 |
| POST | `/api/books` | Create a book | 201 |
| PUT | `/api/books/:id` | Update a book | 200 |
| DELETE | `/api/books/:id` | Delete a book | 204 |
| POST | `/api/users/register` | Register a user | 201 |
| POST | `/api/users/login` | Log in | 200 |
| GET | `/api/users/:id` | Get a user profile | 200 |
| POST | `/api/loans` | Borrow a book | 201 |
| PATCH | `/api/loans/:id/return` | Return a book | 200 |
| GET | `/api/loans/user/:userId` | Get a user's loan history | 200 |
