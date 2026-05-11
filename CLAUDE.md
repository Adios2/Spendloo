# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Activate the virtual environment (required before running anything)
source venv/bin/activate

# Run the development server (port 5001)
python app.py

# Alternatively via Flask CLI
flask run --port 5001

# Run all tests
pytest

# Run a single test file
pytest tests/test_auth.py

# Run a single test by name
pytest -k "test_login"
```

## Architecture

This is **Spendly**, a Flask + SQLite personal expense tracker built as a step-by-step learning project. Routes and features are added incrementally across defined "Steps".

**`app.py`** — single file containing the Flask app instance and all route definitions. Current live routes: `/`, `/register`, `/login`, `/terms`, `/privacy`. Placeholder routes (returning strings) exist for `/logout`, `/profile`, `/expenses/add`, `/expenses/<id>/edit`, `/expenses/<id>/delete` — these are implemented in later steps.

**`database/db.py`** — stub file to be filled in during Step 1. Must provide:
- `get_db()` — SQLite connection with `row_factory = sqlite3.Row` and `PRAGMA foreign_keys = ON`
- `init_db()` — creates all tables using `CREATE TABLE IF NOT EXISTS`
- `seed_db()` — inserts sample rows for development

**`templates/`** — Jinja2 templates. `base.html` defines the full page shell (navbar, `<main>`, footer, Google Fonts, `style.css`, `main.js`). All other templates extend it via `{% extends "base.html" %}` and fill `{% block content %}`. Page-specific scripts go in `{% block scripts %}`.

**`static/css/style.css`** — single stylesheet for the entire app (no build step, no preprocessor).

**`static/js/main.js`** — single JS file loaded on every page. Page-specific inline scripts live in `{% block scripts %}` in each template (see the video modal in `landing.html` for the pattern).

## Key conventions

- Currency is Indian Rupees (₹); keep this consistent in any UI copy.
- The app runs on **port 5001** (not the Flask default 5000) to avoid macOS AirPlay conflicts.
- Database file will live at the project root (e.g., `expenses.db`) — already `.gitignore`d.
- `pytest-flask` is installed; tests should use its `client` fixture against the Flask test client.
