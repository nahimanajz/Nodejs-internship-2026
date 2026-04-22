# Switching from arrays to a real database

> Work through this guide from top to bottom. Do not skip a step.
> Every time you see a command, type it yourself — do not copy and paste.
> If something goes wrong, read the error message carefully before asking for help.

---

## What you are doing and why

Right now your API saves data in JavaScript arrays inside the computer's memory.
This means every time you stop the server, all the data disappears.

A real database saves data to disk. When you stop and restart the server, the data is still there.

You will use **PostgreSQL** as your database and **Sequelize** as the tool that lets your JavaScript code talk to it.

---

## Step 1 — Install PostgreSQL on your Mac

PostgreSQL is the database software. You need to install it once on your computer.

The easiest way on a Mac is to download the Postgres.app application.

1. Go to [postgresapp.com](https://postgresapp.com)
2. Click **Download**
3. Open the downloaded file and drag the elephant icon into your Applications folder
4. Open the app from Applications
5. Click **Initialize** — you will see a list of databases appear
6. Click **Start** to start the database server

PostgreSQL is now running on your Mac. You will need to open this app and click Start every time you restart your computer before using the database.

### Add psql to your terminal

`psql` is the command that lets you talk to PostgreSQL from the terminal. You need to tell Git Bash where to find it.

Open Git Bash and run this command:

```bash
echo 'export PATH="/Applications/Postgres.app/Contents/Versions/latest/bin:$PATH"' >> ~/.bash_profile
```

Then run:

```bash
source ~/.bash_profile
```

Now check it worked:

```bash
psql --version
```

You should see something like `psql (PostgreSQL) 16.x`. If you see that, psql is ready.

---

## Step 2 — Create your project database

Your API needs its own database to store data. Run this in Git Bash:

```bash
psql -U postgres
```

You are now inside the PostgreSQL shell. It looks like this:

```
postgres=#
```

Type this to create the database:

```sql
CREATE DATABASE library_dev;
```

You should see `CREATE DATABASE`. Now exit the shell:

```sql
\q
```

You are back in Git Bash. Your database exists but it is empty — no tables yet. You will create the tables in a later step.

---

## Step 3 — Install the new packages

Open Git Bash and go to your project folder:

```bash
cd ~/path/to/library-api
```

Install the packages your project needs to talk to PostgreSQL:

```bash
npm install sequelize pg pg-hstore
```

Install the Sequelize command line tool as a development tool:

```bash
npm install --save-dev sequelize-cli
```

What these packages do:

| Package | What it does |
|---------|-------------|
| `sequelize` | Lets you write JavaScript instead of SQL to work with the database |
| `pg` | The actual connector between Node.js and PostgreSQL |
| `pg-hstore` | A helper that `pg` needs — install it even if you never use it directly |
| `sequelize-cli` | Gives you commands like `db:migrate` in your terminal |

---

## Step 4 — Create the `.sequelizerc` file

This file tells the Sequelize CLI where your project folders are.
Create it in your project root (the same folder as `package.json`).

In Git Bash:

```bash
touch .sequelizerc
```

Open it in your code editor and write this inside it:

```js
import { resolve, dirname } from 'path';
import { fileURLToPath } from 'url';

const __dirname = dirname(fileURLToPath(import.meta.url));

export default {
  config: resolve(__dirname, 'src/config/database.js'),
  'models-path': resolve(__dirname, 'src/models'),
  'migrations-path': resolve(__dirname, 'src/migrations'),
  'seeders-path': resolve(__dirname, 'src/seeders'),
};
```

Save the file.

> **What this does:** Without this file, Sequelize looks for your files in the wrong folders. This file says "look here instead".

---

## Step 5 — Add new environment variables

Open your `.env` file and add these five lines at the bottom:

```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=library_dev
DB_USER=postgres
DB_PASSWORD=
```

Leave `DB_PASSWORD=` empty — Postgres.app does not set a password by default on Mac.

Also open `.env.example` and add the same five lines but with descriptions instead of values:

```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=your_database_name
DB_USER=your_postgres_username
DB_PASSWORD=your_postgres_password
```

> **Why `.env.example`?** This file is committed to GitHub. It shows anyone who clones the project what variables they need. The real `.env` file with your actual values is never committed.

---

## Step 6 — Create the database configuration file

Create a new folder if it does not exist:

```bash
mkdir -p src/config
```

Create the file:

```bash
touch src/config/database.js
```

Open it and write this inside:

```js
import 'dotenv/config';

const development = {
  username: process.env.DB_USER,
  password: process.env.DB_PASSWORD || null,
  database: process.env.DB_NAME,
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  dialect: 'postgres',
  logging: false,
};

const production = {
  use_env_variable: 'DATABASE_URL',
  dialect: 'postgres',
  dialectOptions: {
    ssl: {
      rejectUnauthorized: false,
    },
  },
  logging: false,
};

export default { development, production };
```

> **What `logging: false` does:** By default Sequelize prints every SQL query it runs. That is useful sometimes but very noisy. Turning it off keeps your terminal clean.

---

## Step 7 — Rewrite your Sequelize models

Your model files in `src/models/` are currently just empty stubs with JSDoc comments.
Now you will replace each one with a real Sequelize model definition.

### `src/models/Book.js`

Delete everything that is there and write this:

```js
export default (sequelize, DataTypes) => {
  return sequelize.define('Book', {
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
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
    isbn: {
      type: DataTypes.STRING,
      allowNull: false,
      unique: true,
    },
    genre: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    totalCopies: {
      type: DataTypes.INTEGER,
      allowNull: false,
    },
    availableCopies: {
      type: DataTypes.INTEGER,
      allowNull: false,
    },
  });
};
```

### `src/models/User.js`

```js
export default (sequelize, DataTypes) => {
  return sequelize.define('User', {
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
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
    passwordHash: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    role: {
      type: DataTypes.ENUM('member', 'admin'),
      defaultValue: 'member',
    },
  });
};
```

### `src/models/Loan.js`

```js
export default (sequelize, DataTypes) => {
  return sequelize.define('Loan', {
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
      primaryKey: true,
    },
    borrowedAt: {
      type: DataTypes.DATE,
      defaultValue: DataTypes.NOW,
    },
    dueDate: {
      type: DataTypes.DATE,
      allowNull: false,
    },
    returnedAt: {
      type: DataTypes.DATE,
      allowNull: true,
      defaultValue: null,
    },
  });
};
```

---

## Step 8 — Create `src/models/index.js`

This is the file that loads all your models and connects them to the database.
Create it:

```bash
touch src/models/index.js
```

Write this inside:

```js
import { Sequelize, DataTypes } from 'sequelize';
import config from '../config/database.js';
import defineBook from './Book.js';
import defineUser from './User.js';
import defineLoan from './Loan.js';

const env = process.env.NODE_ENV || 'development';
const dbConfig = config[env];

let sequelize;

if (dbConfig.use_env_variable) {
  sequelize = new Sequelize(process.env[dbConfig.use_env_variable], dbConfig);
} else {
  sequelize = new Sequelize(
    dbConfig.database,
    dbConfig.username,
    dbConfig.password,
    dbConfig
  );
}

const Book = defineBook(sequelize, DataTypes);
const User = defineUser(sequelize, DataTypes);
const Loan = defineLoan(sequelize, DataTypes);

// Associations — these tell Sequelize how the tables are connected
User.hasMany(Loan, { foreignKey: 'userId' });
Loan.belongsTo(User, { foreignKey: 'userId' });

Book.hasMany(Loan, { foreignKey: 'bookId' });
Loan.belongsTo(Book, { foreignKey: 'bookId' });

export { sequelize, Book, User, Loan };
```

> **What associations do:** They tell Sequelize that one User can have many Loans, and one Loan belongs to one Book. This is what makes it possible to ask for a loan and get the book details attached — without writing SQL yourself.

---

## Step 9 — Create the migrations folder and files

Migrations are files that create your database tables. Think of them as instructions you give to the database: "create this table with these columns."

Create the folder:

```bash
mkdir -p src/migrations
```

### Migration 1 — Users table

Create the file:

```bash
touch src/migrations/001-create-users.js
```

Write this inside:

```js
export async function up(queryInterface, Sequelize) {
  await queryInterface.createTable('Users', {
    id: {
      type: Sequelize.UUID,
      defaultValue: Sequelize.UUIDV4,
      primaryKey: true,
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
    passwordHash: {
      type: Sequelize.STRING,
      allowNull: false,
    },
    role: {
      type: Sequelize.ENUM('member', 'admin'),
      defaultValue: 'member',
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
  await queryInterface.dropTable('Users');
}
```

### Migration 2 — Books table

```bash
touch src/migrations/002-create-books.js
```

Write this inside:

```js
export async function up(queryInterface, Sequelize) {
  await queryInterface.createTable('Books', {
    id: {
      type: Sequelize.UUID,
      defaultValue: Sequelize.UUIDV4,
      primaryKey: true,
    },
    title: {
      type: Sequelize.STRING,
      allowNull: false,
    },
    author: {
      type: Sequelize.STRING,
      allowNull: false,
    },
    isbn: {
      type: Sequelize.STRING,
      allowNull: false,
      unique: true,
    },
    genre: {
      type: Sequelize.STRING,
      allowNull: false,
    },
    totalCopies: {
      type: Sequelize.INTEGER,
      allowNull: false,
    },
    availableCopies: {
      type: Sequelize.INTEGER,
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
  await queryInterface.dropTable('Books');
}
```

### Migration 3 — Loans table

```bash
touch src/migrations/003-create-loans.js
```

Write this inside:

```js
export async function up(queryInterface, Sequelize) {
  await queryInterface.createTable('Loans', {
    id: {
      type: Sequelize.UUID,
      defaultValue: Sequelize.UUIDV4,
      primaryKey: true,
    },
    borrowedAt: {
      type: Sequelize.DATE,
      defaultValue: Sequelize.NOW,
    },
    dueDate: {
      type: Sequelize.DATE,
      allowNull: false,
    },
    returnedAt: {
      type: Sequelize.DATE,
      allowNull: true,
      defaultValue: null,
    },
    userId: {
      type: Sequelize.UUID,
      allowNull: false,
      references: {
        model: 'Users',
        key: 'id',
      },
    },
    bookId: {
      type: Sequelize.UUID,
      allowNull: false,
      references: {
        model: 'Books',
        key: 'id',
      },
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
  await queryInterface.dropTable('Loans');
}
```

> **Why Loans comes last:** The Loans table has columns `userId` and `bookId` that point to rows in the Users and Books tables. PostgreSQL will not let you create those columns unless the tables they point to already exist. That is why migrations run in order: 001 first, then 002, then 003.

---

## Step 10 — Run your migrations

This command reads your migration files and creates the tables in the database:

```bash
npx sequelize-cli db:migrate
```

You should see output like:

```
== 001-create-users: migrating =======
== 001-create-users: migrated (0.123s)
== 002-create-books: migrating =======
== 002-create-books: migrated (0.098s)
== 003-create-loans: migrating =======
== 003-create-loans: migrated (0.201s)
```

### Check the tables exist

Open psql to look inside your database:

```bash
psql -U postgres -d library_dev
```

List all tables:

```sql
\dt
```

You should see `Books`, `Users`, `Loans`, and `SequelizeMeta`.

Exit when done:

```sql
\q
```

> **What is `SequelizeMeta`?** It is a table Sequelize creates automatically to keep track of which migrations have already run. This means running `db:migrate` twice is safe — it will not create the tables again.

### If something goes wrong

If you made a mistake in a migration file and need to undo the last one:

```bash
npx sequelize-cli db:migrate:undo
```

Fix your file, then run `db:migrate` again.

---

## Step 11 — Create a seeder file

A seeder adds sample data to your database so your GET endpoints return something straight away.

Create the folder and file:

```bash
mkdir -p src/seeders
touch src/seeders/001-seed-books.js
```

Write this inside:

```js
export async function up(queryInterface) {
  await queryInterface.bulkInsert('Books', [
    {
      id: '11111111-1111-1111-1111-111111111111',
      title: 'Clean Code',
      author: 'Robert C. Martin',
      isbn: '9780132350884',
      genre: 'Programming',
      totalCopies: 3,
      availableCopies: 3,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: '22222222-2222-2222-2222-222222222222',
      title: 'The Pragmatic Programmer',
      author: 'David Thomas',
      isbn: '9780135957059',
      genre: 'Programming',
      totalCopies: 2,
      availableCopies: 2,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
    {
      id: '33333333-3333-3333-3333-333333333333',
      title: 'You Don\'t Know JS',
      author: 'Kyle Simpson',
      isbn: '9781491924464',
      genre: 'JavaScript',
      totalCopies: 4,
      availableCopies: 4,
      createdAt: new Date(),
      updatedAt: new Date(),
    },
  ]);
}

export async function down(queryInterface) {
  await queryInterface.bulkDelete('Books', null, {});
}
```

Run the seeder:

```bash
npx sequelize-cli db:seed:all
```

---

## Step 12 — Rewrite your repositories

Now you replace the array operations in your three repository files with Sequelize method calls. The function names stay exactly the same — only what happens inside them changes.

### `src/repositories/bookRepository.js`

Open the file, delete everything, and write this:

```js
import { Book } from '../models/index.js';

export const findAll = ({ genre, available } = {}) => {
  const where = {};
  if (genre) where.genre = genre;
  if (available === 'true') where.availableCopies = { [Symbol.for('gt')]: 0 };
  return Book.findAll({ where });
};

export const findById = (id) => Book.findByPk(id);

export const create = (data) => Book.create(data);

export const update = async (id, data) => {
  const book = await Book.findByPk(id);
  if (!book) return null;
  return book.update(data);
};

export const remove = async (id) => {
  const book = await Book.findByPk(id);
  if (!book) return false;
  await book.destroy();
  return true;
};
```

> **Notice:** The function names `findAll`, `findById`, `create`, `update`, `remove` are exactly the same as Phase 1. Your service files do not know or care that the implementation changed.

### `src/repositories/userRepository.js`

```js
import { User } from '../models/index.js';

export const findById = (id) => User.findByPk(id);

export const findByEmail = (email) => User.findOne({ where: { email } });

export const create = (data) => User.create(data);
```

### `src/repositories/loanRepository.js`

```js
import { Loan, Book } from '../models/index.js';

export const create = (data) => Loan.create(data);

export const findById = (id) => Loan.findByPk(id);

export const update = async (id, data) => {
  const loan = await Loan.findByPk(id);
  if (!loan) return null;
  return loan.update(data);
};

export const findByUser = (userId) =>
  Loan.findAll({
    where: { userId },
    include: [
      {
        model: Book,
        attributes: ['id', 'title', 'author'],
      },
    ],
  });
```

> **What `include` does:** Instead of fetching loans and then fetching each book separately, Sequelize does it in one database query. Each loan in the result already has a `Book` object attached with `id`, `title`, and `author`. This is what replaces the manual book lookup you wrote in your Phase 1 loan service.

---

## Step 13 — Delete the in-memory store

You no longer need the array store. Delete the folder:

```bash
rm -rf src/store
```

If your `src/app.js` or any other file imports from `src/store`, remove those import lines now.

---

## Step 14 — Update app.js to test the connection

Open `src/app.js` and add these lines near the top, after your other imports:

```js
import { sequelize } from './models/index.js';

sequelize.authenticate()
  .then(() => console.log('Database connected successfully'))
  .catch((err) => console.error('Database connection failed:', err));
```

> **What `authenticate()` does:** It sends a test message to the database to confirm the connection is alive. It does not create or change any tables. You will see the message in your terminal every time the server starts.

---

## Step 15 — Update README.md

Open your `README.md` and add a new section for database setup. Anyone who clones your project needs to know how to get the database running.

Add this after your existing installation steps:

```markdown
## Database setup

1. Install Postgres.app from https://postgresapp.com and start it
2. Create the database:
   psql -U postgres -c "CREATE DATABASE library_dev;"
3. Run migrations to create the tables:
   npx sequelize-cli db:migrate
4. Add sample books:
   npx sequelize-cli db:seed:all

## Environment variables

| Variable | Description |
|----------|-------------|
| PORT | Port the server runs on (default 3000) |
| DB_HOST | Database host (localhost for local development) |
| DB_PORT | Database port (5432 for PostgreSQL) |
| DB_NAME | Database name (library_dev) |
| DB_USER | PostgreSQL username |
| DB_PASSWORD | PostgreSQL password (empty on Mac by default) |
```

---

## Step 16 — Start the server and test

Start your server:

```bash
npm run dev
```

You should see two messages in the terminal:

```
Server running on port 3000
Database connected successfully
```

If you see an error instead, read it carefully. The most common problems are:

| Error message | What it means | How to fix it |
|---------------|--------------|---------------|
| `connection refused` | PostgreSQL is not running | Open Postgres.app and click Start |
| `database "library_dev" does not exist` | You skipped Step 2 | Run `psql -U postgres -c "CREATE DATABASE library_dev;"` |
| `password authentication failed` | Wrong password in `.env` | Leave `DB_PASSWORD=` empty on Mac |
| `Cannot find module` | A file path is wrong | Check your import paths and file names |

### Test in Postman

Open Postman and send a request to `GET {{base_url}}/api/books`. You should see the three books from your seeder.

Then test the other endpoints. Your goal is for all 11 Postman tests to pass exactly as they did in Phase 1 — the only difference is the data now survives a server restart.

To confirm data persists: create a new book with `POST /api/books`, then stop the server with `Ctrl + C` in the terminal, start it again with `npm run dev`, then send `GET /api/books`. The book you created should still be there.

---

## Checklist — what your mentor will check

| Item | Done? |
|------|-------|
| Postgres.app is installed and running | |
| `library_dev` database exists | |
| All three packages installed (`sequelize`, `pg`, `pg-hstore`) | |
| `.sequelizerc` exists in the project root | |
| New DB variables added to `.env` and `.env.example` | |
| `src/config/database.js` created | |
| All three model files rewritten with Sequelize definitions | |
| `src/models/index.js` created with all four associations | |
| Three migration files created in `src/migrations/` | |
| `npx sequelize-cli db:migrate` runs without errors | |
| Tables visible in psql with `\dt` | |
| Seeder file created and `db:seed:all` runs without errors | |
| All three repository files rewritten — no array operations remain | |
| `src/store/` folder deleted | |
| `sequelize.authenticate()` added to `app.js` | |
| README updated with database setup steps | |
| All 11 Postman tests pass | |
| Data survives a server restart | |
