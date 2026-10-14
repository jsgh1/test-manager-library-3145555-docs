# Verified evidence and acceptance status

**Evidence snapshot:** October 14, 2026. This log is deliberately conservative: a published implementation file is not proof that all requirements work.

## Verified via GitHub Actions

| Area | Evidence | Result |
| --- | --- | --- |
| PHP dependencies, syntax lint, pure-domain PHPUnit tests | [API workflow run 38572889707](https://github.com/jsgh1/test-manager-library-3145555-api/actions/runs/38572889707) (`php` job) | PASS |
| PostgreSQL 16 migrations and bootstrap admin | [API workflow run 38572889707](https://github.com/jsgh1/test-manager-library-3145555-api/actions/runs/38572889707) (`integration` job) | PASS |
| JWT login, five categories, create book/member, issue and return loan, computed closed status | Same API integration workflow | PASS |
| No-token 401 and repeated return 409 | Same API integration workflow | PASS |
| Historical book title remains frozen after rename | Same API integration workflow | PASS |
| Concurrent last-copy loans result in exactly one 201 and one 409 | Same API integration workflow | PASS |
| React/Vite production build | [React workflow run 38572062037](https://github.com/jsgh1/test-manager-library-3145555-app/actions/runs/38572062037) | PASS |

## Local Docker deployment evidence (learner-provided)

- **October 14, 2026, approximately 09:54 America/Bogota:** The learner executed the PowerShell setup from `C:\\Users\\Aprendiz\\Desktop\\ManagerLibrary` and supplied the resulting terminal log.
- `docker compose up --build -d` successfully built `manager-library-api` and `manager-library-app`, pulled `postgres:16-alpine`, created the network and persistent volumes, and started all three services.
- `docker compose ps` reported `manager-library-db-1` **healthy**, `manager-library-api-1` **up**, and `manager-library-app-1` **up**.
- Health check `GET http://localhost:8000/health` returned `{"status":"ok"}`.
- The learner independently opened `http://localhost:5173/`, logged in successfully, and reached the dashboard.
- **Conclusion:** T-20's one-command Docker bootstrap and the entry-level T-17 admin login were verified in the learner's local environment. This does **not** yet certify all end-to-end user-story acceptance criteria.
- **Remediated:** The initial Composer PSR-4 test-autoload warning was corrected in API commit [`5e87949`](https://github.com/jsgh1/test-manager-library-3145555-api/commit/5e87949d633bb3ca00aa63dab0bc900fb520765a); both PHP unit and PostgreSQL integration CI jobs passed in [run 38585063252](https://github.com/jsgh1/test-manager-library-3145555-api/actions/runs/38585063252).

## Verified manual operator workflow (learner screenshots)

**October 14, 2026, approximately 10:06 America/Bogota.** The learner supplied six screenshots from a working React browser session backed by the locally running Docker API and PostgreSQL database. This is manual visual acceptance evidence for the happy path; it does not replace automated negative tests.

| Step | Observed browser evidence | Result |
| --- | --- | --- |
| Create book (HU-03) | `Cien años de soledad`, author `Gabriel García Márquez`, ISBN `9780307474728`, `Novela`, available `3/3` | PASS |
| Create member (HU-05) | `Usuario QA`, document `QA20261014`, record visible in Members table | PASS |
| Issue one-line loan (HU-06) | Green `Loan registered.` banner; loan displays `Open`; catalogue stock changes `3/3 -> 2/3` | PASS |
| Return the item (HU-07) | Green `Return recorded.` banner; item displays `Returned`; loan displays `Closed` | PASS |
| Inventory consistency (HU-03/HU-07) | Catalogue stock restored to `3/3` | PASS |

**Scope of evidence:** confirms the complete primary browser checkout-and-return route, not the full error matrix for HU-03/HU-05/HU-06/HU-07/HU-08. Captures were inspected in the learner conversation; the GitHub repository records the observations but does not itself store the image binaries.

**UI follow-up:** the helper paragraph below the loan form visibly overlaps its grid controls. A cosmetic fix should remove or reposition the obsolete instruction.

## Verified manual negative scenarios (learner screenshots)

**October 14, 2026, approximately 10:15 America/Bogota.** Three additional screenshots captured from the locally running React/Laravel/PostgreSQL stack:

| Acceptance criterion | Input and observed result | Status |
| --- | --- | --- |
| CA-03.2, duplicate ISBN | Attempting to create another active book with ISBN `9780307474728` displayed `ISBN already exists.`; catalogue still showed exactly one matching book | **PASS (browser)** |
| CA-05.1, duplicate member document | Attempting to register another member with `QA20261014` displayed `Document number already exists.`; member list still showed one matching record | **PASS (browser)** |
| CA-03.5, withdraw with open loan | A book on loan (stock `2/3`) remained in active catalogue after `Withdraw` and displayed `Book has open loans.` | **PASS (browser)** |

These results verify individual negative acceptance paths, not the complete response schema or all authorization, input validation and concurrency scenarios.

## Verified application-layer separation (partial architectural remediation)

- **API refactor:** [commit 048b850](https://github.com/jsgh1/test-manager-library-3145555-api/commit/048b8503562e06060cda9ef9dee48b594333151f) moved transactional lending and item returns into `LendBooksService` / `ReturnItemService` using application-owned `LendingPort`, `Clock`, and `IdGenerator` interfaces.
- PostgreSQL persistence, system time and UUID generation now live behind adapters wired by `AppServiceProvider` (the composition root); the HTTP controller delegates checkout/return orchestration.
- **Tests:** [commit dd5e946](https://github.com/jsgh1/test-manager-library-3145555-api/commit/dd5e9468346fb017b7b09435bfea194682d93a3d) added isolated fake-port tests for duplicate books, maximum checked-out books, overdue prohibition, atomic rollback, snapshot dates, and repeat returns, plus dependency-boundary checks.
- **Evidence:** [API CI run 38588634252](https://github.com/jsgh1/test-manager-library-3145555-api/actions/runs/38588634252) completed successfully in both PHP and PostgreSQL integration jobs, including checkout/return, historical snapshot and simultaneous last-copy loan protection.
- **Honest limitation:** authentication, catalogue, membership, overdue listing and report still have application logic within the Laravel HTTP controller. Therefore the system is **not yet a fully hexagonal implementation** and T-07/T-08/T-11/T-19 cannot be marked completely satisfied solely by this partial remediation.

## Published but NOT yet acceptance-verified

- Complete hexagonal separation (T-07/T-08/T-11/T-19): lending and returning now have tested ports/adapters, but authentication, books, membership, listing and reporting still need application service extraction and fuller architecture checks.
- File upload content/size/binary lifecycle, soft deletion with active loan conflict, every error code and complete authorization matrix.
- Full UI acceptance beyond the successful book/member/loan/return happy path: negative cases, authorization by role, pagination, cover upload, report, overdue screen and responsive checks (T-22).
- Public page responsive/browser checks (T-23); the source is intentionally self-contained.
- Tool execution against a live populated API (T-21). The documented three-overdue-item requirement remains impossible with the specified clock API.
- Performance with 5,000 books (RNF-1), all edge and negative cases, and nonfunctional security review.

## Important cautions

The original Python and .NET source specifications remain untouched. The adaptation's technical deviations and two unresolved issues are in [decisions.md](decisions.md). No unchecked row should be marked as passing, and a successful basic smoke test does not certify the full product.
