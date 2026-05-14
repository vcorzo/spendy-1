# Spec: Registration

## Overview
Implement user registration so new visitors can create a Spendly account. This step upgrades the existing stub `GET /register` route into a fully functional form that accepts a POST, validates input server-side, hashes the password, and inserts a new row into the `users` table. On success the user is shown with a sucess message and them redirected to the login page. This is the entry point for all authenticated features that follow.

## Depends on
- Step 01 — Database setup (`users` table and `get_db()` must be in place)

## Routes
- `GET /register` — render registration form — public (upgrade existing stub)
- `POST /register` — process form, validate input, insert user, redirect to `/login` — public

## Database changes
No new tables or columns. The existing `users` table (`id`, `name`, `email`, `password_hash`, `created_at`) covers all requirements.

A new DB helper must be added to `database/db.py`:
- `create_user(name, email, password)` — hashes the password with `werkzeug`, inserts a row into `users`, returns the new `id`. Raises `sqlite3.IntegrityError` if the email is already taken (UNIQUE constraint).

## Templates
- **Modify:** `templates/register.html`
  - Add `method="post"` and `action="{{ url_for('register') }}"` to the form
  - Add `name` attributes to all inputs: `name`, `email`, `password`, `confirm_password`
  - Add a flash message block to display validation errors
  - Keep all existing visual design

## Files to change
- `app.py` — upgrade `register()` to handle `GET` and `POST`; add `app.secret_key`; add flash + redirect logic
- `database/db.py` — add `create_user()` helper
- `templates/register.html` — wire up form action/method and flash message display

## Files to create
None.

## New dependencies
No new dependencies. Uses `werkzeug.security` (already installed) and Flask's built-in `flash`, `redirect`, and `url_for`.

## Rules for implementation
- No SQLAlchemy or ORMs — use raw `sqlite3` via `get_db()`
- Parameterised queries only — never use f-strings in SQL
- Hash passwords with `werkzeug.security.generate_password_hash` — never store plaintext
- `app.secret_key` must be set in `app.py` for `flash()` to work (hardcoded dev string is fine for now)
- Server-side validation must check in this order:
  1. All fields are non-empty
  2. `password == confirm_password`
  3. Email not already registered — catch `sqlite3.IntegrityError`
- On any validation failure: re-render the form with a flashed error, do not redirect
- On success: flash a success message and `redirect(url_for('login'))`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Use `url_for()` for every internal link — never hardcode URLs

## Definition of done
- [ ] `GET /register` renders the registration form without errors
- [ ] Submitting with all valid fields inserts a new row in `users` and redirects to `/login`
- [ ] Submitting with mismatched passwords re-renders the form with an error, no DB insert
- [ ] Submitting with an already-registered email re-renders with "Email already registered" error
- [ ] Submitting with any empty field re-renders with a validation error
- [ ] Password is stored as a hash — never plaintext — verifiable by inspecting `spendly.db`
- [ ] No duplicate user is created on repeated valid submissions with the same email
