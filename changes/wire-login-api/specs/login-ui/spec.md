## ADDED Requirements

### Requirement: Login form calls the auth API

The login page SHALL submit username and password to `POST /api/auth/login` through the existing auth client. The request base URL MUST be `http://localhost:8080/api`. On HTTP 200, the page MUST store the returned `user` and `token` as the authenticated session and navigate to the originally requested path, or `/dashboard` when no path was stored.

#### Scenario: Admin signs in

- **WHEN** the user submits username `admin` and password `admin123` while the API is running
- **THEN** the app stores the returned token and user
- **THEN** the user lands on the dashboard and the session user username is `admin`

#### Scenario: Teacher profile comes from the API

- **WHEN** the user submits username `teacher` and password `teacher123`
- **THEN** the stored user has role `Teacher`, hierarchy id `5`, and hierarchy type `kelompok`
- **THEN** the stored username is the API value `teacher`

### Requirement: Failed login stays on the page

A failed login MUST NOT navigate away from the login page and MUST NOT clear a session by reloading `/login`. The page MUST show the API `error` string when the response body includes one. The submit control MUST leave the loading state after the request finishes.

#### Scenario: Wrong password

- **WHEN** the user submits a known username with a password the API rejects
- **THEN** the page remains on the login route
- **THEN** an error toast shows the API error message and no token is stored

#### Scenario: API unreachable

- **WHEN** the login request fails without an API error body
- **THEN** the page remains on the login route and shows a generic failure message
