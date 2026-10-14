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

## Published but NOT yet acceptance-verified

- Full architecture/ports/adapters independence (T-07…T-11, T-19): the current Laravel controller retains orchestration and persistence responsibilities. Formal hexagonal composition and application fake tests remain missing.
- File upload content/size/binary lifecycle, soft deletion with active loan conflict, every error code and complete authorization matrix.
- Full UI acceptance beyond the successful book/member/loan/return happy path: negative cases, authorization by role, pagination, cover upload, report, overdue screen and responsive checks (T-22).
- Public page responsive/browser checks (T-23); the source is intentionally self-contained.
- Tool execution against a live populated API (T-21). The documented three-overdue-item requirement remains impossible with the specified clock API.
- Performance with 5,000 books (RNF-1), all edge and negative cases, and nonfunctional security review.

## Important cautions

The original Python and .NET source specifications remain untouched. The adaptation's technical deviations and two unresolved issues are in [decisions.md](decisions.md). No unchecked row should be marked as passing, and a successful basic smoke test does not certify the full product.
