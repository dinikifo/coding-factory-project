# Property Operations CRM

A minimal CRM prototype for short term property operations with:
- SQLite relational schema + constraints and triggers
- Validator layer (one validator per table)
- CRUD repository layer
- **JWT auth via HttpOnly cookies + CSRF protection** (admin/user roles)
- Bottle-based REST API with OpenAPI docs
- Bootstrap/HTML frontend (CRUD, Login, Pipeline Map)
- Unit & integration tests (pytest)

---

## Requirements

- Python **3.10+**
- SQLite (bundled `airbnb.db`)

## Installation

1) **Unzip the project**

2) **(Dev HTTPS) mkcert certificates** (optional but recommended for `Secure` cookies in local dev):

mkdir -p certs
mkcert -cert-file certs/cert.pem -key-file certs/key.pem localhost 127.0.0.1 ::1
cp "$(mkcert -CAROOT)/rootCA.pem" certs/rootCA.pem
chmod 600 certs/key.pem

Secrets & users
Use the helper to set the JWT secret and create users (admin + regular):

# set secret key
python3 SECRET_USERS_PW_ADMIN.py SECRET_KEY

# set admin password
python3 SECRET_USERS_PW_ADMIN.py admin <NEW_ADMIN_PASSWORD>

# create/update a user (example: jane)
python3 SECRET_USERS_PW_ADMIN.py jane <NEW_USER_PASSWORD>

Install dependencies:
bottle
PyJWT
pytest       


Database

Use the provided airbnb.db.
To rebuild from SQL (if provided):

reference the appendix of the pdf for the sql create statements, triggers and complete schema.

Running the App

python APP_MAIN.10.py

Open frontend pages in your browser:

    index.html (Login & navigation)

    create-update.html, read.html

    pipeline-map.html

    docs.html (API docs)

Authentication (cookies + CSRF)

Model (server):

    On successful login, server sets two HttpOnly cookies:

        access_token (~30 min)

        refresh_token (~14 days)

    Cookies use SameSite=Lax; use HTTPS in real deployments (Secure).

    A /refresh endpoint issues a new access token using the refresh cookie.

    Logout clears cookies and revokes refresh token.

Model (client):

    For all requests, the browser automatically includes cookies.

    For state-changing requests (POST/PUT/DELETE), include a CSRF header:

        X-CSRF-Token: <value provided at login> (the login response or a separate /csrf route provides it).

    When using fetch, always pass credentials: "include".

API & Docs

    OpenAPI served at /openapi and docs at /docs.

    Contract uses DD.MM.YYYY date format in this MVP.

Tests

Run:

pytest -q
# with coverage:
pytest --cov=. --cov-report term-missing --cov-report html

Tests in the black.box folder require mkcert credentials. Please place a copy in the folder before running.

Current Release: v1.0.0

Covers the current database schema (12 tables, including owner_property_map with map_id as PK).

Validators and OpenAPI 3.1 are aligned with this schema.

Includes authentication (JWT in cookies + CSRF), CRUD endpoints, and error envelope handling.

See docs.html and openapi-3.1.yaml for API reference.

This document is the source of truth for v1.0.0.


Recommended additions:

    Auth matrix (401/403/200), refresh, logout revocation

    Error envelope shape for 4xx/409

    Range GET bounds & limits

    Date edge cases (invalid calendar dates)

    Database normalization (3NF): refactor of lookup tables, FKs, and uniqueness constraints.

    Temporal Owner–Property Contracts: replacing owner_property_map with owner_property_contract, adding start_date, end_date, status, and enforcing no overlapping intervals.

     Validators & DTOs: migration to DTOs as the single source of OpenAPI schemas, reducing drift.

     OpenAPI regeneration: spec auto-generated from DTOs in the codebase.

     Testing enhancements: contract-drift tests (DTO ↔ Validator ↔ OpenAPI) and additional role-matrix black-box tests.

Future Work

    Postgres + SQLAlchemy + Alembic

    CI/CD & containerization

    Pagination/filtering; ISO-8601 dates

    RBAC expansion

Roadmap

v1.1 (planned): temporal owner–property contracts, new migration script, updated tests.

v2.0 (planned): relational database use, CI/CD  & containerization, Pagination/filtering; ISO-8601 dates, RBAC expansion, full 3NF refactor, DTO-driven OpenAPI, expanded dashboard and reporting endpoints.

Deprecations

owner_property_map is deprecated and will be superseded by owner_property_contract in v1.1.

A compatibility view will be provided to smooth the transition.
License

Educational use (final project).