# TC-03: Password field masks input characters

Source: EX-1
Traces to: The password field must be masked.
Status: READY

## Preconditions
- The user is on `/login`.

## Steps
1. Click into the "Password" field.
2. Type any 8-character string.

## Expected Result
Each typed character is rendered as a mask character (e.g. `•` or `*`) rather than the plaintext character. The underlying input value still equals the typed string when submitted.

## Notes
