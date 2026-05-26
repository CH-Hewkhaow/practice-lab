# Go API Reference Guide

Welcome to the comprehensive Go API Reference Guide. This multi-part series covers Go from fundamental syntax to advanced API architecture, with a strong emphasis on clean design and the standard library.

## Full Table of Contents

*   **Part 1: Fundamentals & Package Boundaries** (This Document)
*   **Part 2: Concurrency & Generics**
*   **Part 3: Building Go APIs (Library, REST, CLI)**
*   **Part 4: Standard Library Deep Dive & Best Practices**

---

## Part 1: Fundamentals & Package Boundaries

### 1. Go Fundamentals

#### 1.1. Syntax, Variables, and Constants
Go prioritizes readability and simplicity. 

```go
package main

import "fmt"

// Constants
const APIVersion = "v1.0.0"

func main() {
    // Explicit typing
    var port int = 8080
    
    // Type inference (idiomatic)
    host := "localhost"
    
    fmt.Printf("Starting server on %s:%d (API %s)\n", host, port, APIVersion)
}
```

#### 1.2. Types: Structs, Pointers, and Interfaces
Go is strongly typed. **Structs** group data, while **Interfaces** define behavior.

```go
// Structs hold state
type User struct {
    ID       string
    Username string
    isActive bool   // unexported (private to package)
}

// Interfaces define behavior
type UserRepository interface {
    FindByID(id string) (*User, error)
    Save(u *User) error
}
```

**Pointers (`*`)** are used for mutation or avoiding large memory copies:
```go
func (u *User) Activate() {
    u.isActive = true // Mutates the original struct
}
```

#### 1.3. Collections: Slices and Maps
**Slices** are dynamic arrays, the workhorse of Go data structures.
**Maps** are hash tables for key-value pairs.

```go
// Slice initialization
users := []User{
    {ID: "1", Username: "alice"},
    {ID: "2", Username: "bob"},
}

// Appending to slices
users = append(users, User{ID: "3", Username: "charlie"})

// Map initialization
cache := make(map[string]*User)
cache["1"] = &users[0]
```

#### 1.4. Errors
Errors in Go are values, returned as the last value from functions. There are no exceptions (only panics, which should be reserved for unrecoverable state).

```go
import "errors"

var ErrNotFound = errors.New("record not found")

func getUser() (*User, error) {
    // ... lookup logic
    return nil, ErrNotFound
}
```

---

### 2. Package Structure & Boundaries

A Go API's primary boundary is its package. How you structure your packages dictates how others interact with your code.

#### 2.1. Exported vs. Unexported Identifiers
In Go, visibility is controlled by capitalization.

*   **Exported (Public):** Starts with an uppercase letter (e.g., `Server`, `Connect()`). Usable by any other package.
*   **Unexported (Private):** Starts with a lowercase letter (e.g., `server`, `connect()`). Only visible within the defining package.

**Best Practice:** Expose the absolute minimum required surface area. Return unexported structs via exported interfaces if the internal state should not be tampered with.

```go
package database

// Exported: API consumers use this
type DBClient struct {
    conn string
}

// Unexported: Implementation detail, hidden from consumers
func (c *DBClient) ping() error {
    return nil 
}

// Exported constructor
func NewClient(connectionString string) *DBClient {
    return &DBClient{conn: connectionString}
}
```

#### 2.2. Package Naming Conventions
*   **Keep it short, clear, and lowercase:** e.g., `time`, `http`, `models`.
*   **Avoid stutter:** Don't name a package `user` and its main struct `UserModel`. That forces the caller to write `user.UserModel`. Name it `User`, so callers write `user.User`.
*   **Avoid meaningless names:** `utils`, `helpers`, and `common` are anti-patterns. Group by capability (e.g., `strings`, `math`).

#### 2.3. Public Interfaces & Documentation Comments
Documentation is a first-class citizen in Go APIs. Use `go doc`.

*   Every exported identifier must have a doc comment.
*   The comment must begin with the name of the identifier.

```go
package payment

// Processor defines the required behavior for processing payments.
// Any backend (Stripe, PayPal) implementing this interface can be used.
type Processor interface {
    // Charge attempts to bill the provided amount.
    // It returns ErrInsufficientFunds if the card is declined.
    Charge(amount int) error
}
```

#### 2.4. The `internal/` Package Boundary
Go has a unique feature: any package placed inside a directory named `internal/` can only be imported by packages rooted in the parent of `internal/`. 

This is the ultimate tool for API boundary control. Use it to share code across your own packages without exposing it to third-party consumers.

```text
myproject/
├── main.go
├── api/
│   └── public.go       // CAN import internal/auth
└── internal/
    └── auth/
        └── auth.go     // CANNOT be imported by outside projects!
```