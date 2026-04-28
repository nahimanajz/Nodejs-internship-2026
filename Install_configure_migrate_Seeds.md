# Sequelize Setup Guide — `library-api`

> **Who is this for?**  
> You are a new developer on the `library-api` project.  
> This guide helps you install and configure Sequelize step by step.  
> Read each step carefully. Do not skip any step.

---

## What is Sequelize?

Sequelize is a tool that helps Node.js talk to a database using JavaScript — no SQL needed.

Instead of writing:
```sql
SELECT * FROM books;
```

You write:
```js
await Book.findAll();
```

Sequelize also has a special tool called the **CLI** (Command Line Interface).  
The CLI helps you create tables and add data to your database automatically.

---

## What you will do in this guide

1. Install the packages
2. Add all scripts to `package.json`
3. Configure the Sequelize CLI (`.sequelizerc`)
4. Configure the database connection
5. Run migrations (create the tables)
6. Run seeders (add test data)
7. Create your models

> **One rule in this project:** You never run Sequelize commands directly in the terminal.  
> Every command goes through `npm run <script-name>`.  
> This keeps the project consistent for everyone on the team.

---

## Step 1 — Install the packages

Open your terminal. Go to the project folder:

```bash
cd library-api
```

Install Sequelize, the PostgreSQL driver, and the Sequelize CLI:

```bash
npm install sequelize pg pg-hstore
npm install --save-dev sequelize-cli
```

**What are these packages?**
- `sequelize` — the main Sequelize tool
- `pg` — lets Node.js connect to PostgreSQL
- `pg-hstore` — required by Sequelize for PostgreSQL
- `sequelize-cli` — a command line tool to generate files (only needed during development)

---

## Step 2 — Add all scripts to `package.json`

This is the most important step for this project.  
All Sequelize commands must be run as `npm run` scripts — never typed directly.

Open `package.json` and replace the `"scripts"` section with this:

```json
"scripts": {
  "start": "node index.js",
  "dev": "nodemon index.js",

  "db:migrate": "sequelize db:migrate",
  "db:migrate:undo": "sequelize db:migrate:undo",
  "db:migrate:undo:all": "sequelize db:migrate:undo:all",

  "db:seed": "sequelize db:seed:all",
  "db:seed:undo": "sequelize db:seed:undo:all",

  "migration:create": "sequelize migration:generate --name",
  "seeder:create": "sequelize seed:generate --name",

  "db:setup": "npm run db:migrate && npm run db:seed"
}
```

**What does each script do?**

| Script | What it does |
|---|---|
| `npm run db:migrate` | Creates all tables in the database |
| `npm run db:migrate:undo` | Undoes the last migration |
| `npm run db:migrate:undo:all` | Undoes all migrations (deletes all tables) |
| `npm run db:seed` | Adds test data to the database |
| `npm run db:seed:undo` | Removes all test data |
| `npm run migration:create -- create-users` | Creates a new migration file |
| `npm run seeder:create -- demo-users` | Creates a new seeder file |
| `npm run db:setup` | Runs migrate + seed together (useful first time) |

> **Note the `--` when creating files:**  
> `npm run migration:create -- create-users`  
> The `--` separates npm arguments from the script arguments. Without it, the name will not be passed correctly.

---

## Step 3 — Configure the Sequelize CLI

The Sequelize CLI needs to know where to put its files.  
You must create a file called `.sequelizerc` in the **root of the project** (same level as `package.json`).

Create the file and open it:

```bash
touch .sequelizerc
```

Write this inside `.sequelizerc`:

```js
import { fileURLToPath } from 'url';
import path from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

export default {
  config: path.resolve(__dirname, 'src/config/database.cjs'),
  'models-path': path.resolve(__dirname, 'src/models'),
  'seeders-path': path.resolve(__dirname, 'src/seeders'),
  'migrations-path': path.resolve(__dirname, 'src/migrations'),
};
```

**What does each line mean?**

| Line | What it does |
|---|---|
| `config` | Where the database connection settings are |
| `models-path` | Where model files will be saved |
| `seeders-path` | Where seeder files will be saved |
| `migrations-path` | Where migration files will be saved |

> **Why `.cjs` for the config?**  
> This project uses ES6 (`"type": "module"` in `package.json`).  
> But the Sequelize CLI does not support ES6 config files yet.  
> A `.cjs` file uses the old format that the CLI can read.  
> Only this one config file is `.cjs` — all other files stay as ES6.

---

## Step 4 — Configure the database connection

### 4a — Update your `.env` file

The `.env.example` in this project only has `PORT`. You need to add your database variables.

Copy the example file:

```bash
cp .env.example .env
```

Open `.env` and add these lines:

