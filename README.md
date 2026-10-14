# Manager Library — Specification Repository

**Technical assessment:** October 14, 2026, 06:50–11:00 America/Bogota. Submit work to the learner's six forks before the cutoff.

## Specification authority

- [spec-react-laravel/](spec-react-laravel/README.md) — English implementation specification and traceability for the required **React + PHP/Laravel** stack.
- [spec-python/](spec-python/) and [spec-.net/](spec-.net/) — original reference baselines retained unmodified for auditability.
- The reference business scope, 10 user stories, acceptance criteria, six business tables, API paths, and business rules are **not expanded or weakened** by changing frameworks.
- The implementation uses PostgreSQL 16 and the `library` schema; deployment infrastructure must never own business DDL.
- Open ambiguities and proposed, testable resolutions are recorded explicitly in [decisions](spec-react-laravel/decisions.md); nothing should be silently invented.

## Repository separation

| Repository | Responsibility |
| --- | --- |
| docs | Reference + React/Laravel adaptation, architectural decisions, acceptance traceability |
| api | Laravel HTTP adapter, application services, pure domain, PostgreSQL migrations, tests |
| app | React authenticated operator UI |
| page | Independent public static presentation; must not call the API |
| infra | Compose, networking, volumes, empty PostgreSQL server; no domain SQL |
| tool | Demo data **via the HTTP API**, never direct SQL |

All new documentation and implementation text is written in English. The five canonical database category values retain their specification-defined spellings for data compatibility.

**Completion means executable evidence**, not the presence of source files alone. Commit and push incremental verifiable results before 11:00.
