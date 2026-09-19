# P1 — Production API Foundation: Requirements

**Days:** 011–045 | **Status:** planned | **Last updated:** 2026-09-19

> Draft. The domain (a multi-tenant case API) and every target below are starting assumptions. Review them at the start of Day 011, then measure a baseline and replace the starter targets with real numbers.

## 1. Problem

Every later project needs the same production baseline: authentication, tenant isolation, persistence with migrations, tests, a container image, CI, and observability. P1 builds that baseline once, around a small multi-tenant case-intake API that P3 (Intake-to-Resolution Workflow) can later drive.

## 2. Users and tenants

Multi-tenant from the first commit. Three roles per tenant: `admin`, `agent` and `viewer`. Service-to-service callers are out of scope for P1.

## 3. Functional requirements

1. The API authenticates requests with a signed JWT. Missing, expired or invalid tokens return `401`.
2. Every request is scoped to the `tenant_id` in the token. A resource that belongs to another tenant is reported as not found.
3. Cases support create, get, list (with pagination and filtering by status), update status, and close.
4. Authorization is role-based: `viewer` reads, `agent` reads and writes cases, `admin` also reads the audit log.
5. Case creation is idempotent when an `Idempotency-Key` header is supplied.
6. Every mutating request writes an audit record: who, tenant, action, resource, time.
7. The API is versioned under `/v1`, publishes an OpenAPI contract, and returns errors in one standard format (problem details).
8. Health and readiness endpoints report process and database status.

## 4. Non-functional requirements

Starter targets. Confirm or replace after the first baseline.

| Attribute | Target | How measured |
|---|---|---|
| Latency (p95) | Reads under 200 ms at 50 requests per second on one container | Load test against Docker Compose |
| Availability | Readiness reflects database health; no availability target for a single local instance | Health endpoint checks |
| Throughput | 50 requests per second sustained for 5 minutes | Load test |
| Cost per request | USD 0 (local); CI stays within the free tier | CI usage |
| Recovery (RTO / RPO) | Database restored from a backup by a documented runbook; RPO equals the last backup | Restore drill |
| Security / tenancy | No cross-tenant read or write in tests; no secrets in the repository; OWASP API Top 10 reviewed | Automated tests, checklist |

## 5. Load and growth assumptions

Assumptions, not measurements: 5 tenants, about 1,000 cases per tenant, 50 requests per second at peak. For design discussion, assume 10x growth in tenants and cases, which stresses indexing on `tenant_id` and pagination.

## 6. Out of scope

User interface, external identity provider integration (tokens are issued locally for tests), event streaming, rate limiting (deferred to the gateway in P8), multi-region deployment.

## 7. Failure modes

| What can fail | Expected behaviour |
|---|---|
| Database unavailable | Readiness fails; requests return `503` in the standard error format; no partial writes |
| Expired or forged token | `401`; nothing is logged with the token value |
| Client retries a create after a timeout | Same result returned; only one case created |
| Migration fails during deploy | Deploy halts; the previous version keeps serving |
| Request for another tenant's case | Not found; the attempt is written to the audit log |

## 8. Security and tenancy

Token claims carry `sub`, `tenant_id` and `roles`. Tenant scoping is enforced in one data-access layer and covered by tests, not repeated in each handler. A threat-model entry for P1 will be written before Day 045.

## 9. Evaluation

P1 is not an AI project, so evaluation means an automated test suite (unit, integration and contract tests) plus a recorded load-test baseline. Both run in CI.

## 10. Cost assumptions

Local Docker Compose and free-tier CI. No paid cloud resources.

## 11. Definition of done

- [ ] All functional requirements have a passing test
- [ ] Every NFR has a measured value
- [ ] OpenAPI contract and API checklist completed
- [ ] Dockerfile and a CI run executing the tests
- [ ] Threat-model entry and an authorization test
- [ ] Evaluation set runs and meets its thresholds
- [ ] At least one ADR
- [ ] README with how to run it and what was measured

## 12. Key decisions

Candidate ADRs to write during P1:

- Tenant isolation strategy (shared tables with a `tenant_id` filter, row-level security, or schema per tenant)
- Token format and signing approach
- Database migration tool
- Error format
