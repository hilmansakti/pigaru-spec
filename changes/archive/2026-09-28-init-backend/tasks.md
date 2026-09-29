## 1. Module and configuration

- [x] 1.1 Create the Go module `pigaru-backend` in `./backend` with `cmd/api` and `internal/` packages for config, database, models, handlers, middleware, and routing
- [x] 1.2 Add Gin, `gin-contrib/cors`, GORM, the PostgreSQL driver, `golang-jwt/jwt/v5`, and `golang.org/x/crypto`
- [x] 1.3 Load `PORT`, `DATABASE_URL`, `JWT_SECRET`, `JWT_TTL`, and `CORS_ORIGIN` from the environment, with defaults for port (`8080`), token lifetime (`24h`), and CORS origin (`http://localhost:5173`), and exit on startup when `DATABASE_URL` or `JWT_SECRET` is empty
- [x] 1.4 Add `backend/.env.example` documenting those variables and keep a real `.env` out of git

## 2. Persistence

- [x] 2.1 Define GORM models for users and roles, including unique username, bcrypt password hash excluded from JSON, email, full name, nullable hierarchy id and type, and a many-to-many role list
- [x] 2.2 Connect to PostgreSQL and run AutoMigrate for users and roles before the server accepts traffic
- [x] 2.3 Seed `admin` / `admin123` and `teacher` / `teacher123` only when the users table is empty, with the profiles in the design

## 3. HTTP server

- [x] 3.1 Mount routes on `/api` and add `GET /api/health` returning HTTP 200 and `{ "status": "ok" }`
- [x] 3.2 Apply CORS for the configured origin, methods `GET`, `POST`, and `OPTIONS`, and headers `Authorization` and `Content-Type`
- [x] 3.3 Start Gin on the configured port from `cmd/api`

## 4. Authentication

- [x] 4.1 Implement bcrypt hashing and JWT issue/parse using `JWT_SECRET`, with the user id as the subject
- [x] 4.2 Add bearer middleware that rejects missing or invalid tokens with HTTP 401 and a JSON `error` body
- [x] 4.3 Implement `POST /api/auth/login` with length checks (username at least 3, password at least 6), identical 401 responses for unknown users and bad passwords, and a 200 body `{ "token", "user" }` matching the frontend user fields
- [x] 4.4 Implement `GET /api/auth/me` and `POST /api/auth/logout` for authenticated callers (`200` user JSON and `204` respectively)

## 5. Verification

- [x] 5.1 Confirm the process refuses to start without `DATABASE_URL` or `JWT_SECRET`
- [x] 5.2 Confirm login, current user, logout, invalid credentials, and short input against a running PostgreSQL database