```env
PORT=4000

DB_HOST=localhost
DB_PORT=5432
DB_NAME=library_db
DB_USER=your_postgres_username
DB_PASSWORD=your_postgres_password
```

Replace `your_postgres_username` and `your_postgres_password` with your real values.

> **Important:** Never push `.env` to GitHub. It is already in `.gitignore`. Good.

---

### 4b — Create the CLI config file

Create the folder and file:

```bash
mkdir -p src/config
touch src/config/database.cjs
```

Open `src/config/database.cjs` and write this:

```js
require('dotenv').config();

module.exports = {
  development: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    port: process.env.DB_PORT || 5432,
    dialect: 'postgres',
    logging: false,
  },
  test: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME + '_test',
    host: process.env.DB_HOST,
    port: process.env.DB_PORT || 5432,
    dialect: 'postgres',
    logging: false,
  },
  production: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    port: process.env.DB_PORT || 5432,
    dialect: 'postgres',
    logging: false,
  },
};
```

> **Why three environments?**  
> - `development` — your local machine while you are coding (CLI uses this by default)  
> - `test` — used when running automated tests  
> - `production` — the real server with real users

---

### 4c — Create the ES6 database connection file

This file is used by your models — not by the CLI.

```bash
touch src/config/database.js
```

Open `src/config/database.js` and write this:

```js
import { Sequelize } from 'sequelize';
import 'dotenv/config';

const sequelize = new Sequelize(
  process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASSWORD,
  {
    host: process.env.DB_HOST,
    port: process.env.DB_PORT || 5432,
    dialect: 'postgres',
    logging: false,
  }
);

export default sequelize;
```

---

### 4d — Create the PostgreSQL database

Run this command to create the database:

```bash
createdb library_db
```

If that does not work, try:

```bash
psql -U your_postgres_username -c "CREATE DATABASE library_db;"
```

---

## Step 5 — Create and run migrations

A **migration** is a file that creates or changes a table in the database.  
Think of it like a history of all changes to your database structure.

### Generate the migration files

Run these commands **one by one**:

```bash
npm run migration:create -- create-users
npm run migration:create -- create-books
npm run migration:create -- create-loans
```

This creates three files in `src/migrations/`. They look like:  
`20250101120000-create-users.js`  
`20250101120001-create-books.js`  
`20250101120002-create-loans.js`

The numbers at the start are timestamps. Sequelize runs migrations **in order by timestamp**.  
This is why you must generate `create-loans` last — loans depend on users and books existing first.

---

### Write the users migration

Open `src/migrations/XXXXXX-create-users.js` and replace everything with:

```js
export async function up(queryInterface, Sequelize) {
  await queryInterface.createTable('users', {
    id: {
      type: Sequelize.INTEGER,
      autoIncrement: true,
      primaryKey: true,
      allowNull: false,
    },
    name: {
      type: Sequelize.STRING,
      allowNull: false,
    },
    email: {
      type: Sequelize.STRING,
      allowNull: false,
      unique: true,
    },
    password: {
      type: Sequelize.STRING,
      allowNull: false,
    },
    role: {
      type: Sequelize.ENUM('admin', 'user'),
      defaultValue: 'user',
    },
    createdAt: {
      type: Sequelize.DATE,
      allowNull: false,
    },
    updatedAt: {
      type: Sequelize.DATE,
      allowNull: false,
    },
  });
}

export async function down(queryInterface) {
  await queryInterface.dropTable('users');
}
```

> **What is `up` and `down`?**  
> - `up` runs when you apply the migration → creates the table  
> - `down` runs when you undo the migration → deletes the table

---

### Write the books migration

Open `src/migrations/XXXXXX-create-books.js` and replace everything with:

```js
export async function up(queryInterface, Sequelize) {
  await queryInterface.createTable('books', {
    id: {
      type: Sequelize.INTEGER,
      autoIncrement: true,
      primaryKey: true,
      allowNull: false,
    },
    title: {
      type: Sequelize.STRING,
      allowNull: false,
    },
    author: {
      type: Sequelize.STRING,
      allowNull: false,
    },
    availableCopies: {
      type: Sequelize.INTEGER,
      defaultValue: 1,
      allowNull: false,
    },
    createdAt: {
      type: Sequelize.DATE,
      allowNull: false,
    },
    updatedAt: {
      type: Sequelize.DATE,
      allowNull: false,
    },
  });
}

export async function down(queryInterface) {
  await queryInterface.dropTable('books');
}
```

---

### Write the loans migration

A **Loan** is created when a user borrows a book. This table links a user to a book.

Open `src/migrations/XXXXXX-create-loans.js` and replace everything with:

