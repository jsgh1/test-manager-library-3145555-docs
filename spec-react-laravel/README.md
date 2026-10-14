# Manager Library — React + Laravel implementation specification

## Sources and interpretation

This is a **stack adaptation**, not a change of product scope. Read the reference `spec-python/spec.md`, `data-model.md`, `api-contract.md`, `tasks.md` and ADR-001…004 as the authoritative business and data contracts. The .NET variant describes the same product. Only framework/runtime-specific prescriptions are replaced here.

**Actors:** `admin` manages users, books, members and reports; `librarian` manages loans/returns and reads books/members/overdues; a `member` is data, not an account. Admin is provisioned from environment only.

**In scope:** five fixed categories, ISBN-13 books with optional covers, active-only searchable catalogue, members, 1–3 line loans, per-item return, overdues, historical most-borrowed report, JWT login. **Out of scope:** fines, reservations, renewals, emails, multi-branch, import, microservices, multitenancy or additional book attributes.

## Technology mapping

| Reference concern | React / Laravel decision |
| --- | --- |
| FastAPI / ASP.NET controllers | Laravel HTTP routes and thin request/response adapters |
| Python dataclasses / C# domain classes | Plain PHP value objects and domain classes with no Laravel or persistence dependencies |
| Python/C# use cases + ports | PHP application services with interfaces defined in the application layer |
| SQLAlchemy/Alembic or EF migrations | Laravel migrations in **api**, using PostgreSQL 16 schema-qualified singular business tables |
| Dependency injection composition | Laravel service provider / composition root, no framework imports in domain |
| Password hashing | Laravel-compatible password hashing behind a port (Argon2id where available); never plaintext storage |
| JWT | Signed HS256 bearer JWT with 8h expiry; secret from environment, `sub` and `role` only |
| Storage | Opaque `cover_key`, local disk adapter in development, pluggable object store |
| UI | React application, bearer token, role-restricted views, exhaustive error formatting |
| Test tools | PHPUnit/Pest for PHP unit, HTTP, integration and architectural checks; concurrent PostgreSQL integration scenario |

## Domain invariants (unchanged)

- Active ISBN uniqueness; `total_copies >= 1`; `0 <= available_copies <= total_copies`; soft-delete active books only; cannot withdraw a book with open loan lines.
- Each loan has 1–3 **distinct** books; member may have at most three unreturned lines **across all loans**; overdue blocks new loans; each line decrements exactly one copy atomically; due date is exactly 14 days after loan timestamp.
- Return only once per line, incrementing available copies once; statuses `open`, `overdue`, `closed` are calculated, never stored.
- Loan item snapshot stores `book_title`, `isbn`, `category_name` at checkout; reports aggregate historical snapshot by `book_id + book_title + category_name` and never expose members/staff.
- No direct SQL writes from `tool`; no migration DDL from `infra`; no plaintext secrets in code, logs, or HTTP response.

## API contract

The canonical [reference API contract](../spec-python/api-contract.md) stays valid: base `/api/v1`; snake_case JSON; UUID v4; UTC `Z` timestamps; `page=1&size=20` (max 50); unified `{code,message,details}` errors with 401/403/404/409/422 as specified. Authentication exempt only for `POST /auth/login` and `GET /health`. A cover binary GET is also specified in prose, so the Laravel HTTP adapter must implement it.

## Physical data model

**Exactly six business tables**, in PostgreSQL schema `library`: `category`, `book`, `member`, `loan`, `loan_item`, `user`. UUID PKs supplied by application; **no column DEFAULT**, no `created_at` / `updated_at`; mandatory FK actions and indexes, partial unique ISBN on active books, and engine-enforced range check. Five categories use the canonical IDs and original names `Novela`, `Ciencia`, `Tecnología`, `Historia`, `Infantil`. They are identifiers/data values, **not prose to translate**.

## Architectural rules

Domain imports no Laravel/facades/DB classes. Application owns outbound ports; adapters implement ports; composition root wires the two. Use transactions and database-enforced protections for copy counts and returns. Cover deletion persists `cover_key = null` first, commits, then deletes the blob.

## Open issues

Read [decisions.md](decisions.md) BEFORE implementing behavior where original sources disagree. Reconcile the resulting change with acceptance evidence. Do not declare unresolved scenarios as passing.
