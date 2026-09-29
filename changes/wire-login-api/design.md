## Context

`LoginPage` checks `admin` / `admin123` and `teacher` / `teacher123` locally, then calls `setAuth` with a hardcoded user and a `demo-token`. `authService.login` already posts `{ username, password }` to `/auth/login` and expects `{ token, user }`. The Axios client base URL is `VITE_API_URL`, which is currently `http://localhost:3000`. The API listens at `http://localhost:8080/api`.

The response interceptor treats every HTTP 401 as an expired session: it clears auth and assigns `window.location.href = '/login'`. A failed login is also a 401, so that interceptor would reload the page before the form can show the API error.

## Goals / Non-Goals

**Goals:**

- Sign in through `POST /api/auth/login` and persist the returned user and token.
- Stay on the login page and show the API `error` message when credentials are rejected.
- Send login requests to `http://localhost:8080/api`.

**Non-Goals:**

- Calling `POST /api/auth/logout` from the sidebar. Logout keeps clearing local state only.
- Implementing `POST /api/auth/refresh` or the "Ingat saya" flag. The checkbox stays in the form and is not sent.
- Password reset, registration, or changing the seeded credential hint on the page.
- Changing API routes or the user JSON shape.

## Decisions

### Call the existing auth service

`onSubmit` calls `authService.login({ username, password })`, then `setAuth(user, token)`, a success toast that uses `user.full_name`, and the existing redirect to `from` or `/dashboard`.

Alternative: keep a local `fetch` in the page. Rejected because the service already matches the API contract and attaches JSON headers.

### Do not redirect on a login 401

The response interceptor skips `clearAuth` and the `/login` navigation when the failed request URL ends with `/auth/login`. It still rejects the promise so the page can read `error.response.data.error`. Other 401 responses keep the current session-clear and redirect.

Alternative: remove all 401 redirects. Rejected because protected calls should still send an expired token back to the login page.

### API base URL

Set `VITE_API_URL` in `frontend/.env` to `http://localhost:8080/api`. The code fallback in `api.ts` is already that value; the env file is what overrides it today.

The Vite dev server must be restarted after the env change. Backend `CORS_ORIGIN` must be the Vite origin (default `http://localhost:5173`).

### Surface the API error

On failure, if the response body has an `error` string, show that string in the error toast. If the request fails without that body, show a generic failure message. Always clear the loading state in `finally`.

## Risks / Trade-offs

- [Env not reloaded] → Login still targets port 3000 until the Vite process restarts. The verification step restarts or reloads the dev server before trying a login.
- [CORS origin mismatch] → The browser blocks the call if the Vite port is not `CORS_ORIGIN`. Confirm the running origin before treating a network error as an auth bug.
- [Teacher display name] → The API user username is `teacher`, while the old demo object used `teacher_fatih`. The UI must render `user` from the response, not the old hardcoded object.

## Migration Plan

1. Update the env value and the login page, then restart `yarn dev`.
2. Sign in as `admin` / `admin123` and confirm the dashboard loads with the stored token.
3. Rollback is restoring the demo branch in `LoginPage` and the previous `VITE_API_URL`.

## Open Questions

- None.
