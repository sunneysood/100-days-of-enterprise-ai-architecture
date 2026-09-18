# API Design

Production API design is broader than endpoint naming and HTTP verbs.

## Six-Pillar Review

### 1. API Contracts

- OpenAPI
- versioning
- standard error formats
- idempotency

### 2. Resource & Interface Design

- resource modeling
- hierarchy
- pagination
- filtering
- field selection

### 3. Security by Design

- authentication
- authorization
- TLS/mTLS
- API keys where appropriate
- rate limiting and throttling

### 4. Performance & Scalability

- caching
- compression
- asynchronous APIs
- batching
- streaming

### 5. Reliability & Observability

- retries and backoff
- timeouts
- circuit breakers
- logging
- metrics
- tracing/correlation

### 6. Developer Experience & Governance

- documentation
- examples
- SDKs/tools
- sandbox/testing
- lifecycle management
- deprecation

## Architecture Rule

An API is not production-ready because its endpoints are well named.

Review the full lifecycle: **contract → security → performance → reliability → observability → adoption/governance**.
