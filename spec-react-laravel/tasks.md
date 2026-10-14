# Task execution and acceptance traceability — React + Laravel

Keep the reference task numbers, user stories and evidence unchanged, replacing only framework-specific tools and module names.

| Task | Deliverable | Verifiable evidence |
| --- | --- | --- |
| T-01 | Laravel skeleton, health endpoint | GET `/health` yields `{"status":"ok"}` and boots in container |
| T-02 | Six PostgreSQL `library` tables and fixed seed | `information_schema`, FK/index introspection, no DEFAULT columns |
| T-03 | PHP pure value objects `Isbn`, `Quantity`, `DateRange` | PHPUnit boundary tests |
| T-04 | Pure `Book` domain | Cannot withdraw below zero or restock beyond total |
| T-05 | Pure `Loan` domain | Frozen snapshot; 14-day deadline; state with fake clock |
| T-06 | Pure member/user validation | Normalized username, librarian-only API role |
| T-07 | Login / create-librarian application ports + services | Fake-port tests HU-01, HU-02 |
| T-08 | Books and cover use cases | Fake-port tests HU-03, HU-04 |
| T-09 | Lending use case | Fake-port tests limits, overdue, availability |
| T-10 | Return / overdue use cases | Duplicate return 409 and no double increment |
| T-11 | Snapshot report reader | Rename does not rewrite historical report |
| T-12 | PostgreSQL adapters (books/members/users) | Real DB integration |
| T-13 | Loan persistence and concurrency | Simultaneous last-copy checkout: one 201, one 409 |
| T-14 | Hash/JWT adapters | Never emit plaintext/hash; expiry is 28800 seconds |
| T-15 | Cover storage adapter | 2MB valid image accepted; 3MB rejected |
| T-16 | Versioned API and uniform errors | HTTP contract covers all endpoints and status codes |
| T-17 | Composition and bootstrapped admin | Empty database migration, admin login |
| T-18 | `ck_book_total_positive`, `ck_user_role` | Invalid manual INSERT rejected; reflect actual DDL here |
| T-19 | Domain/application dependency architecture | Automated forbidden-reference checks |
| T-20 | Compose/Postgres/Laravel/React | One command starts empty database engine + apps |
| T-21 | API-only demo tool | At least 40 books, 10 members, 15 loans; overdue exception D-01 |
| T-22 | React operational workflows | Auth, catalogue, members, lending, returns, overdues, admin report |
| T-23 | Standalone public page | Works responsive without backend access |

**Progress status at specification commit:** all implementation tasks are still pending until executable evidence exists.
