# Roadmap

## Phase 0 — Planning

Start date and status: see [PROGRESS.md](./PROGRESS.md).

---

## 100-Day Engineering Roadmap

### Days 001–010 — Architecture Foundations

- requirements and NFRs
- scalability, availability, latency, cost and security
- capacity thinking
- C4 modeling
- architecture alternatives
- ADRs
- real-world system case studies

### Days 011–035 — Distributed Systems

- networking basics
- serialization
- queues
- Kafka
- partitioning
- replication
- consistency
- idempotency
- retries and timeouts
- circuit breakers
- backpressure
- fault tolerance
- multi-region thinking
- observability and chaos experiments

### Days 036–045 — Data & Messaging

- SQL/PostgreSQL
- transactions and indexes
- ingestion
- ETL/ELT
- data quality
- event-driven integration
- data lineage
- governance

### Days 046–055 — Enterprise RAG

- document ingestion
- chunking
- embeddings
- hybrid retrieval
- metadata filtering
- reranking
- ACL-aware retrieval
- tenant isolation
- citations and provenance
- RAG evaluation

### Days 056–065 — Agents

- stateful workflows
- tools
- planning
- routing
- handoffs
- retries
- checkpoints
- human-in-the-loop
- durable execution
- tool authorization
- agent evaluation
- event-triggered workflows (Kafka)
- approval and remediation loop

### Days 066–072 — Document Intelligence & Data Onboarding

- probabilistic extraction
- deterministic validation
- structured outputs
- confidence and review
- schema validation
- reconciliation
- exception handling

### Days 073–080 — Evaluation & Security

- model vs application evaluation
- offline vs online evaluation
- evaluator pipelines
- RAG evaluation
- regression testing
- security testing
- prompt injection
- data leakage
- scope/authorization failures

### Days 081–090 — LLM Infrastructure

- model selection
- inference fundamentals
- batching
- KV cache
- quantization
- model routing
- vLLM
- GPU utilization
- latency/throughput
- distributed tracing
- Kubernetes
- autoscaling

### Days 091–100 — Enterprise AI Architecture

- business capabilities
- architecture principles
- solution alternatives
- security architecture
- governance
- FinOps
- reliability
- deployment architecture
- production readiness
- final architecture review

## Cross-Cutting Tracks

These threads run through the day blocks above. They do not add days; they set a minimum bar every project must meet.

| Thread | Introduced | Applied in | Minimum evidence per project |
|---|---|---|---|
| API design & contracts                                     | Days 011–035 (P1)          | Every project                                      | OpenAPI contract, versioning note, completed `api-design/api-design-checklist.md` |
| Containers & CI/CD                                         | Days 011–035 (P1 skeleton) | Every project                                      | Dockerfile and a CI run that executes the tests                                   |
| Security & threat modeling                                 | Days 011–035               | Every project; deepened in Days 073–080            | One threat-model entry and one authorization test                                 |
| Observability                                              | Days 011–035 (P1)          | Every project; distributed tracing in Days 081–090 | Structured logs, metrics, and a request-ID or trace path                          |
| Evaluation                                                 | Days 046–055 (P2)          | Every AI project; formalized in Days 073–080       | A small labelled evaluation set that reruns on every change                       |
| Cost / FinOps                                              | Days 046–055               | Every AI project; consolidated in Days 091–100     | A cost-per-request estimate                                                       |
| ADRs                                                       | Days 001–010               | Every project                                      | At least one ADR per project                                                      |
| Infrastructure as code (Terraform, Azure)                  | Days 081–090 (P7)          | P7, P8                                             | Reproducible deploy from code. Docker Compose is enough before P7                 |

## Project-to-Day Mapping

| Project | Built during | Notes |
|---|---|---|
| P1 Production API Foundation     | Days 011–045                                  | Grows block by block: Docker/CI first, PostgreSQL in Days 036–045                                                                   |
| P2 Enterprise Knowledge System   | Days 046–055                                  |                                                                                                                                     |
| P3 Intake-to-Resolution Workflow | Days 056–065                                  | Its workflow core is reused by P6                                                                                                   |
| P4 Document Intelligence         | Days 066–072                                  |                                                                                                                                     |
| P5 Customer Data Onboarding      | Starts Days 036–045, finishes Days 066–072    | Ingestion, validation and reconciliation overlap the Data & Messaging block                                                         |
| P6 Operations AI                 | Days 056–065                                  | Event-triggered extension of P3. Reuses Kafka from Days 011–035. Adds an approval and remediation loop rather than a separate build |
| P7 Production LLM Serving        | Days 081–090                                  |                                                                                                                                     |
| P8 Flagship Platform             | Starts Days 046–055, hardened in Days 091–100 | Merge each project into the platform as it finishes, so Days 091–100 are hardening, not first integration                           |

## Compute and Budget

GPU spending cap: **USD 50 maximum in total** for the whole journey. P7 (Days 081–090) is the main consumer.
The detailed plan (provider, instance type, hours) is a future task, to be done before Day 081.
Fallbacks if the cap is tight: run a small model on CPU, rent a GPU only for benchmark sessions, and stop instances after every session.

---

## Resource Policy

Resources are categorized as:

- **Core** — directly used in the current phase
- **Reference** — consulted when a specific gap appears
- **Optional** — useful but not required

A new course or certification is not added unless it fills a clearly identified capability gap.

---

## Success Criteria

By Day 100, the repository should contain:

- meaningful working code
- architecture diagrams
- ADRs
- API contracts
- security decisions
- evaluation suites
- reliability experiments
- performance measurements
- cost assumptions
- production-readiness thinking
- a clear path into the flagship platform
