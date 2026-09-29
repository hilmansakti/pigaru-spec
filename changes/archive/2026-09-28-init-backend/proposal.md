## Why

The frontend login screen still accepts only hardcoded demo accounts. PIGARU needs a real API in `./backend` so username and password can be checked against stored users and a token returned in the shape the existing client already expects.

## What Changes

- Add a Go HTTP API under `./backend` using Gin, GORM, and CORS, listening on port 8080 with routes under `/api`.
- Persist users in PostgreSQL and seed the two demo accounts the login page already documents (`admin` / `admin123`, `teacher` / `teacher123`).
- Add username/password login that returns a JWT and the user profile the frontend `User` type uses.
- Add authenticated current-user and logout endpoints so the existing `authService` contract can call the API.
- Leave the frontend demo login in place; wiring the UI to this API is out of scope.

## Capabilities

### New Capabilities

- `backend-server`: Runnable Go API process with configuration, PostgreSQL via GORM, CORS for the Vite frontend, and a health check.
- `user-auth`: Username and password login, JWT bearer sessions, current-user lookup, and logout.

### Modified Capabilities

- None. There are no existing specs under `openspec/specs/`.

## Impact

- New code and module files in `./backend` only. No frontend files change in this change.
- New runtime dependency on a PostgreSQL database and environment configuration (`DATABASE_URL`, `JWT_SECRET`, CORS origin).
- Public HTTP surface: `GET /api/health`, `POST /api/auth/login`, `GET /api/auth/me`, `POST /api/auth/logout`.
- Response body for login matches `frontend/src/services/auth.service.ts`: `{ token, user }` with `user` fields `id`, `username`, `email`, `roles`, `hierarchy_id`, `hierarchy_type`, `full_name`.
