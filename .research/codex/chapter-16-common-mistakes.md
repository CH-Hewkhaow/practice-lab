# Chapter 16. Common Mistakes

These are the recurring failure modes in Go API projects. They are usually avoidable.

## API and HTTP Mistakes

- relying on default timeouts;
- returning inconsistent response shapes;
- mixing transport logic with business logic;
- using vague status codes or hiding errors behind generic 500s.

## Concurrency Mistakes

- starting goroutines without a shutdown path;
- sharing mutable state without synchronization;
- ignoring context cancellation;
- creating unbounded worker fan-out.

## Data Mistakes

- querying without indexes;
- triggering N+1 database access patterns;
- using global DB state without lifecycle control;
- hiding SQL behavior inside overly complex abstraction layers.

## Security Mistakes

- logging secrets or tokens;
- accepting unvalidated input;
- using weak password storage;
- leaving CORS open by default;
- assuming JWT alone solves authorization.

## Delivery Mistakes

- skipping integration tests;
- deploying without rollback planning;
- ignoring observability until after the first incident;
- treating Docker and CI as an afterthought.

## Source Notes

- Internal references: `docs/go-api-guide/04-stdlib-and-best-practices.md`, `05-production-setup-and-integration.md`, `07-authentication-jwt-access-refresh.md`, `docs/goroutines-research.md`.
- Official anchors: Go memory model, `context`, `net/http`, OWASP API security, database docs.

## Summary

Most production defects in Go APIs are not language defects. They are lifecycle, boundary, and operational discipline failures.
