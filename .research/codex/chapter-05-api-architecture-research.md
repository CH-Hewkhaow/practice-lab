# Chapter 5. API Architecture Research

## 1. Executive Summary

Architecture in Go should reduce ambiguity and cognitive load, not add abstraction for its own sake. Unlike enterprise Java environments where extensive layering is often expected by default, idiomatic Go favors pragmatic simplicity. The primary research target of this chapter is to define how to choose the "smallest possible architecture" that still protects core domain business logic and ensures operational safety as the application scales. This chapter deeply explores Standard Layouts, Clean Architecture, Hexagonal Architecture, and Domain-Driven Design (DDD) specifically through the lens of Go.

## 2. Definitions and Key Terms

- **Domain Logic:** The core business rules of the application. It should know nothing about HTTP, databases, or JSON.
- **Port / Interface:** A Go `interface{}` that defines a contract. The domain defines the interface it *needs* (e.g., `UserRepository`), and the outer layers implement it.
- **Adapter:** The concrete implementation of a port. An HTTP Handler is an adapter for user input. A Postgres repository is an adapter for database output.
- **Dependency Inversion:** A principle stating that high-level modules (domain) should not depend on low-level modules (database). Both should depend on abstractions (interfaces).

## 3. Background and Context

Developers coming to Go from Ruby on Rails or Django often look for the "MVC" (Model-View-Controller) folder structure. When they don't find a unified Go MVC framework, they often make one of two mistakes:
1. They put all code in one massive `main.go` file (the "Flat" layout).
2. They attempt to strictly port a massive Java Spring architecture, resulting in hundreds of tiny files with interfaces for every single struct (the "Over-Engineered" layout).

Go requires a middle ground. Because Go compiles fast, has no circular dependency tolerance, and relies on structural typing (implicit interfaces), architecture in Go is primarily about managing package boundaries.

## 4. Core Concepts

The foundation of any Go architecture relies on these rules:
- **No Circular Dependencies:** Package A cannot import Package B if Package B imports Package A. Good architecture prevents this naturally by having clear dependency direction (outside-in).
- **The Dependency Rule:** Dependencies must point inward toward the domain logic. 
- **Small Interfaces:** Go interfaces should be small, often 1-3 methods (e.g., `io.Reader`).

## 5. Detailed Explanation of Architectural Patterns

### 5.1 Standard Go Project Layout
A pragmatic default for most services. It is not an official standard maintained by the Go core team, but it is heavily adopted by the community.
- `cmd/`: Contains the main applications for this project. E.g., `cmd/api/main.go`.
- `internal/`: Private application and library code. This code cannot be imported by other repositories.
- `pkg/`: Library code that is okay to use by external applications.

### 5.2 Layered Architecture
Familiar and easy to onboard. Separates code by technical concern:
- **Handler Layer (HTTP):** Parses JSON, validates input.
- **Service Layer (Business):** Executes business rules.
- **Repository Layer (Database):** Executes SQL.
*Risk:* These layers often become "leaky." Handlers start doing database lookups, or Repositories start returning HTTP status codes.

### 5.3 Clean / Hexagonal Architecture (Ports and Adapters)
These are variations of the same concept: isolating the domain.
- The **Domain** is at the center. It consists of pure Go structs and logic.
- The **Use Cases** orchestrate the domain logic.
- The **Adapters** sit at the edges. (HTTP, gRPC, PostgreSQL, Redis).
If you want to swap PostgreSQL for MongoDB, you only rewrite the database adapter; the Domain and Use Cases do not change.

### 5.4 Domain-Driven Structure (Feature-Based Layout)
Instead of grouping by technical layer (e.g., all handlers in one folder), files are grouped by business capability (e.g., `user/`, `billing/`).
Each feature folder contains its own handler, service, and repository.

## 6. Step-by-Step Process for Architecting a Go API

1. **Start Flat:** For a brand new project, put everything in `main.go` or a single `app` package.
2. **Extract Handlers:** When routing gets messy, pull HTTP handlers into a `delivery/http` package.
3. **Extract Domain:** When business logic gets mixed with HTTP, define pure structs in a `domain/` package.
4. **Extract Persistence:** When database queries clutter the handlers, extract them into `repository/` and define an interface in the domain.
5. **Move to internal/:** Hide these packages in `internal/` so they aren't accidentally exposed.

## 7. Practical Examples

