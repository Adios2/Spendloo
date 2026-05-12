# Spec: Login and Logout

## Overview
This feature implements credential-based login and session-backed logout for Spendly. Users submit their email and password via `POST /login`; the route verifies the password hash, writes the user's id to the Flask session, and redirects to a protected dashboard (or back to the form on failure). `GET /logout` clears the session and redirects to the landing page. This is the authentication foundation that all subsequent protected routes (Step 4 onward) will rely on.

## Depends on
- Step 1 — Database Setup (`get_db()`, users table with `password_hash` column)
- Step 2 — Registration (`create_user()`, confirmed that `password_hash` is stored with werkzeug)

## Routes
- `POST /login` — Validate email+password, set session, redirect to `/profile` on success or re-render form with error — public
- `GET /logout` — Clear session and redirect to `/` — public (no auth guard needed; clearing an empty session is harmless)

## Database changes
No new tables or columns. One new helper function must be added to `database/db.py`:

- `get_user_by_email(email)` — SELECT by email, returns a `sqlite3.Row` or `None`

## Templates
- **Modify:** `templates/login.html` — add `<form method="POST">` with email + password fields, CSRF-free submit, flash message rendering, and error display consistent with register.html
- **Modify:** `templates/base.html` — add conditional nav links: show "Login" / "Register" when no session, show "Profile" / "Logout" when `session.user_id` is set

## Files to change
- `app.py` — convert `GET /login` stub to `GET+POST`, implement `GET /logout`
- `database/db.py` — add `get_user_by_email(email)`
- `templates/login.html` — wire up form (POST, fields, errors, flash)
- `templates/base.html` — conditional nav based on session

## Files to create
- `static/css/login.css` — page-specific styles for the login form (mirrors pattern from `register.css`)

## New dependencies
No new dependencies. `check_password_hash` is already available via `werkzeug.security` (installed with Flask).

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only (`?` placeholders — never f-strings in SQL)
- Passwords verified with `check_password_hash` from `werkzeug.security`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Session stores only `user_id` (integer) — never store the password or hash in the session
- On failed login: re-render `login.html` with a generic error ("Invalid email or password") — do not reveal which field was wrong
- Redirect after successful login uses `url_for()` — never a hardcoded path
- `GET /logout` must call `session.clear()` (not `session.pop`) to wipe all keys
- No `@login_required` decorator in this step — that pattern is introduced when the first protected route is built (Step 4)

## Definition of done
- [ ] Visiting `GET /login` renders the login form
- [ ] Submitting `POST /login` with valid credentials (e.g. demo@spendly.com / demo123) redirects to `/profile` stub
- [ ] Submitting `POST /login` with a wrong password re-renders the form with a generic error message and does not set a session
- [ ] Submitting `POST /login` with an unknown email re-renders the form with the same generic error message
- [ ] After a successful login, `session["user_id"]` is set to the correct integer id
- [ ] Visiting `GET /logout` clears the session and redirects to `/`
- [ ] After logout, `session["user_id"]` is no longer present
- [ ] `base.html` nav shows "Login" and "Register" links when no session exists
- [ ] `base.html` nav shows "Profile" and "Logout" links when a session exists
- [ ] No raw SQL appears in `app.py` — all DB access goes through `database/db.py`
- [ ] `get_user_by_email()` uses a parameterised query
- [ ] Plaintext passwords are never stored or logged
