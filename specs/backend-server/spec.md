## Purpose

TBD

## Requirements

### Requirement: API process lives in the backend folder

The system SHALL provide a Go module in `./backend` that starts an HTTP server with Gin.

#### Scenario: Server starts

- **WHEN** the API process starts with a valid configuration
- **THEN** it listens for HTTP requests on the configured port, defaulting to `8080`

### Requirement: Routes are served under /api

The system SHALL mount application routes on the `/api` path prefix.

#### Scenario: Health path

- **WHEN** a client requests `GET /api/health`
- **THEN** the server responds with HTTP 200 and a JSON body that includes `"status": "ok"`

### Requirement: PostgreSQL persistence through GORM

The system SHALL connect to PostgreSQL with GORM using `DATABASE_URL` and MUST refuse to start when `DATABASE_URL` is missing.

#### Scenario: Missing database URL

- **WHEN** the process starts without `DATABASE_URL`
- **THEN** it exits with an error and does not listen for requests

#### Scenario: Schema on startup

- **WHEN** the process starts with a reachable database
- **THEN** it migrates the tables required for users and roles before accepting traffic

### Requirement: CORS allows the frontend origin

The system SHALL apply CORS so a browser app on the configured origin can call the API. The allowed origin MUST come from `CORS_ORIGIN`, defaulting to `http://localhost:5173`. Allowed methods MUST include `GET`, `POST`, and `OPTIONS`. Allowed request headers MUST include `Authorization` and `Content-Type`.

#### Scenario: Browser preflight from the Vite origin

- **WHEN** the frontend origin sends an `OPTIONS` preflight for an `/api` route with `Authorization` and `Content-Type`
- **THEN** the server responds with CORS headers that allow that origin, those methods, and those headers

### Requirement: Required secrets fail fast

The system SHALL read `JWT_SECRET` from the environment and MUST refuse to start when it is empty.

#### Scenario: Missing JWT secret

- **WHEN** the process starts without `JWT_SECRET`
- **THEN** it exits with an error and does not listen for requests
