# Projects

These are progressive capability slices that eventually feed the flagship platform.

## P1 — Production API Foundation

FastAPI • PostgreSQL • auth • testing • Docker • CI/CD • observability

## P2 — Enterprise Knowledge System

Identity • tenants • ACL-aware retrieval • hybrid search • reranking • citations • audit

## P3 — Intake-to-Resolution Workflow

Stateful workflow • tools • checkpoints • retries • HITL • recovery

## P4 — Document Intelligence

Probabilistic extraction • deterministic validation • confidence • review

## P5 — Customer Data Onboarding

Ingestion • validation • transformation • reconciliation • exception handling

## P6 — Operations AI

Event-triggered extension of P3 (Days 056–065); reuses Kafka from Days 011–035.

Events • analysis • decision • approval • remediation • incident loop

## P7 — Production LLM Serving

vLLM • gateway • routing • Kubernetes • load testing • tracing • autoscaling • cost

## P8 — Flagship Enterprise AI Platform

Integrate the capabilities above and harden the platform for security, reliability, observability, evaluation, FinOps and DR.

## Starter Success Metrics

Measure a baseline first, then set the target. Replace the wording below with numbers in each project's requirements file (copy `_TEMPLATE.md` to `pN-requirements.md`).

| Project Headline success metric  |                                                                                                           |
| -------------------------------- | --------------------------------------------------------------------------------------------------------- |
| P1                               | p95 latency and error rate under a load test; CI runs tests on every commit; auth enforced on every route |
| P2                               | recall\@k and citation accuracy on a labelled set; **zero** cross-tenant leakage in tests                 |
| P3                               | A workflow killed mid-run resumes from its checkpoint with no duplicated side effects                     |
| P4                               | Field-level accuracy; share auto-approved vs sent to review; false-accept rate on validated fields        |
| P5                               | Every input record is accounted for as accepted, rejected or exception; zero silent drops                 |
| P6                               | Every remediation action requires approval; simulated incident loop completes end to end                  |
| P7                               | Time to first token, tokens per second and cost per 1k tokens under load; autoscaling behaviour observed  |
| P8                               | End-to-end SLOs met; DR drill meets RTO/RPO; security tests pass; evaluation regression gate in CI        |
