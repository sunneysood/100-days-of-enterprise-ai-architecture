# API Architecture Review Checklist

## Contract

- [ ] OpenAPI contract exists
- [ ] Versioning strategy defined
- [ ] Standard error model defined
- [ ] Idempotency considered

## Resource & Interface

- [ ] Resources are modeled consistently
- [ ] Pagination defined where required
- [ ] Filtering/sorting behavior defined
- [ ] Field selection considered

## Security

- [ ] Authentication defined
- [ ] Authorization model defined
- [ ] Transport security enabled
- [ ] Rate limits defined

## Performance

- [ ] Caching strategy assessed
- [ ] Compression assessed
- [ ] Async processing assessed
- [ ] Batch operations assessed
- [ ] Streaming assessed

## Reliability & Observability

- [ ] Timeout policy defined
- [ ] Retry/backoff policy defined
- [ ] Circuit breaking considered
- [ ] Logs defined
- [ ] Metrics defined
- [ ] Correlation/distributed tracing defined

## Developer Experience & Governance

- [ ] Documentation
- [ ] Examples
- [ ] Test/sandbox approach
- [ ] Lifecycle and deprecation policy
- [ ] Ownership defined
