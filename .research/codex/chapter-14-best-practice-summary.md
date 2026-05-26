# Chapter 14. Best Practice Summary

This chapter condenses the research into practical recommendations by system size and operational pressure.

## Small API

- use `net/http` or Chi;
- use simple project layout and explicit handlers;
- use PostgreSQL or SQLite depending on scope;
- keep logging structured but minimal;
- use Docker Compose for local orchestration;
- keep auth simple unless external identity is required.

## Medium API

- use Chi or Echo;
- add validation, structured logging, metrics, and typed SQL access;
- use PostgreSQL, Redis, and migrations;
- add integration tests and CI checks;
- define versioning and error schema early;
- add refresh-token or session auth with clear policy.

## Enterprise API

- use feature-based clean or hexagonal architecture;
- keep domain logic isolated from transport and storage;
- add tracing, queue integration, policy control, and stronger secret management;
- use Kubernetes, rollout strategy, and managed services;
- enforce code quality and security gates in CI;
- standardize contract documentation and deprecation policy.

## Microservice

- prefer small surface area and explicit contracts;
- keep outbound calls resilient;
- use circuit breakers, metrics, traces, and idempotency;
- avoid overcoupling to shared libraries.

## High-Load API

- measure before optimizing;
- use bounded concurrency and connection pooling;
- optimize database queries and caching before framework switching;
- keep request paths short and predictable.

## Source Notes

- Internal references: `.research/antigravity/golang-api-research-plan.md`, `docs/go-api-guide/*`.
- Research anchors: framework docs, database docs, security references, observability references, deployment references.

## Summary

The right stack depends on scale. The default recommendation is minimal tooling first, then add complexity only when the measured need is real.
