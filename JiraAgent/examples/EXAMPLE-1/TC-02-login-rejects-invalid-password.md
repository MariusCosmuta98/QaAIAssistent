# TC-02: Login rejects invalid password

Source: EX-1
Traces to: Given a registered user, when they submit an incorrect password, then the form shows "Invalid credentials" and they stay on /login.
Status: READY

## Preconditions
- A registered user account exists (email known to the tester).
- The user is signed out.

## Steps
1. Navigate to `/login`.
2. Enter the registered email in the "Email" field.
3. Enter a wrong password in the "Password" field.
4. Click "Sign in".

## Expected Result
The URL is still `/login`. An error message "Invalid credentials" is visible near the form. No navigation to `/dashboard` occurs.

## Notes
