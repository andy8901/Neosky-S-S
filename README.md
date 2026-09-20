# NeoSky Service & Support

A production-oriented Android app + REST backend that lets NeoSky/TAS drone
customers securely manage everything related to their registered drones:
service tickets, flight logs, warranty, maintenance, invoices/billing, and
documents. The backend is designed so the same API can later power a web
dashboard for NeoSky's service/support team.

## Repository layout

```
android/    Kotlin + Jetpack Compose customer app (MVVM/Clean Architecture)
backend/    FastAPI REST backend (Python) + business logic
database/   PostgreSQL schema (single source of truth for all tables)
docs/       REST API specification (contract shared by backend and app)
```

## Stack

| Layer    | Technology |
|----------|------------|
| Android  | Kotlin, Jetpack Compose, Material 3, Hilt, Retrofit, Room, Coroutines/Flow, WorkManager, Firebase Cloud Messaging |
| Backend  | Python, FastAPI, SQLAlchemy 2.0, Pydantic v2, JWT auth, bcrypt |
| Database | PostgreSQL 16 |

## Quick start

1. **Database + Backend** — see [`backend/README.md`](backend/README.md).
   Fastest path: `cd backend && docker compose up` — this starts Postgres,
   applies `../database/schema.sql`, seeds demo data, and serves the API at
   `http://localhost:8000` (interactive docs at `/docs`).
2. **Android app** — see [`android/README.md`](android/README.md). Open
   `android/` in Android Studio; the debug build already points at
   `http://10.0.2.2:8000/api/` (the emulator's host-loopback address), so it
   talks to the backend from step 1 with no extra config beyond dropping in
   your own `google-services.json` for Firebase.
3. **Demo login** (seeded by `backend/scripts/seed.py`):
   `aniket@throttle.aero` / `NeoSky@123`.

## Architecture & data model

- [`database/schema.sql`](database/schema.sql) — 19 tables (users, customers,
  drones, drone_components, tickets, ticket_comments, ticket_attachments,
  flight_logs, maintenance_schedule, maintenance_records, warranties,
  warranty_claims, invoices, invoice_items, payments, service_records,
  documents, notifications, audit_logs), with every customer-owned row
  traceable back to a `customer_id` so per-customer data isolation can be
  enforced structurally rather than trusted from client input.
- [`docs/API_SPEC.md`](docs/API_SPEC.md) — the full REST contract (auth,
  customer profile, dashboard, drones, tickets, flight logs, warranty,
  maintenance, invoices, service history, documents, notifications, search,
  and an admin/engineer surface for a future web dashboard), plus the binding
  security rules (JWT + refresh tokens, bcrypt, RBAC, customer isolation,
  audit logging). The backend also serves this contract live as OpenAPI/Swagger
  at `/docs` once running.

## Security highlights

- JWT access tokens (short-lived) + refresh tokens; passwords hashed with
  bcrypt, never logged.
- Every customer-scoped API route derives `customer_id` from the
  authenticated token, never from a client-supplied id — a customer
  requesting another customer's drone/ticket/invoice by id gets `404`, not
  their data. Verified by a dedicated automated test
  (`backend/tests/test_customer_isolation.py`).
- Role-based access control (`customer`, `service_engineer`, `admin`) gates
  the admin/engineer API surface.
- All mutating admin actions are written to `audit_logs`.

## Roadmap (MVP phasing)

- **Phase 1** (this build): login, dashboard, my drones, raise ticket, ticket
  tracking, flight logging, warranty.
- **Phase 2** (this build): maintenance, service history, invoices,
  documents, notifications.
- **Phase 3** (future): deeper analytics, richer offline sync, the web-based
  service/admin dashboard (the backend's `/api/admin/*` surface is already
  designed for it), advanced maintenance analytics.

## Status

Backend: fully implemented against `docs/API_SPEC.md`, with an automated
test suite passing against real PostgreSQL (see `backend/README.md` for
results). Android app: full MVVM/Compose implementation of all screens
listed above, with unit + instrumented tests (see `android/README.md`).

---
🤖 Built with [Claude Code](https://claude.com/claude-code)
