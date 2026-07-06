# TC-01: User can log in with valid credentials

Source: EX-1
Traces to: Given a registered user, when they submit correct email and password, then they land on /dashboard.
Status: READY

## Preconditions
- A registered user account exists (email + password known to the tester).
- The user is signed out.

## Steps
1. Navigate to `/login`.
2. Enter the registered email in the "Email" field.
3. Enter the correct password in the "Password" field.
4. Click "Sign in".

## Expected Result
The browser navigates to `/dashboard` and the user's name is visible in the top-right menu.

## Notes
