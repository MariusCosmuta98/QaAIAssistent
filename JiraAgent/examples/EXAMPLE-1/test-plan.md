# [EX-1] User can log in with email + password

Source: EX-1 (Story, In Progress)
Goal: Registered users authenticate with their email and password on the /login page.

## Acceptance Criteria
- Given a registered user, when they submit correct email and password, then they land on /dashboard.
- Given a registered user, when they submit an incorrect password, then the form shows "Invalid credentials" and they stay on /login.
- The password field must be masked.

## Test Cases

| #     | Title                                          | Traces to AC              | Status | File                                                        |
|-------|------------------------------------------------|---------------------------|--------|-------------------------------------------------------------|
| TC-01 | User can log in with valid credentials         | AC 1                      | READY  | [TC-01-user-can-log-in-with-valid-credentials.md](TC-01-user-can-log-in-with-valid-credentials.md) |
| TC-02 | Login rejects invalid password                 | AC 2 (negative)           | READY  | [TC-02-login-rejects-invalid-password.md](TC-02-login-rejects-invalid-password.md) |
| TC-03 | Password field masks input characters          | AC 3                      | READY  | [TC-03-password-field-masks-input.md](TC-03-password-field-masks-input.md) |
