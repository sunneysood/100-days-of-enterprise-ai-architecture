# <P#> — <Project name>: Requirements

**Days:** <range> | **Status:** planned / in progress / done | **Last updated:** <date>

## 1. Problem

One or two sentences. Who has the problem and what does it cost them?

## 2. Users and tenants

Who uses it? Single tenant or multi-tenant? Which roles?

## 3. Functional requirements

Numbered, testable, 3–7 items. "The system shall ..."

## 4. Non-functional requirements

| Attribute | Target | How measured |
|---|---|---|
| Latency (p95)                  |   |   |
| Availability                   |   |   |
| Throughput                     |   |   |
| Cost per request               |   |   |
| Recovery (RTO / RPO)           |   |   |
| Security / tenancy             |   |   |

## 5. Load and growth assumptions

Users, requests per day, data volume, expected growth. State that these are assumptions.

## 6. Out of scope

What this project deliberately does not do.

## 7. Failure modes

| What can fail | Expected behaviour |
|---|---|

## 8. Security and tenancy

Identities, authorization model, data boundaries, link to the threat-model entry.

## 9. Evaluation

Dataset, metrics, pass thresholds, how it reruns on every change.

## 10. Cost assumptions

Main cost drivers and the estimate per request or per day.

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

Links to ADRs.