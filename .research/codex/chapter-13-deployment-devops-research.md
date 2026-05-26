# Chapter 13. Deployment & DevOps Research

Deployment quality determines how safely the API reaches users and how quickly it can be recovered when something fails.

## Deployment Layers

### Containers

- use multi-stage Docker builds;
- prefer minimal runtime images;
- keep build-only tooling out of production images.

### Orchestration

- Docker Compose is appropriate for local and lightweight staging setups;
- Kubernetes is appropriate when scaling, rollout control, and service orchestration matter;
- Helm helps package and standardize Kubernetes deployments.

### CI/CD

- run formatting, linting, tests, and vulnerability checks in pipeline;
- include build artifacts and deployment promotion gates;
- make rollback paths explicit.

## Runtime Concerns

- inject configuration through environment variables;
- ensure log shipping and alerting are part of runtime design;
- protect startup and shutdown sequences with timeouts;
- define backup and disaster recovery expectations.

## Deployment Strategies

- blue-green for low-risk cutover;
- canary for gradual exposure;
- rollback as a first-class operation, not a manual emergency task.

## Cloud Integration

- managed databases reduce operational burden;
- managed secrets services reduce secret exposure risk;
- load balancers and API gateways control ingress policy;
- cloud runtime products should match the service's scaling and packaging model.

## Source Notes

- Internal references: `docs/go-api-guide/05-production-setup-and-integration.md`.
- Official anchors: Docker docs, Kubernetes docs, GitHub Actions docs, GitLab CI docs, cloud provider runtime docs.

## Summary

A deployment pipeline is part of the product. It must be reproducible, observable, and reversible.
