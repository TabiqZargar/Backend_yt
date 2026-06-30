# Project Camp — Backend

A Node.js/Express REST API for project management with JWT authentication, role-based authorization, file uploads, and email verification.

## Tech Stack

- **Runtime:** Node.js (Express 5)
- **Database:** MongoDB + Mongoose
- **Auth:** JWT (access + refresh tokens), bcrypt
- **Email:** Nodemailer + Mailgen (Mailtrap for dev)
- **File Upload:** Multer
- **Validation:** express-validator

## Project Structure

```
src/
├── index.js                 # Entry point — connects to DB, starts server
├── app.js                   # Express app setup (middleware, routes)
├── db/
│   └── index.js             # MongoDB connection via Mongoose
├── models/                  # Mongoose schemas
│   ├── user.models.js
│   ├── project.models.js
│   ├── projectmember.models.js
│   ├── task.models.js
│   ├── subtask.models.js
│   └── note.models.js
├── routes/                  # Route definitions
│   ├── healthcheck.routes.js
│   ├── auth.routes.js
│   └── project.routes.js
├── controllers/             # Business logic
│   ├── healthcheck.controllers.js
│   ├── auth.controllers.js  # Register, login, logout, refresh, verify, reset
│   ├── project.controllers.js
│   └── task.controllers.js  # Stub — not yet wired
├── middlewares/             # Custom middleware
│   ├── auth.middleware.js   # verifyJWT + validateProjectPermission
│   ├── multer.middleware.js # File upload handling
│   └── validator.middleware.js
├── validators/              # express-validator chains
│   └── index.js
└── utils/                   # Shared utilities
    ├── api-error.js         # Custom error class
    ├── api-response.js      # Standard response wrapper
    ├── async-handler.js     # Async error forwarding
    ├── constants.js         # Enums (roles, statuses)
    └── mail.js              # Email service
```

## Folder Breakdown

| Folder | Purpose |
|--------|---------|
| **models/** | Mongoose schemas for 6 collections: User, Project, ProjectMember, Task, Subtask, Note |
| **routes/** | Maps HTTP endpoints to controllers with middleware chains |
| **controllers/** | Request handlers — each controller receives `(req, res)` and returns an `ApiResponse` or throws `ApiError` |
| **middlewares/** | `verifyJWT` (token extraction + verification), `validateProjectPermission` (role check), Multer (file upload), `validate` (validation runner) |
| **validators/** | Reusable `express-validator` validation chains for user input |
| **utils/** | `ApiError`, `ApiResponse`, `asyncHandler` wrapper, constants, and email helper |

## Auth System

- **Register** → hashed password via bcrypt, verification email sent
- **Login** → returns access token + refresh token (cookies + body)
- **Access Token** — short-lived JWT (`_id`, `email`, `username`)
- **Refresh Token** — long-lived JWT (`_id`), stored in DB, rotatable
- **verifyJWT** middleware extracts token from cookies or `Authorization: Bearer` header
- **validateProjectPermission([roles])** checks user's role in project via `ProjectMember` collection
- **Roles:** `admin`, `project_admin`, `member`

## API Endpoints

All routes are prefixed with `/api/v1`.

### Health Check
| Method | Path | Description |
|--------|------|-------------|
| GET | `/healthcheck` | Server status |

### Auth (`/auth`)
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/register` | — | Register user |
| POST | `/login` | — | Login |
| GET | `/verify-email/:token` | — | Verify email |
| POST | `/refresh-token` | — | Refresh access token |
| POST | `/forgot-password` | — | Send reset email |
| POST | `/reset-password/:token` | — | Reset password |
| POST | `/logout` | JWT | Logout |
| POST | `/current-user` | JWT | Get current user |
| POST | `/change-password` | JWT | Change password |
| POST | `/resend-email-verification` | JWT | Resend verification |

### Projects (`/projects`)
All require JWT.

| Method | Path | Role | Description |
|--------|------|------|-------------|
| GET | `/` | any | List user's projects |
| POST | `/` | any | Create project |
| GET | `/:projectId` | any member | Get project details |
| PUT | `/:projectId` | admin | Update project |
| DELETE | `/:projectId` | admin | Delete project |
| GET | `/:projectId/members` | any member | List members |
| POST | `/:projectId/members` | admin | Add member |
| PUT | `/:projectId/members/:userId` | admin | Update member role |
| DELETE | `/:projectId/members/:userId` | admin | Remove member |

## Environment Variables

Create a `.env` file in the root:

```
PORT=3000
MONGO_URI=mongodb://localhost:27017/projmanagement
CORS_ORIGIN=http://localhost:5173

ACCESS_TOKEN_SECRET=your-access-secret
ACCESS_TOKEN_EXPIRY=15m
REFRESH_TOKEN_SECRET=your-refresh-secret
REFRESH_TOKEN_EXPIRY=7d

MAILTRAP_SMTP_HOST=sandbox.smtp.mailtrap.io
MAILTRAP_SMTP_PORT=2525
MAILTRAP_SMTP_USER=your-user
MAILTRAP_SMTP_PASS=your-pass

FORGOT_PASSWORD_REDIRECT_URL=http://localhost:5173/reset-password
SERVER_URL=http://localhost:3000
```

## Getting Started

```bash
npm install
npm run dev     # nodemon src/index.js
npm start       # node src/index.js
```

## Planned Features

Task and Note modules are stubbed but not yet wired into `app.js`. Routes for `/tasks` and `/notes` are defined in `PRD.md`.
