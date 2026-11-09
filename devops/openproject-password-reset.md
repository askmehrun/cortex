# OpenProject Password Reset (shell)

Reset the admin password of a self-hosted OpenProject when you have shell access but lost the credentials, by setting the password directly through the Rails runner. Works both under Docker and for the system-package install (`sudo apt install openproject`).

## When to Use
- The admin password is lost and the password-reset email flow is not available.
- You are already at a shell on the host or inside the container, and the `openproject` command is on PATH.
- The `openproject` shell command means it is a system-package install. Under Docker use `docker exec -it <container> openproject run ...` instead of bare `openproject run`.

## How
1. Confirm the admin login exists (default is `admin`):
   ```bash
   openproject run rails runner "User.all.each { |u| puts \"#{u.login} admin=#{u.admin?} email=#{u.email}\" }"
   ```
2. Set a new password for the admin user. The value must satisfy the password policy — lowercase, uppercase, numeric, and a special character:
   ```bash
   openproject run rails runner "u = User.find_by(login: 'admin'); u.password = 'NewPass_123'; u.password_confirmation = 'NewPass_123'; u.save!"
   ```
   If the login is not `admin`, replace it in `find_by(login: ...)`. If the user is missing, create it with `User.admin.create!(login: 'admin', password: 'NEW', password_confirmation: 'NEW')`.
3. Log in at the project URL with the admin login and the new password.
4. If the user shows as deactivated, reactivate it:
   ```bash
   openproject run rails runner "User.find_by(login: 'admin').update!(active: true)"
   ```

## Gotchas
- Password policy rejects simple values like `NewPassword123` (no special character). Every character class is enforced.
- The `!` character triggers bash history expansion, so even inside double quotes a trailing `!` produces `-bash: !': event not found`. Use a different special character such as `_` or `-` instead.
- The `Using unprefixed environment variables is deprecated` line printed before the run is a harmless warning; the command still executes.
- The runner sets the password but does not change the login, so keep the existing username.

## Rule
The password must contain a special character, and the special character must not be `!` — bash history expansion breaks it.
