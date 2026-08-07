# Auth Workflow

A complete, production-style authentication system built with Node.js, Express, and MongoDB — featuring email verification, JWT access/refresh token rotation via httpOnly cookies, and a forgot/reset password flow. Includes a React frontend to test the full flow end to end.

This project builds on top of my [e-commerce-api](https://github.com/awaiskamil-dev/e-commerce-api) project, taking its basic JWT auth and extending it into a full, stateful session-management system.

## Features

- **Register & Email Verification** — new accounts are inactive until the user verifies via a tokenized email link
- **Login with Access + Refresh Tokens** — short-lived access token cookie for normal requests, long-lived refresh token backed by a database record for session control
- **Silent Session Refresh** — expired access tokens are transparently refreshed using the stored refresh token, so users stay logged in without re-entering credentials
- **Revocable Sessions** — each login creates a `Token` record in MongoDB; invalidating it (`isValid: false`) immediately kills that session, even if the JWT itself hasn't expired
- **Logout** — clears both cookies and deletes the session's `Token` record
- **Forgot / Reset Password** — emails a time-limited, hashed reset token; raw token never touches the database
- **Role-based Access Control** — user/admin roles, with the first registered account automatically becoming admin
- **Security Hardening** — `helmet`, `express-rate-limit`, `express-mongo-sanitize`, `xss-clean`, signed cookies

## Tech Stack

**Backend:** Node.js, Express, MongoDB/Mongoose, JWT, bcrypt, SendGrid
**Frontend:** React

## How It Works

1. User registers → receives a verification email → verifies their account
2. On login, the server checks for an existing valid session (`Token` doc) for that user — reuses it if found, otherwise creates a new one and issues fresh access/refresh token cookies
3. Every authenticated request checks the access token first; if it's expired, the server falls back to the refresh token, validates it against the stored `Token` record, and silently re-issues new cookies
4. Logging out deletes the `Token` record and expires both cookies immediately
5. Forgot password emails a random token (hashed before storage); reset password verifies the hash and expiry before allowing a password change

## Getting Started

### Backend

```bash
cd server
npm install
```

Create a `.env` file in `server/` with:

```
MONGO_URL=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
JWT_LIFETIME=1d
SENDGRID_API_KEY=your_sendgrid_api_key
NODE_ENV=production
PORT=5000
```

Run the server:

```bash
npm start
```

### Frontend

```bash
cd front-end
npm install
npm start
```

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/v1/auth/register` | Register a new user |
| POST | `/api/v1/auth/verify-email` | Verify email with token |
| POST | `/api/v1/auth/login` | Log in, receive session cookies |
| DELETE | `/api/v1/auth/logout` | Log out, clear session |
| POST | `/api/v1/auth/forgot-password` | Request a password reset email |
| POST | `/api/v1/auth/reset-password` | Reset password with token |

## Notes

This project was built as part of a Node.js/Express backend course, extended on top of a pre-existing e-commerce API to focus specifically on building out a robust, real-world auth system.