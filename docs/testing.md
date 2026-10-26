# Testing & Verification Strategy — CPAE

## 1. Testing Philosophy

Regressions in access control, agenda and financial logic have direct operational and regulatory impact, so testing prioritizes domain correctness and authorization boundaries.

## 2. Test Pyramid & Tooling

Counts below were measured on a recent snapshot of the codebase by counting test files and `it`/`test` declarations. They are approximate and change over time.

| Test Type | Scope | Framework / Tools | Size (approx.) | Focus |
|---|---|---|---|---|
| **Unit tests** | Services, guards, Luna components | Jest, ts-jest | 80 spec files, ~750 cases | RBAC, agenda transfer, financial rules, Luna routing/grounding/isolation, golden tests |
| **End-to-end API tests** | Controllers and ORM layer | Jest, Supertest | 8 e2e suites, ~465 cases | BOLA/RBAC, agenda, financial, Luna |
| **Browser tests** | Frontend flows | Playwright | 3 spec files, 7 tests | Financial, agenda transfer (including FORCE visibility by role), visual checks |
| **Schedule verification** | Operational integrity | Node.js audit scripts | n/a | Schedule and pending-evolution audits |

Testcontainers is not used.

## 3. Operational Integrity Scripts

- `schedule:weekly:audit`: scans weekly schedule data for inconsistencies.
- `audit:agenda-production`: audits agenda data against production rules.
- `clinical-records:pending:audit`: lists pending evolutions that still need completion.

## 4. Luna Reliability

Luna is covered by golden routing tests, adversarial and empirical specs, grounding and sandbox hardening specs, isolation/RBAC specs and opt-in specs. New assistant behavior is expected to ship with a golden test.