```js
export async function up(queryInterface, Sequelize) {
  await queryInterface.createTable('loans', {
    id: {
      type: Sequelize.INTEGER,
      autoIncrement: true,
      primaryKey: true,
      allowNull: false,
    },
    userId: {
      type: Sequelize.INTEGER,
      allowNull: false,
      references: {
        model: 'users', // must match the users table name exactly
        key: 'id',
      },
      onUpdate: 'CASCADE',
      onDelete: 'CASCADE',
    },
    bookId: {
      type: Sequelize.INTEGER,
      allowNull: false,
      references: {
        model: 'books', // must match the books table name exactly
        key: 'id',
      },
      onUpdate: 'CASCADE',
      onDelete: 'CASCADE',
    },
    borrowedAt: {
      type: Sequelize.DATE,
      defaultValue: Sequelize.NOW,
      allowNull: false,
    },
    returnedAt: {
      type: Sequelize.DATE,
      allowNull: true, // null means the book is not yet returned
    },
    createdAt: {
      type: Sequelize.DATE,
      allowNull: false,
    },
    updatedAt: {
      type: Sequelize.DATE,
      allowNull: false,
    },
  });
}

export async function down(queryInterface) {
  await queryInterface.dropTable('loans');
}
```

> **What is `references`?**  
> It means `userId` is a **foreign key**. Its value must match an `id` in the `users` table.  
> If you try to create a loan with a `userId` that does not exist, the database will reject it.  
> `onDelete: 'CASCADE'` means: if a user is deleted, their loans are deleted automatically too.

---

### Run the migrations

```bash
npm run db:migrate
```

You should see:

```
== 20250101120000-create-users: migrating =======
== 20250101120000-create-users: migrated (0.045s)

== 20250101120001-create-books: migrating =======
== 20250101120001-create-books: migrated (0.032s)

== 20250101120002-create-loans: migrating =======
== 20250101120002-create-loans: migrated (0.028s)
```

If you made a mistake in a migration file and need to fix it:

```bash
# Undo only the last migration
npm run db:migrate:undo

# Undo all migrations and start again
npm run db:migrate:undo:all
```

---

## Step 6 — Create and run seeders

A **seeder** adds test data to your database.  
This is useful so you can test the API without creating data manually every time.

### Generate the seeder files

```bash
npm run seeder:create -- demo-users
npm run seeder:create -- demo-books
```

This creates two files in `src/seeders/`.

---

### Write the users seeder

Open `src/seeders/XXXXXX-demo-users.js` and replace everything with:

```js
import bcrypt from 'bcrypt';

export async function up(queryInterface) {
  const hashedPassword = await bcrypt.hash('password123', 10);

  await queryInterface.bulkInsert('users', [
    {
      name: 'Admin User',
      email: 'admin@library.com',
      password: hashedPassword,
      role: 'admin',
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      name: 'Alice Reader',
      email: 'alice@library.com',
      password: hashedPassword,
      role: 'user',
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  ]);
}

export async function down(queryInterface) {
  await queryInterface.bulkDelete('users', null, {});
}
```

> **What is `bulkInsert`?**  
> It inserts many rows at the same time — much faster than one by one.  
> `down` uses `bulkDelete` to remove all rows when you undo the seeder.

---

### Write the books seeder

Open `src/seeders/XXXXXX-demo-books.js` and replace everything with:

```js
export async function up(queryInterface) {
  await queryInterface.bulkInsert('books', [
    {
      title: 'The Alchemist',
      author: 'Paulo Coelho',
      availableCopies: 3,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      title: 'Atomic Habits',
      author: 'James Clear',
      availableCopies: 2,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      title: 'Clean Code',
      author: 'Robert C. Martin',
      availableCopies: 1,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  ]);
}

export async function down(queryInterface) {
  await queryInterface.bulkDelete('books', null, {});
}
```

---

### Run the seeders

```bash
npm run db:seed
```

You should see:

```
== 20250101120003-demo-users: migrating =======
== 20250101120003-demo-users: migrated (0.022s)

== 20250101120004-demo-books: migrating =======
== 20250101120004-demo-books: migrated (0.018s)
```

To remove all seeded data:

```bash
npm run db:seed:undo
```

> **Tip:** You can check the data is in the database by running:
> ```bash
> psql -U your_postgres_username -d library_db -c "SELECT * FROM users;"
> ```

---

### First time setup shortcut

After you finish writing all migration and seeder files, you can run both together with one command:

```bash
npm run db:setup
```

This runs `db:migrate` then `db:seed` automatically.

---

## Step 7 — Create the Models

A **model** is a JavaScript file that represents a table. You use it in your code to read and write data.

> **Migration vs Model — what is the difference?**  
> - **Migration** = creates the table in the database. You run it once.  
> - **Model** = JavaScript object you use every day in your code to query the database.  
>
> They must match. If the migration has a `title` column, the model must also have `title`.

