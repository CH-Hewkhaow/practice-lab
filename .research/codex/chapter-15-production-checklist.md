# Chapter 15. Production Checklist

This checklist is the release gate for a Go API. If a section is incomplete, the service should not be treated as production-ready.

## Security

- [ ] CORS policy is explicit.
- [ ] TLS is enabled or terminated safely.
- [ ] Rate limits exist for public endpoints.
- [ ] Input validation covers request payloads.
- [ ] Secrets are not logged.
- [ ] Dependencies are scanned for vulnerabilities.

## Testing

- [ ] Unit tests cover core business logic.
- [ ] Integration tests cover persistence and external boundaries.
- [ ] Critical routes have end-to-end coverage.
- [ ] Table-driven tests are used where scenarios branch.
- [ ] Race testing is part of CI for concurrency-sensitive code.

## Performance

- [ ] HTTP timeouts are configured.
- [ ] DB pool settings are explicit.
- [ ] Hot paths are benchmarked when needed.
- [ ] Cache behavior is defined and measurable.
- [ ] Expensive allocations have been reviewed.

## Observability

- [ ] Structured logs include request/correlation ID.
- [ ] Metrics expose latency, errors, and saturation.
- [ ] Tracing works across service boundaries.
- [ ] Health, readiness, and liveness endpoints exist.
- [ ] Alerting criteria are defined.

## Deployment

- [ ] Build is reproducible in CI.
- [ ] Container image is minimal and reviewed.
- [ ] Rollback path is documented.
- [ ] Environment promotion is explicit.
- [ ] Backup and recovery expectations are known.

## Database

- [ ] Migrations are versioned.
- [ ] Transactions are used where atomicity matters.
- [ ] Indexes support real query patterns.
- [ ] Connection limits are configured.
- [ ] N+1 patterns were checked.

## API Design

- [ ] Resource naming is consistent.
- [ ] Error schema is stable.
- [ ] Pagination and filtering are standardized.
- [ ] Versioning strategy is documented.
- [ ] Backward compatibility and deprecation rules exist.

## Documentation

- [ ] OpenAPI or equivalent API docs exist.
- [ ] Runbooks and operational notes exist.
- [ ] Auth and integration assumptions are documented.

## Incident Response

- [ ] On-call or escalation path is known.
- [ ] Logs and traces are sufficient for diagnosis.
- [ ] Rollback and recovery steps are tested.

## Summary

Use this checklist before production promotion and after major change sets. It prevents the common mistake of treating a working service as an operationally safe one.
