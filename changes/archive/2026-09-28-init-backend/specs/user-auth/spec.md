## ADDED Requirements

### Requirement: Users are stored with hashed passwords

The system SHALL persist a user with a unique username, a bcrypt password hash, email, full name, a list of role names, and optional hierarchy id and hierarchy type. Hierarchy type, when present, MUST be one of `daerah`, `desa`, or `kelompok`. The password hash MUST NOT appear in any API response.

#### Scenario: Empty database is seeded

- **WHEN** the API starts against a database with no users
- **THEN** it creates user `admin` with password `admin123`, role `Admin`, email `admin@ppgbs1.org`, full name `Administrator`, and no hierarchy
- **THEN** it creates user `teacher` with password `teacher123`, role `Teacher`, email `fatih@ppgbs1.org`, full name `Muhammad Fatih`, hierarchy id `5`, and hierarchy type `kelompok`

#### Scenario: Existing users are left in place

- **WHEN** the API starts against a database that already contains users
- **THEN** it does not insert the demo accounts again

### Requirement: Login accepts username and password

The system SHALL expose `POST /api/auth/login` accepting JSON `{ "username", "password" }`. Username MUST be at least 3 characters and password MUST be at least 6 characters. A valid account MUST receive HTTP 200 and JSON `{ "token", "user" }`. The `user` object MUST include `id`, `username`, `email`, `roles`, `hierarchy_id`, `hierarchy_type`, and `full_name`. The token MUST be a JWT signed with `JWT_SECRET` whose subject is that user's id.

#### Scenario: Admin login

- **WHEN** the client posts username `admin` and password `admin123`
- **THEN** the response status is 200
- **THEN** `user.username` is `admin`, `user.roles` contains `Admin`, and `token` is a non-empty string

#### Scenario: Teacher login

- **WHEN** the client posts username `teacher` and password `teacher123`
- **THEN** the response status is 200
- **THEN** `user.roles` contains `Teacher`, `user.hierarchy_id` is `5`, and `user.hierarchy_type` is `kelompok`

#### Scenario: Invalid credentials

- **WHEN** the client posts a username that does not exist or a password that does not match
- **THEN** the response status is 401
- **THEN** the JSON body contains an `error` message and does not include a token

#### Scenario: Input too short

- **WHEN** the client posts a username shorter than 3 characters or a password shorter than 6 characters
- **THEN** the response status is 400
- **THEN** the JSON body contains an `error` message

### Requirement: Current user requires a bearer token

The system SHALL expose `GET /api/auth/me`. A request with a valid `Authorization: Bearer` token MUST return HTTP 200 and the same user fields as login, without a password hash. A missing or invalid token MUST return HTTP 401.

#### Scenario: Valid token

- **WHEN** the client calls `GET /api/auth/me` with the token from a successful login
- **THEN** the response status is 200 and `username` matches the logged-in user

#### Scenario: Missing token

- **WHEN** the client calls `GET /api/auth/me` without an `Authorization` header
- **THEN** the response status is 401

### Requirement: Logout acknowledges an authenticated session

The system SHALL expose `POST /api/auth/logout`. An authenticated request MUST return HTTP 204. A missing or invalid token MUST return HTTP 401. The server MUST NOT require a stored session to accept logout.

#### Scenario: Authenticated logout

- **WHEN** the client calls `POST /api/auth/logout` with a valid bearer token
- **THEN** the response status is 204
