## Why

The login page still accepts `admin` and `teacher` inside the browser and stores a fake token. The Go API already checks those credentials and returns a real JWT, so the screen should use that response.

## What Changes

- Submit the login form through `authService.login` and store the returned user and token.
- Show the API error message when login fails, and keep the user on the login page.
- Stop the Axios 401 interceptor from reloading `/login` when the failed request is the login call itself.
- Point `VITE_API_URL` at `http://localhost:8080/api`, matching the API base path. The current frontend env value is `http://localhost:3000`.
- Leave logout, token refresh, password reset, and the seeded demo-credential hint as they are.

## Capabilities

### New Capabilities

- `login-ui`: The login page authenticates against `POST /api/auth/login` and starts a session from the API user and token.

### Modified Capabilities

- None. `backend-server` and `user-auth` already describe the API this screen will call.

## Impact

- `frontend/src/pages/LoginPage.tsx` replaces the hardcoded demo branch.
- `frontend/src/services/api.ts` changes 401 handling so a rejected login does not force a navigation.
- `frontend/.env` updates `VITE_API_URL`. No backend routes or response shapes change.