**A Clean Architecture Directory Structure:**
```text
cmd/
  api/
    main.go           # Wires dependencies and starts server
internal/
  domain/
    user.go           # Structs and Repository Interfaces
  user/
    delivery/
      http_handler.go # Parses HTTP, calls UseCase
    usecase/
      user_ucase.go   # Business logic, calls Repository
    repository/
      postgres_repo.go# Implements domain.UserRepository interface
```

**Interface Injection Example:**
```go
// In domain/user.go
type UserRepository interface {
    GetByID(ctx context.Context, id int) (*User, error)
}

// In user/usecase/user_ucase.go
type UserUseCase struct {
    repo domain.UserRepository // Depends on interface, not postgres!
}
```

## 8. Real-World Use Cases

- **Simple CLI or Cron Job:** Use the *Standard Go Project Layout*. Just `cmd/job/main.go` and a few helper packages.
- **Mid-Sized SaaS Backend:** Use *Feature-Based Domain-Driven Structure*. It keeps teams from stepping on each other's toes when working on `billing` vs `users`.
- **Enterprise Fintech Core:** Use *Strict Clean Architecture*. Business rules must be 100% unit-testable without spinning up databases or HTTP servers.

## 9. Comparisons and Alternatives

| Scale | Recommended Style | Tradeoff / Alternative |
| ----- | ----------------- | ---------------------- |
| Small API | Simple / standard layout | Easy to outgrow; quickly becomes a "big ball of mud." |
| Medium API | Layered with strong boundaries | Requires discipline to prevent layers from leaking. |
| Enterprise API | DDD or Hexagonal with feature packages | High upfront boilerplate; requires deep DI wiring. |

## 10. Benefits and Advantages of Strong Architecture

- **Testability:** By depending on interfaces, you can instantly mock the database and write blazing-fast unit tests for your business logic.
- **Maintainability:** You can upgrade your HTTP router (e.g., from Gin to standard `net/http`) without touching your business logic.
- **Team Scaling:** Multiple developers can work on different feature packages simultaneously with zero merge conflicts.

## 11. Limitations, Risks, and Tradeoffs

- **Premature Abstraction:** Creating interfaces for everything before you actually need multiple implementations (or mocks) clutters the codebase.
- **The "Folder Soup" Problem:** Overly strict adherence to Clean Architecture can result in navigating 7 directories deep just to change a SQL query.

## 12. Edge Cases

- **Proxy Services:** If a microservice's sole job is to receive an HTTP request and forward it to another gRPC service, strict Clean Architecture is useless. It has no business logic. A flat handler is sufficient.

## 13. Common Mistakes and Misconceptions

- **Mistake:** Returning HTTP status codes from the database repository (e.g., returning `http.StatusNotFound` when a SQL row is missing). This violates dependency inversion; the DB layer should not know about HTTP.
- **Mistake:** Creating a single massive `models` package that every other package imports, leading to tight coupling and potential circular dependencies.
- **Misconception:** "Go needs a framework to be structured." False. Go's package system *is* the structuring mechanism.

## 14. Best Practices

- **Accept Interfaces, Return Structs:** (Postel's Law applied to Go). When a function requires a dependency, ask for an interface. When it returns a value, return the concrete struct.
- **Keep `main.go` Clean:** `main.go` should only read config, wire dependencies, start the server, and handle graceful shutdown.
- **Use `internal/` Aggressively:** Default to putting code in `internal/` unless you are explicitly building an open-source library meant for consumption.

## 15. Open Questions and Further Research

- How to best handle cross-cutting concerns (like database transactions that span multiple DDD feature boundaries) without violating Clean Architecture? (This is a notorious problem in Go).
- Exploring the use of code generation tools (like Wire) to manage the massive amount of Dependency Injection boilerplate required by Hexagonal architecture.

## 16. References or Source Notes

- `Standard Go Project Layout` (github.com/golang-standards/project-layout - note: community-driven, not official).
- "Clean Architecture" by Robert C. Martin.
- "Hexagonal Architecture" (Ports and Adapters) by Alistair Cockburn.

## 17. Chapter Summary

The right architecture in Go is the one that keeps business logic stable and testable while allowing external concerns (routers, databases, deployments) to be swapped or upgraded safely. For most medium-to-large production APIs, a Domain-Driven, feature-based layout built on Clean Architecture principles (utilizing Go interfaces for dependency inversion) offers the best balance of maintainability, testability, and team scalability.
