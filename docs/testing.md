# Testing & Verification Strategy — CPAE

## 1. Testing Philosophy

In healthcare software, regressions in access control or scheduling logic can cause direct operational disruption and regulatory penalties. Testing prioritizes **domain logic correctness** and **boundary security**.

## 2. Test Pyramid & Tooling

| Test Type | Scope | Framework / Tools | Focus |
|---|---|---|---|
| **Unit Tests** | Domain Services & Pipes | Jest, ts-jest | RBAC permission evaluation, billing rule engines |
| **Integration Tests** | Controllers & ORM Layer | Supertest, Testcontainers | DB transactions, tenant isolation middleware |
| **Schedule Verification** | Operational Integrity | Custom Node.js audit scripts | Weekly schedule collision detection, vacancy auditing |

## 3. Operational Integrity Scripts

To ensure appointment consistency and database sanity during production updates, automated verification scripts were built into the deployment pipeline:
- `schedule:weekly:audit` — Scans upcoming weekly slots for duplicate bookings or room overlaps.
- `audit:agenda-production` — Cross-references practitioner availability matrices with active patient recurrences.
- `clinical-records:pending:audit` — Identifies draft evolutions requiring practitioner signature beyond the 48-hour compliance window.
