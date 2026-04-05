# 📚 Library Management System API

A RESTful API for managing a library — built with **Node.js**, **Express**, and **Sequelize ORM** on top of a relational database. Supports user authentication, book management, and a full borrow/return flow with role-based access control.

---

## 🚀 Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js |
| Framework | Express.js |
| ORM | Sequelize |
| Database | MySQL (configurable dialect) |
| Auth | Manual (email + password) |

---

## 📁 Project Structure

```
src/
├── database/
│   ├── models/
│   │   ├── book.model.js          # Book schema (title, author, isbn, copies)
│   │   ├── borrowedBook.model.js  # BorrowedBook schema (status, dates, FK refs)
│   │   ├── relation.js            # Sequelize associations
│   │   └── user.model.js          # User schema (name, email, password, role)
│   └── connection.js              # Sequelize instance, connect & sync
├── modules/
│   ├── auth/
│   │   ├── auth.controller.js     # POST /signup, POST /login
│   │   └── auth.service.js        # Business logic for auth
│   ├── books/
│   │   ├── books.controller.js    # POST /add-book, GET /get-all-books, GET /get-book-by-id/:id
│   │   └── books.service.js       # Business logic for books
│   └── borrowedBook/
│       ├── borrowed.controller.js # Borrow, return, and view endpoints
│       └── borrowed.service.js    # Business logic for borrowing
├── app.controller.js              # Express app setup + route registration
└── main.js                        # Entry point
```

---

## ⚙️ Environment Variables

Create a `config/env.service.js` file (or `.env` with a loader) and export the following:

```js
export const databaseName     = 'your_db_name';
export const databaseUser     = 'your_db_user';
export const databasePassword = 'your_db_password';
export const databaseHost     = 'localhost';
export const databaseDialect  = 'mysql'; // or 'postgres', 'sqlite', etc.
```

---

## 🛠️ Installation & Setup

```bash
# 1. Clone the repo
git clone https://github.com/your-username/library-management-system.git
cd library-management-system

# 2. Install dependencies
npm install

# 3. Configure your environment variables
# Edit config/env.service.js with your DB credentials

# 4. Start the server
node src/main.js
```

Server runs on **http://localhost:3000**

---

## 📌 API Endpoints

### 🔐 Auth — `/api/auth`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/signup` | Register a new user |
| POST | `/login` | Login and retrieve user data |

**POST `/signup` — Request Body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secret123",
  "role": "user"
}
```

**POST `/login` — Request Body:**
```json
{
  "email": "john@example.com",
  "password": "secret123"
}
```

---

### 📖 Books — `/api/books`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/add-book` | Add a new book |
| GET | `/get-all-books` | Fetch all books |
| GET | `/get-book-by-id/:id` | Fetch a single book by ID |

**POST `/add-book` — Request Body:**
```json
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "978-0132350884"
}
```

---

### 📦 Borrowing — `/api/borrow`

| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/borrow-book` | Borrow a book | User |
| GET | `/user/:userID` | Get borrowed books for a user | User |
| GET | `/all/:userID` | Get ALL borrowed records (admin only) | Admin |
| POST | `/return/:id` | Return a borrowed book | User |

**POST `/borrow-book` — Request Body:**
```json
{
  "userID": 1,
  "bookID": 3
}
```

---

## 🗄️ Database Models

### User
| Field | Type | Notes |
|---|---|---|
| userID | INTEGER | PK, Auto Increment |
| name | STRING | Required |
| email | STRING | Unique, validated |
| password | STRING | Required |
| role | ENUM | `user` \| `admin`, default: `user` |

### Book
| Field | Type | Notes |
|---|---|---|
| bookID | INTEGER | PK, Auto Increment |
| title | STRING | Required |
| author | STRING | Required |
| isbn | STRING | Unique |
| availableCopies | INTEGER | Default: 1 |
| totalCopies | INTEGER | Default: 1 |

### BorrowedBook
| Field | Type | Notes |
|---|---|---|
| borrowID | INTEGER | PK, Auto Increment |
| userID | INTEGER | FK → User |
| bookID | INTEGER | FK → Book |
| borrowDate | DATE | Default: NOW |
| returnDate | DATE | Nullable |
| status | ENUM | `borrowed` \| `returned`, default: `borrowed` |

---

## 🔗 Model Associations

```
User     ──< BorrowedBook >── Book
(hasMany)                  (belongsTo)
```

- A **User** can borrow many books
- A **Book** can be borrowed many times
- **BorrowedBook** is the join table with extra fields (status, dates)
- Cascade on update and delete

---

## 🎯 Core Features

- ✅ User registration & login
- ✅ Role-based access (`user` / `admin`)
- ✅ Add books with ISBN uniqueness check
- ✅ Borrow a book with available copies tracking (auto-decrement)
- ✅ Return a book (auto-increment available copies + set returnDate)
- ✅ Admin-only endpoint to view all borrowed records across all users
- ✅ Global error handler middleware
- ✅ Sequelize auto-sync on startup

---

## 📋 Notes

- Passwords are stored as plain text — consider adding **bcrypt** hashing before production.
- No JWT is implemented yet — consider adding token-based auth for protected routes.
- `availableCopies` is automatically decremented on borrow and incremented on return.
- Admin access is checked manually via `role` field — no middleware guard yet.
