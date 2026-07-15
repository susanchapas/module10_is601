# Reflection — Module 10

## Overview

In this module I extended the FastAPI project with a secure, database-backed user
model. The goal was to store users safely (never plain-text passwords), validate
input with Pydantic, prove it works with automated tests against a real database,
and wire up a CI/CD pipeline that ships a Docker image to Docker Hub.

## What I Built

- A SQLAlchemy `User` model with unique constraints on `username` and `email`, a
  `password_hash` column, and timestamps.
- bcrypt-based `hash_password` / `verify_password` methods so raw passwords are
  hashed before storage and verified without ever comparing plain text.
- Pydantic schemas: `UserCreate` for incoming registration data (with password
  rules — length, upper/lower/digit) and `UserRead` for responses, which
  deliberately omits `password_hash`.
- Unit tests for the schema validation and hashing logic, and integration tests
  that run against PostgreSQL to confirm uniqueness violations raise
  `IntegrityError`, registration rejects duplicates and weak passwords, and
  authentication issues a token.
- A GitHub Actions workflow with three stages — test (with a Postgres service
  container), security scan (Trivy), and deploy (push to Docker Hub).

## Challenges

- **Hashing vs. verification.** The trickiest concept was keeping the boundary
  clear: the schema accepts a `password`, but the model only ever stores a
  `password_hash`. Getting a test to store a pre-hashed value while other tests
  pass raw passwords forced me to be precise about which layer hashes.
- **Testing against a real database.** Unit tests are easy; integration tests
  need a live Postgres. Understanding the fixtures that create/drop tables and
  truncate between tests (for isolation) took some reading, and I had to make
  sure `DATABASE_URL` pointed at the right database both locally and in CI.
- **Uniqueness constraints.** Verifying that duplicate emails/usernames fail
  meant intentionally triggering `IntegrityError` and then rolling back the
  session so the rest of the test could continue — a good lesson in transactions.
- **CI/CD and Docker Hub.** The deploy step failed until I understood it needed
  `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` secrets and had to target *my own*
  Docker Hub repository. I also fixed the container health check, which was
  calling `curl` (not installed in the slim image) — I switched it to a small
  Python one-liner hitting a new `/health` endpoint.

## Takeaways

Security has to be built into the data model, not bolted on afterward. Automated
tests against a real database gave me real confidence that constraints and hashing
actually behave as intended, and a green CI/CD pipeline that builds, scans, and
publishes an image made the whole thing feel like a real deployment rather than
just code on my laptop.
