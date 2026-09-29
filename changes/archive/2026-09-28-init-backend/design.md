## Context

PIGARU V2 has a Vite frontend whose login page still checks `admin` / `admin123` and `teacher` / `teacher123` in the browser. `frontend/src/services/auth.service.ts` already calls `POST /auth/login`, `GET /auth/me`, and `POST /auth/logout` through an Axios client whose base URL defaults to `http://localhost:8080/api`. The `User` shape is `id`, `username`, `email`, `roles`, `hierarchy_id`, `hierarchy_type`, and `full_name`. There is no `backend/` directory yet.

## Goals / Non-Goals

**Goals:**

- Stand up a Go API in `./backend` with Gin, GORM, and CORS.
- Authenticate by username and password and return a JWT plus the user profile the frontend already types.
- Expose current-user and logout so the existing auth client methods have a server.
- Seed the two documented demo accounts on an empty database.

**Non-Goals:**

- Replacing the demo login in `LoginPage.tsx` or changing any frontend file.
- Registration, password reset, token refresh, or a server-side token denylist.
- Roles beyond storing and returning the role names the frontend already uses.
- Hierarchy management. `hierarchy_id` and `hierarchy_type` are stored fields only.

## Decisions

### Module layout under `./backend`

A single Go module, `pigaru-backend`, with `cmd/api` as the process entry and `internal/` for config, database, models, handlers, middleware, and routing.

Alternative: one `main.go` at the module root. Rejected because auth, persistence, and HTTP will grow past a single file.

### Gin on port 8080, routes under `/api`

The process listens on `PORT` (default `8080`). Every route is mounted on the `/api` group so paths match the frontend base URL without a rewrite.

Alternative: serve at the root and change `VITE_API_URL`. Rejected because this change does not touch the frontend.

### PostgreSQL through GORM

Persistence uses `gorm.io/driver/postgres` and `DATABASE_URL`. Startup runs `AutoMigrate` for the user and role tables, then seeds demo users only when the users table is empty.

Alternative: SQLite for zero infrastructure. Rejected because later domain data (classes, attendance, hierarchy) is expected to live in PostgreSQL; starting there avoids a second migration.

### Users and roles

`users` holds `username` (unique), `password_hash`, `email`, `full_name`, nullable `hierarchy_id`, and nullable `hierarchy_type` (`daerah`, `desa`, or `kelompok`). Roles are a `roles` table (`name` unique) joined many-to-many to users. The login and current-user JSON returns `roles` as a string array. `password_hash` is never serialized.

Seeded accounts, created only when no users exist:

| Username | Password   | Roles   | Profile |
| --- | --- | --- | --- |
| `admin` | `admin123` | `Admin` | email `admin@ppgbs1.org`, full name `Administrator`, no hierarchy |
| `teacher` | `teacher123` | `Teacher` | email `fatih@ppgbs1.org`, full name `Muhammad Fatih`, `hierarchy_id` 5, `hierarchy_type` `kelompok` |

The teacher login username is `teacher`, matching the credentials printed on the login page. The demo UI currently returns `username: teacher_fatih` after a local check; the API returns the stored username `teacher`.

### Password hashing and JWT

Passwords are stored with bcrypt. Login compares with bcrypt and returns the same `401` body for an unknown username and a wrong password.

Access tokens are HMAC-SHA256 JWTs (`github.com/golang-jwt/jwt/v5`) signed with `JWT_SECRET`. Claims are the user id (`sub`) and expiry. Default lifetime is 24 hours (`JWT_TTL`). The frontend `remember` flag is not sent by `authService.login`, so the server ignores it.

Protected routes read `Authorization: Bearer <token>`. Missing or invalid tokens get `401`.

Logout is stateless: an authenticated `POST /api/auth/logout` returns `204` and the client drops the token. There is no revocation list in this change.

### CORS

Use `github.com/gin-contrib/cors`. Allow the origin in `CORS_ORIGIN` (default `http://localhost:5173`), methods `GET`, `POST`, and `OPTIONS`, and headers `Authorization` and `Content-Type`. The client sends the JWT in a header, so cookie credentials are not required.

### Configuration

Read environment variables at startup. Optional `.env` loading is for local development only; the process still fails fast when `DATABASE_URL` or `JWT_SECRET` is empty. Document the variables in `backend/.env.example`. Do not commit a real `.env`.

### Error body

Failed requests return JSON `{ "error": "<message>" }` with the HTTP status. Login validation requires username length of at least 3 and password length of at least 6, matching the login form schema, and responds `400` when that fails.

## Risks / Trade-offs

- [Stateless logout] → A stolen token stays valid until expiry. Acceptable for this init; revocation can be added later without changing the login contract.
- [AutoMigrate plus seed] → Schema drift is easy to miss once more tables exist. Fine for the first schema; later changes should add explicit migrations before production data exists.
- [Demo passwords in seed] → Only inserted into an empty database, and only for local development. Production deploys must set `JWT_SECRET` and must not rely on these accounts.
- [Teacher username differs from the in-browser demo object] → The API follows the printed credentials (`teacher`). The frontend wiring change must use the returned `user`, not the hardcoded `teacher_fatih` object.

## Migration Plan

1. Provide PostgreSQL and set `DATABASE_URL`, `JWT_SECRET`, and optionally `PORT`, `JWT_TTL`, and `CORS_ORIGIN`.
2. Run the API from `./backend`. The first start creates tables and seeds the two users.
3. Rollback is removing the `./backend` process. No existing data store is altered.

## Open Questions

- None for this change. Frontend cutover from the demo login is a follow-up.
