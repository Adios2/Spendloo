# Spec: Registration

## Overview

Implement user registration so that new visitors can create a Spendly account. This step converts the existing `GET /register` stub (which currently only renders the empty form) into a fully working sign-up flow: the form is submitted via `POST`, the server validates the input, hashes the password, inserts the new user into the database, and redirects to the login page on success and user is shown a success message. This is the first feature that writes user data and is a prerequisite for every authenticated route that follows.

---

## Depends on

- Step 1 — Database setup (`get_db()`, `init_db()`, `users` table must exist)

---

## Routes

- `GET /register` — Render the registration form — public (already exists, extend to pass form errors back)
- `POST /register` — Handle form submission: validate, hash password, insert user, redirect — public

---

## Database changes

No new tables or columns. The existing `users` table (from Step 1) is sufficient:

| Column        | Used by registration |
|---------------|----------------------|
| name          | from form            |
| email         | from form            |
| password_hash | hashed from form     |
| created_at    | auto (default)       |

A new helper `create_user()` must be added to `database/db.py`.

---

## Templates

- **Modify:** `templates/register.html`
  - Add `method="POST"` and `action="{{ url_for('register') }}"` to the `<form>` tag
  - Add fields: `name` (text), `email` (email), `password` (password), `confirm_password` (password)
  - Render inline validation errors next to each field when present
  - Add a success flash message area (or rely on redirect to login with a flash)
  - All links must use `url_for()`

---

## Files to change

- `app.py` — add `POST /register` handler; update `GET /register` to pass errors back to template; import `create_user` from `database/db`; add `flash` and `redirect` imports
- `database/db.py` — add `create_user(name, email, password)` helper
- `templates/register.html` — wire up the form and error display
- `static/css/` — add `register.css` for any page-specific styles (do not use inline `<style>` tags)

---

## Files to create

- `static/css/register.css` — page-specific styles for the registration form

---

## New dependencies

No new dependencies. `werkzeug.security.generate_password_hash` is already available via the existing `werkzeug` install.

---

## Rules for implementation

- No SQLAlchemy or ORMs
- Parameterised queries only — never f-strings in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash` — never store plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- DB logic lives exclusively in `database/db.py` — the route function must not contain any SQL
- Use `abort(400)` or re-render with errors — never `return "error string"`
- Duplicate email must be caught and shown as a user-friendly error (catch `sqlite3.IntegrityError`)
- The `POST /register` route must redirect to `url_for('login')` on success, with a flash message
- `app.secret_key` must be set for `flash()` to work — add it in `app.py` if missing
- Do **not** implement login, session, or any authenticated route — that is Step 3

### Validation rules (server-side)

| Field            | Rule                                      |
|------------------|-------------------------------------------|
| name             | Required, 1–100 characters                |
| email            | Required, must contain `@`                |
| password         | Required, minimum 8 characters            |
| confirm_password | Must match password                       |

---

## Definition of done

- [ ] `GET /register` renders the registration form without errors
- [ ] Submitting the form with all valid fields creates a new row in the `users` table with a hashed password
- [ ] After successful registration the browser is redirected to `/login` and a flash message is visible
- [ ] Submitting with a duplicate email shows an inline error and does **not** create a duplicate row
- [ ] Submitting with mismatched passwords shows a validation error and does not write to the database
- [ ] Submitting with any required field empty shows a validation error for that field
- [ ] Submitting with a password shorter than 8 characters is rejected with an error
- [ ] Plaintext password is never stored — `password_hash` column contains a werkzeug hash
- [ ] No SQL is written inside `app.py` — all DB access goes through `database/db.py`
- [ ] The registration form and all links use `url_for()` — no hardcoded URLs
- [ ] App starts and all existing routes (`/`, `/login`) remain functional
