# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Node.js/Express backend for an Expense Tracker application using MongoDB. The backend uses ES6 modules (`"type": "module"` in package.json) and provides RESTful APIs for user authentication and transaction management.

## Development Commands

- **Start development server**: `npm run dev` (uses nodemon for auto-reload)
- **Entry point**: `app.js`

## Architecture

### Core Structure

The application follows a standard MVC pattern:

- **app.js** - Express server entry point, middleware configuration, and route registration
- **DB/Database.js** - MongoDB connection using mongoose
- **models/** - Mongoose schemas for data models
- **controllers/** - Business logic and request handlers
- **Routers/** - Express route definitions

### Data Models

**User (models/UserSchema.js)**:
- Fields: name, email, password (hashed), isAvatarImageSet, avatarImage, transactions (array), createdAt
- User has a one-to-many relationship with transactions stored as array references

**Transaction (models/TransactionModel.js)**:
- Fields: title, amount, category, description, transactionType, date, user (ref), createdAt
- Each transaction belongs to one user via ObjectId reference

### API Routes

All routes are prefixed as follows:
- `/api/v1` - Transaction routes
- `/api/auth` - User authentication routes

**Transaction endpoints** (Routers/Transactions.js):
- POST `/api/v1/addTransaction` - Create new transaction
- POST `/api/v1/getTransaction` - Get filtered transactions (supports type, frequency, date range filters)
- POST `/api/v1/deleteTransaction/:id` - Delete transaction by ID
- PUT `/api/v1/updateTransaction/:id` - Update transaction by ID

**User endpoints** (Routers/userRouter.js):
- POST `/api/auth/register` - User registration with bcrypt password hashing
- POST `/api/auth/login` - User login with password verification
- POST `/api/auth/setAvatar/:id` - Set user avatar image

### Key Implementation Details

**Transaction Filtering** (controllers/transactionController.js):
- `getAllTransactionController` supports dynamic filtering by:
  - Transaction type (income/expense/all)
  - Frequency (last N days or "custom")
  - Custom date range (startDate to endDate)
- Uses moment.js for date manipulation and comparison

**User-Transaction Relationship**:
- When a transaction is created, it's also pushed to the user's transactions array
- When deleting, transaction is removed from both Transaction collection and user's transactions array

**Authentication**:
- Passwords are hashed using bcrypt with salt rounds of 10
- No JWT implementation currently (passwords are deleted from response but no token-based auth)
- User controller includes an unused `allUsers` function at userController.js:124

### Configuration

- Environment variables loaded from `./config/config.env`
- Required env vars: `PORT`, `MONGO_URL`
- CORS configured for specific allowed origins (currently configured for Amplify and Vercel deployments)

### Middleware Stack

In order of execution:
1. express.json()
2. cors (with credentials and specific allowed origins)
3. helmet (with cross-origin resource policy)
4. morgan (dev mode logging)
5. bodyParser (json and urlencoded)

### Import/Export Pattern

All files use ES6 modules:
- Use `import` instead of `require`
- Use `export default` or named exports
- File extensions (.js) must be included in import statements
