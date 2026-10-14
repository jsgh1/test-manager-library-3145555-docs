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

## Verified security, application and report port adaptations

- [API authentication refactor f72fd5e](https://github.com/jsgh1/test-manager-library-3145555-api/commit/f72fd5e14fee63708509011bd2040252d8adbd1f) introduced application-owned user, password and token ports, adapters, `LoginService` and `CreateLibrarianService`; the HTTP controller now delegates these use cases.
- [API authentication fake-port tests 137501d](https://github.com/jsgh1/test-manager-library-3145555-api/commit/137501d612acfe95a1b0eb3b615a618ca5622caf) verify username normalization, secrecy of hashes in responses, denial of admin self-provisioning, valid login and undifferentiated invalid credential errors.
- [API integration run 38589211868](https://github.com/jsgh1/test-manager-library-3145555-api/actions/runs/38589211868) verified API librarian creation, librarian read access, 403 on admin-only routes, duplicate ISBN and member 409, and open-loan withdrawal 409, plus prior concurrency checks.
- [API report port refactor c89cda1](https://github.com/jsgh1/test-manager-library-3145555-api/commit/c89cda1225055cac0987e0aa7186e6dd5453db14) and [report fake tests 6727c23](https://github.com/jsgh1/test-manager-library-3145555-api/commit/6727c23ab7a97d6a181426084bb8185505931218) preserve historical snapshots and inclusive UTC date ranges.
- **All current API tests** (PHP syntax, unit/fake-port architecture checks and PostgreSQL integration) passed in [run 38590276701](https://github.com/jsgh1/test-manager-library-3145555-api/actions/runs/38590276701).
- The independent public `page` completed [static checks](https://github.com/jsgh1/test-manager-library-3145555-page/actions/runs/38589644644); the `tool` Node + PowerShell entrypoints completed [syntax checks](https://github.com/jsgh1/test-manager-library-3145555-tool/actions/runs/38589641697).
- [Tool's PowerShell Docker launcher](https://github.com/jsgh1/test-manager-library-3145555-tool/blob/main/Iniciar-Semillero.ps1) is published, **but an actual end-to-end demonstration seed run has not yet been evidenced**.

## Verified catalogue application-service refactor

- [API catalogue refactor 8425135](https://github.com/jsgh1/test-manager-library-3145555-api/commit/8425135c7e1101d845731620b3cd2cf84772419f) introduced three focused application services: `CreateBookService`, `UpdateBookService`, `WithdrawBookService`, backed by application-owned `BookCatalog` and a PostgreSQL adapter.
- [Fake catalogue acceptance tests c62dce2](https://github.com/jsgh1/test-manager-library-3145555-api/commit/c62dce2ab5e13efde48796a3c1510d1cf91c8225) verify copy availability and trim on creation, duplicate ISBN rejection, stock-safe total adjustments, and refusal to withdraw a book with open loans.
- **PHP and live PostgreSQL CI jobs both passed**: [API workflow 38590795028](https://github.com/jsgh1/test-manager-library-3145555-api/actions/runs/38590795028).
- Reading/listing, membership and cover orchestration are still implemented in the controller and remain pending for **full** hexagonal architecture acceptance.

## Manual librarian-role acceptance checks (learner screenshots)

**October 14, 2026, approximately 10:38 America/Bogota.** Six screenshots provided by the learner from the running Docker/React/Laravel/PostgreSQL application establish the following browser observations:

| Test | Observed result | Outcome |
| --- | --- | --- |
| Admin creates a librarian | Administrator-only *Librarians* screen displays `Librarian created.` | **PASS (browser)** |
| Login under librarian role | `Librarian` account label visible after a new login | **PASS (browser)** |
| UI role separation | Sidebar includes Overview, Catalogue, Members, Loans & returns, Overdue items; Reports and Librarians admin links are absent; Catalogue has no Edit/Withdraw actions | **PASS (browser, UI only)** |
| Librarian catalogue search | Searching for `Cien años` returns the existing title; stock observed at `2/3` | **PASS (browser)** |
| Librarian registers a member | Green `Saved successfully.` banner and new `Usuario Stage` member, document `QA20261015`, present in listing | **PASS (browser)** |
| Librarian issues loan | Green `Loan registered.` banner and newly displayed `Open` checkout | **PASS (browser)** |
| Inventory decreases correctly | Catalogue inventory changes from `2/3` to `1/3` following checkout | **PASS (browser)** |

**Authorization distinction:** the screenshots demonstrate **UI visibility** restrictions, not direct backend rejection; the latter is separately demonstrated by the 403 assertions in [API CI run 38589211868](https://github.com/jsgh1/test-manager-library-3145555-api/actions/runs/38589211868).

**Remaining:** full manual testing of librarian returns, overdue-report handling, responsive behavior and role-specific HTTP error cases, and demonstration data generator execution.

## Published but NOT yet acceptance-verified

- Complete hexagonal separation (T-08/T-19): lending, returns, authentication, historical reporting and book create/update/withdraw now use application services and tested ports/adapters. Book searching/detail, cover operations, member management and loan/overdue listings still retain infrastructure orchestration in the HTTP controller.
- File upload content/size/binary lifecycle, soft deletion with active loan conflict, every error code and complete authorization matrix.
- Full UI acceptance beyond the confirmed admin book/member/checkout/return and librarian creation/login/search/member/checkout paths: pagination, cover upload, report, overdue screen, role-specific HTTP cases and responsive checks (T-22).
- Public page responsive/browser checks (T-23); the source is intentionally self-contained.
- Tool execution against a live populated API (T-21). The documented three-overdue-item requirement remains impossible with the specified clock API.
- Performance with 5,000 books (RNF-1), all edge and negative cases, and nonfunctional security review.

## Important cautions

The original Python and .NET source specifications remain untouched. The adaptation's technical deviations and two unresolved issues are in [decisions.md](decisions.md). No unchecked row should be marked as passing, and a successful basic smoke test does not certify the full product.
