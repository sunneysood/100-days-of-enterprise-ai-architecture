# 100 Days of Enterprise AI Architecture

> A hands-on journey from system design to production-grade Enterprise AI architecture.

This repository is an engineering apprenticeship, not a course-completion challenge.

The objective is to develop the ability to **understand, design, build, break, measure, secure, and explain** production AI systems.

## North Star

By the end of the journey, the repository should demonstrate a coherent capability stack:

**System Design → Distributed Systems → API Design → Data → AI Systems → RAG → Agents → Evaluation → LLM Infrastructure → Cloud/DevOps → Security/Governance → Enterprise Architecture**

All of these converge into **one flagship Enterprise AI platform**.

## Learning Philosophy

Every meaningful topic follows:

**Learn → Build → Break → Measure → Decide → Document**

The repository values:

- architectural reasoning over memorization
- trade-offs over diagrams
- working implementations over tutorial completion
- production failure modes over happy-path demos
- evidence over certificates

## Repository Map

```
architecture/              Architecture artifacts and decisions
system-design/             Distributed systems and real-world system studies
api-design/                Production API architecture
data/                      Data engineering and governance
ai/                        LLMs, RAG, agents, evaluation, inference
distributed-systems-lab/   Hands-on distributed systems implementation
projects/                  Progressive capability slices
experiments/               Focused technical experiments
days/                      Day-by-day engineering journal
docs/                      Durable technical notes
infrastructure/            Docker, Kubernetes, Terraform, Azure
.github/                   CI, security and evaluation automation
```

## The 100-Day Structure

| Days | Focus |
|---|---|
| 001–010 | Architecture foundations & system design |
| 011–035 | Distributed systems |
| 036–045 | Data & messaging |
| 046–055 | Enterprise RAG |
| 056–065 | Agents & stateful workflows |
| 066–072 | Document intelligence & data pipelines |
| 073–080 | AI evaluation & security |
| 081–090 | LLM infrastructure & serving |
| 091–100 | Enterprise AI architecture synthesis |

The sequence is intentionally progressive. Earlier capabilities become building blocks for later ones.

## Core Engineering Questions

For every system, ask:

1. What problem are we solving?
2. What are the functional requirements?
3. What are the quality attributes and NFRs?
4. What are the expected load and growth assumptions?
5. What can fail?
6. What should be synchronous vs asynchronous?
7. Where does state live?
8. How do we secure identities, data, tools and tenants?
9. How do we observe and evaluate the system?
10. What does it cost?
11. How does it recover?
12. Why is this architecture preferable to the alternatives?

## Flagship Platform

The journey ultimately converges into a production-oriented Enterprise AI platform containing:

- multi-tenant identity and authorization
- API and AI gateway
- enterprise knowledge/RAG
- stateful agent workflows
- document intelligence
- customer/data onboarding
- operational AI workflows
- model routing and LLM serving
- evaluation and regression testing
- observability and tracing
- security and governance
- FinOps and cost controls
- resilience, scaling and disaster recovery

The flagship is the **proof of integration**, not another unrelated project.

## Current Status

**Phase 0 — PgMP preparation**

The engineering journey begins after the PgMP milestone. The repository can be prepared beforehand; the 100-day execution starts afterward.

## Guiding Principle

> **Do not ask: “What course should I take next?”  
> Ask: “What architectural problem can I solve now that I could not solve before?”**
