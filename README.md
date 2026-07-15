# Module 10 — Secure User Model & CI/CD Pipeline

A FastAPI application with a secure, database-backed user model built on
SQLAlchemy and Pydantic, with bcrypt password hashing, a full pytest suite
(unit + integration + e2e), and a GitHub Actions pipeline that tests, scans,
and deploys a Docker image to Docker Hub.

## Docker Hub

- **Image:** [`susanchapas/module10_is601`](https://hub.docker.com/r/susanchapas/module10_is601)

```bash
docker pull susanchapas/module10_is601:latest
docker run -p 8000:8000 susanchapas/module10_is601:latest
# App: http://localhost:8000  |  Health: http://localhost:8000/health
```

## Features

- **SQLAlchemy `User` model** ([app/models/user.py](app/models/user.py)) with unique
  `username` and `email` constraints, a `password_hash` column, and a `created_at` timestamp.
- **Password hashing** — `User.hash_password()` (bcrypt via passlib) and
  `User.verify_password()` to check a plain-text password against the stored hash.
- **Pydantic schemas** — `UserCreate` (username, email, password + validation) in
  [app/schemas/base.py](app/schemas/base.py) and `UserRead` (returns user details,
  omits `password_hash`) in [app/schemas/user.py](app/schemas/user.py).
- **JWT auth helpers** for token creation/verification and login.
- **Tests** — unit tests plus integration tests that run against a real Postgres
  database (uniqueness violations, invalid emails, hashing, registration, auth).

## Tech Stack

FastAPI · SQLAlchemy 2 · Pydantic 2 · PostgreSQL · passlib/bcrypt · python-jose ·
pytest · Playwright · Docker · GitHub Actions · Trivy

## Prerequisites

- Python 3.10+
- Docker + Docker Compose (for Postgres and containerized runs)

## Setup

```bash
git clone git@github.com:susanchapas/module10_is601.git
cd module10_is601

python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Running the Application

**With Docker Compose (recommended — includes Postgres + pgAdmin):**

```bash
docker compose up --build
```

**Locally with uvicorn** (requires a running Postgres — see below):

```bash
uvicorn main:app --reload
```

Open http://localhost:8000 for the calculator UI and API docs at
http://localhost:8000/docs.

## Running Tests Locally

The integration tests require a real PostgreSQL database. The simplest way is to
start the Postgres service from `docker-compose.yml`, then point the app at it via
`DATABASE_URL`.

**1. Start Postgres:**

```bash
docker compose up -d db
```

**2. Point the tests at the test database and run pytest:**

```bash
export DATABASE_URL="postgresql://postgres:postgres@localhost:5432/fastapi_db"

# Everything (unit + integration), with coverage
pytest

# Just unit tests (no database needed for the calculator logic)
pytest tests/unit/

# Just integration tests (database-backed: uniqueness, hashing, auth)
pytest tests/integration/

# End-to-end browser tests (starts the server and drives it with Playwright)
playwright install chromium
pytest tests/e2e/
```

Coverage reports are generated automatically (configured in `pytest.ini`): a
terminal summary plus an HTML report in `htmlcov/`.

> On the first e2e run, `playwright install chromium` downloads the browser.

## CI/CD Pipeline

The pipeline is defined in [.github/workflows/test.yml](.github/workflows/test.yml)
and runs on every push/PR to `main`:

1. **test** — spins up a Postgres service and runs the unit, integration, and e2e suites.
2. **security** — builds the Docker image and scans it with Trivy for CRITICAL/HIGH vulnerabilities.
3. **deploy** — on `main`, builds and pushes the image to Docker Hub as
   `susanchapas/module10_is601:latest` (and a commit-SHA tag).

### Connecting Docker Hub to this repo

The deploy job authenticates using two GitHub Actions secrets. To set them up:

1. On Docker Hub, create the repository `module10_is601` (or let the first push create it).
2. Docker Hub → **Account Settings → Security → New Access Token** (Read/Write). Copy the token.
3. In this GitHub repo → **Settings → Secrets and variables → Actions → New repository secret**, add:
   - `DOCKERHUB_USERNAME` = `susanchapas`
   - `DOCKERHUB_TOKEN` = the access token from step 2
4. Push to `main` — the workflow builds, scans, and pushes the image automatically.

## Screenshots

_Add the required proof-of-work screenshots here:_

- **GitHub Actions — successful run:** ![CI/CD passing](screenshots/github-actions.png)
- **Docker Hub — pushed image:** ![Docker Hub image](screenshots/docker-hub.png)

## Reflection

See [REFLECTION.md](REFLECTION.md) for a write-up of the development experience and
challenges faced.
