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

## Published but NOT yet acceptance-verified

- Full architecture/ports/adapters independence (T-07…T-11, T-19): the current Laravel controller retains orchestration and persistence responsibilities. Formal hexagonal composition and application fake tests remain missing.
- File upload content/size/binary lifecycle, soft deletion with active loan conflict, every error code and complete authorization matrix.
- Full Docker Compose one-command deployment (T-20) on a real Docker host; the component CI runs do not prove the Compose images start together.
- Full operator UI manual journey, responsive checks and populated loan picker beyond the build test (T-22).
- Public page responsive/browser checks (T-23); the source is intentionally self-contained.
- Tool execution against a live populated API (T-21). The documented three-overdue-item requirement remains impossible with the specified clock API.
- Performance with 5,000 books (RNF-1), all edge and negative cases, and nonfunctional security review.

## Important cautions

The original Python and .NET source specifications remain untouched. The adaptation's technical deviations and two unresolved issues are in [decisions.md](decisions.md). No unchecked row should be marked as passing, and a successful basic smoke test does not certify the full product.
