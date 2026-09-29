## 1. API client

- [x] 1.1 Set `VITE_API_URL` in `frontend/.env` to `http://localhost:8080/api`
- [x] 1.2 Skip session clear and the `/login` redirect in the Axios 401 interceptor when the failed request is `/auth/login`, and still reject the promise

## 2. Login page

- [x] 2.1 Replace the hardcoded admin and teacher branch in `LoginPage` with `authService.login({ username, password })`
- [x] 2.2 On success, store the returned user and token, toast the user's full name, and navigate to the saved path or `/dashboard`
- [x] 2.3 On failure, stay on the login page, toast the API `error` string or a generic message when that body is missing, and clear the loading state

## 3. Verification

- [x] 3.1 Restart the Vite dev server so it picks up `VITE_API_URL`, with the API running and `CORS_ORIGIN` matching the Vite origin
- [x] 3.2 Sign in as `admin` / `admin123` and confirm the dashboard session username is `admin`
- [x] 3.3 Sign in as `teacher` / `teacher123` and confirm role `Teacher`, hierarchy id `5`, hierarchy type `kelompok`, and username `teacher`
- [x] 3.4 Submit a wrong password and confirm the page stays on `/login`, shows the API error, and stores no token
