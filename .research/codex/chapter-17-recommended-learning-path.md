# Chapter 17. Recommended Learning Path

This path is ordered to reduce rework. Each step adds one layer of capability without skipping fundamentals.

## Phase 1: Go Foundations

- syntax, packages, modules, pointers, structs, interfaces;
- errors, doc comments, and package boundaries;
- `context`, concurrency, and the memory model.

## Phase 2: HTTP and API Basics

- `net/http`, middleware, handlers, JSON encoding/decoding;
- status codes, headers, request lifecycle, validation;
- route design, versioning, and error schema.

## Phase 3: Persistence

- `database/sql`, `pgx`, transactions, migrations;
- query patterns, indexing, and repository boundaries;
- SQLC or an ORM selection based on project needs.

## Phase 4: Production Hardening

- structured logging, metrics, tracing;
- auth, security, rate limiting, secret handling;
- testing, CI, and deployment basics.

## Phase 5: Advanced Systems

- queues, webhooks, retries, circuit breakers;
- performance profiling and caching;
- Kubernetes, cloud services, and operational readiness.

## Suggested Reading Order

1. Effective Go
2. Go docs and `net/http`
3. Go `context` and memory model
4. API and HTTP references
5. Framework docs
6. Database and migration docs
7. Security and observability docs
8. Testing and deployment docs

## Summary

The recommended path is foundation first, then delivery, then data, then production systems. Skipping that order usually creates avoidable design churn.
