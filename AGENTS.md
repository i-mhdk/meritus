# AGENTS.md

## Shape of the codebase
- Single-file Flask app: **all** config, models, and routes live in `app/main.py` (~2360 lines). There is no blueprint/package split; add new models and routes there. Models are defined inline (no `models.py`).
- Frontend is vanilla ES modules — **no build step, no npm**. `app/templates/dashboard.html` loads `app/static/js/dashboard.js`, which imports feature modules from `app/static/js/modules/*.js`. New UI = new JS module + new `/api/*` route.
- Server-rendered templates are only `index.html`, `login.html`, `signup.html`; everything else is a JS-driven SPA shell.

## Local run (needs both Postgres and mail)
1. `.env` in repo root (gitignored) with `DATABASE_URL="postgresql://...:5432/..."` — app uses `load_dotenv()`.
2. Email confirmation on signup requires a local SMTP debug server in a separate terminal:
   `python -m smtpd -n -c DebuggingServer localhost:8025`
   (stdlib `smtpd` is deprecated/removed on Python ≥3.12 — `pip install aiosmtpd` if your interpreter is newer).
3. Start with `python app/main.py` (runs `db.create_all()` + debug server on :5000). The DB must exist first; there is no auto-create.

## Migrations (Flask-Migrate / Alembic)
Always pass the app path — bare `flask db ...` may fail:
- `flask --app app/main.py db migrate -m "desc"`
- `flask --app app/main.py db upgrade`
Generate/apply a migration whenever you change a model. Manual table edits won't be reflected until you do.

## Gotchas
- Job eligibility logic is **duplicated**: helpers `_eligibility_profile()` / `_is_eligible_for_job()` (app/main.py:766) and an inlined copy inside `browse_jobs()` (app/main.py:1703). Update both when changing eligibility rules.
- Test/interview system only supports type `'Q'` questionnaires (type `'T'` exams are rejected as unavailable); questions persist via `_persist_questions()` and are locked once a test has submissions.
- No automated tests, linter, typecheck, or CI exist in this repo — don't manufacture `test`/lint steps.