Create the models folder:

```bash
mkdir -p src/models
```

---

### `src/models/User.js`

```js
import { DataTypes } from 'sequelize';
import sequelize from '../config/database.js';

const User = sequelize.define('User', {
  id: {
    type: DataTypes.INTEGER,
    autoIncrement: true,
    primaryKey: true,
  },
  name: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
  },
  password: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  role: {
    type: DataTypes.ENUM('admin', 'user'),
    defaultValue: 'user',
  },
}, {
  tableName: 'users',  // must match the table name in the migration exactly
  timestamps: true,    // Sequelize manages createdAt and updatedAt automatically
});

export default User;
```

---

### `src/models/Book.js`

```js
import { DataTypes } from 'sequelize';
import sequelize from '../config/database.js';

const Book = sequelize.define('Book', {
  id: {
    type: DataTypes.INTEGER,
    autoIncrement: true,
    primaryKey: true,
  },
  title: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  author: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  availableCopies: {
    type: DataTypes.INTEGER,
    defaultValue: 1,
    allowNull: false,
  },
}, {
  tableName: 'books',
  timestamps: true,
});

export default Book;
```

---

### `src/models/Loan.js`

```js
import { DataTypes } from 'sequelize';
import sequelize from '../config/database.js';

const Loan = sequelize.define('Loan', {
  id: {
    type: DataTypes.INTEGER,
    autoIncrement: true,
    primaryKey: true,
  },
  userId: {
    type: DataTypes.INTEGER,
    allowNull: false,
  },
  bookId: {
    type: DataTypes.INTEGER,
    allowNull: false,
  },
  borrowedAt: {
    type: DataTypes.DATE,
    defaultValue: DataTypes.NOW,
    allowNull: false,
  },
  returnedAt: {
    type: DataTypes.DATE,
    allowNull: true, // null means the book is not yet returned
  },
}, {
  tableName: 'loans',
  timestamps: true,
});

export default Loan;
```

---

### `src/models/index.js`

This file imports all models and defines the **relationships** between them.  
**Always import your models from this file** — never directly from each model file.

```js
import sequelize from '../config/database.js';
import User from './User.js';
import Book from './Book.js';
import Loan from './Loan.js';

// One User can have many Loans
User.hasMany(Loan, { foreignKey: 'userId' });
Loan.belongsTo(User, { foreignKey: 'userId' });

// One Book can have many Loans
Book.hasMany(Loan, { foreignKey: 'bookId' });
Loan.belongsTo(Book, { foreignKey: 'bookId' });

export { sequelize, User, Book, Loan };
```

---

## Final folder structure

After all steps, your project should look like this:

```
library-api/
├── src/
│   ├── config/
│   │   ├── database.js            ← ES6, used by models
│   │   └── database.cjs           ← CommonJS, used by Sequelize CLI only
│   ├── migrations/
│   │   ├── XXXX-create-users.js
│   │   ├── XXXX-create-books.js
│   │   └── XXXX-create-loans.js
│   ├── seeders/
│   │   ├── XXXX-demo-users.js
│   │   └── XXXX-demo-books.js
│   └── models/
│       ├── index.js               ← always import from here
│       ├── User.js
│       ├── Book.js
│       └── Loan.js
├── index.js
├── .env                           ← your local secrets (never push to GitHub)
├── .env.example
├── .sequelizerc                   ← tells CLI where to put files
└── package.json
```

---

## All scripts — quick reference

All commands use `npm run`. You never type `sequelize` directly.

| What you want to do | Command |
|---|---|
| Create all tables | `npm run db:migrate` |
| Undo the last migration | `npm run db:migrate:undo` |
| Undo all migrations | `npm run db:migrate:undo:all` |
| Add test data | `npm run db:seed` |
| Remove test data | `npm run db:seed:undo` |
| Create tables + add data | `npm run db:setup` |
| Create a new migration file | `npm run migration:create -- your-migration-name` |
| Create a new seeder file | `npm run seeder:create -- your-seeder-name` |

---

## Common errors and fixes

**`Cannot find module 'dotenv'` in database.cjs**  
Run: `npm install dotenv`

**`dialect postgres not found`**  
Run: `npm install pg pg-hstore`

**`database "library_db" does not exist`**  
Run: `createdb library_db`

**`password authentication failed`**  
Check `DB_USER` and `DB_PASSWORD` in your `.env` file.

**`SequelizeDatabaseError: relation "users" does not exist`**  
You did not run migrations yet. Run: `npm run db:migrate`

**`SyntaxError: Cannot use import statement` in a migration file**  
Make sure your migration files use `export async function up(...)` — not `module.exports`.

---

You are now ready to use Sequelize in this project. The next step is to use the models inside your controllers and routes.
