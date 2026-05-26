# Backend Architecture: Hybrid Clean & DDD

This guide outlines a **Hybrid Clean/DDD (Domain-Driven Design)** architecture for Go. It centralizes shared domain models and infrastructure while organizing business logic into modular, feature-based packages.

---

## 1. Project Directory Structure

This layout ensures that infrastructure (migrations) and the core domain (structs) are decoupled from the specific business features.

```text
.
├── migrations/              # SQL Migration files (Goose)
├── cmd/
│   └── api/
│       └── main.go          # Dependency Injection & App Start
├── internal/
│   ├── domain/              # Centralized "Source of Truth"
│   │   ├── user.go          # Shared Structs (User, Profile)
│   │   ├── product.go       # Shared Structs (Product, Category)
│   │   └── interfaces.go    # Global interfaces (if any)
│   ├── shared/              # Cross-cutting concerns
│   │   ├── response/        # Global API response helpers
│   │   ├── apperror/        # Custom error types
│   │   └── pager/           # Pagination logic
│   ├── features/            # Feature-based business logic
│   │   ├── auth/
│   │   │   ├── delivery/    # HTTP Handlers (API)
│   │   │   ├── usecase/     # Business Logic
│   │   │   └── repository/  # Database Implementation
│   │   └── catalog/
│   │       ├── delivery/
│   │       ├── usecase/
│   │       └── repository/
│   └── infrastructure/      # Drivers & Shared Config
│       ├── config/
│       ├── database/
│       └── cache/
└── Makefile
```

---

## 2. The Centralized Domain (`internal/domain`)

The domain layer contains only **data structures** and should have zero dependencies on other internal packages. This makes it easy to share types with the frontend or other services.

**Example (`internal/domain/user.go`):**
```go
package domain

import "time"

type User struct {
	ID        string    `json:"id"`
	Email     string    `json:"email"`
	Password  string    `json:"-"`
	CreatedAt time.Time `json:"created_at"`
}
```

---

## 3. Feature-Based Modules (`internal/features`)

Each feature follows the "Clean" layering:

### A. Delivery (Handler)
Responsible for parsing requests (JSON) and returning responses. It **only** calls the UseCase.

### B. UseCase (Service)
Contains the **Pure Business Logic**. It is agnostic to whether the data comes from HTTP or a CLI. It calls the Repository via an interface.

### C. Repository (Data)
Implements the database-specific logic (SQL, Redis). It returns Domain models to the UseCase.

---

## 4. Example Data Flow (User Registration)

1.  **Delivery:** `auth/delivery/handler.go` receives a POST request.
2.  **UseCase:** `auth/usecase/register.go` checks if the email exists, hashes the password, and creates a `domain.User` struct.
3.  **Repository:** `auth/repository/postgres.go` saves the `domain.User` to the database using `pgx`.

---

## 5. Benefits of this Structure

1.  **Scalability:** New features (e.g., `orders`, `billing`) can be added in their own folders without touching existing code.
2.  **Shared Models:** By keeping models in `internal/domain`, you avoid "circular dependency" errors which are common in Go when features need to reference each other's types.
3.  **Infrastructure Isolation:** Migrations live in their own root folder, ensuring your Go code only cares about the schema, not the migration tool's state.
4.  **Testability:** You can easily mock the Repository layer to unit test the UseCase business logic.
