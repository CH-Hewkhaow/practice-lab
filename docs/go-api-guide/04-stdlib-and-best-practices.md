# Go API Reference Guide: Part 4 - Stdlib & Best Practices

This final section covers essential packages from the standard library, advanced error handling, performance considerations, and an API design checklist.

---

## 8. Standard Library Deep Dive

Go's standard library is famously comprehensive. Always prefer the standard library over third-party dependencies when possible.

### 8.1. `io` and `os` (I/O Boundaries)
The `io.Reader` and `io.Writer` interfaces are the fundamental abstractions for data streams in Go.

```go
import (
    "io"
    "os"
)

// CopyFile demonstrates using io.Copy to move data between os.Files
// without loading the entire file into memory.
func CopyFile(src, dst string) error {
    source, err := os.Open(src)
    if err != nil {
        return err
    }
    defer source.Close()

    destination, err := os.Create(dst)
    if err != nil {
        return err
    }
    defer destination.Close()

    _, err = io.Copy(destination, source)
    return err
}
```

### 8.2. `database/sql` (Database APIs)
The `database/sql` package provides a generic interface around SQL databases. You must import a driver (like `github.com/lib/pq` for Postgres).

```go
import (
    "database/sql"
    _ "github.com/lib/pq" // Blank import to register the driver
)

func fetchUserName(db *sql.DB, id int) (string, error) {
    var name string
    // Use parameter binding (?) to prevent SQL injection
    err := db.QueryRow("SELECT name FROM users WHERE id = $1", id).Scan(&name)
    if err != nil {
        if err == sql.ErrNoRows {
            return "", nil // Handle not found explicitly
        }
        return "", err
    }
    return name, nil
}
```

---

## 9. API Versioning & Backward Compatibility

**Rule #1 of Go APIs: Don't break backward compatibility.**

1.  **SemVer:** Use Semantic Versioning (`vMAJOR.MINOR.PATCH`).
2.  **Go Modules:** Go modules handle versioning natively. If you make a breaking change (v2.0.0), you must update your module path (e.g., `github.com/user/project/v2`).
3.  **Additive Changes:** Prefer adding new functions or fields rather than changing existing ones.

```go
// v1.0.0
func Process(data []byte) error { ... }

// v1.1.0 (Non-breaking additive change)
func ProcessWithOptions(data []byte, opts Options) error { ... }
```

---

## 10. Advanced Error Handling

Since Go 1.13, errors can wrap other errors to provide context.

### 10.1. Wrapping Errors (`fmt.Errorf` with `%w`)
Always provide context when returning an error from a lower layer.

```go
import (
    "errors"
    "fmt"
)

var ErrDatabaseConnection = errors.New("database connection failed")

func ConnectDB() error {
    // ... connection fails
    underlyingErr := fmt.Errorf("timeout after 5s")
    
    // Wrap the error
    return fmt.Errorf("ConnectDB: %w: %w", ErrDatabaseConnection, underlyingErr)
}
```

### 10.2. Checking Errors (`errors.Is` and `errors.As`)
*   **`errors.Is(err, target)`:** Checks if `err` or any wrapped error matches the `target` value.
*   **`errors.As(err, &target)`:** Checks if `err` or any wrapped error matches the *type* of `target`, and extracts it.

```go
err := ConnectDB()
if errors.Is(err, ErrDatabaseConnection) {
    // We know it's a connection error, regardless of how many times it was wrapped.
}
```

---

## 11. Performance Notes & Anti-Patterns

*   **Anti-Pattern:** Passing large structs by value. Pass a pointer (`*MyStruct`) to avoid copying memory.
*   **Anti-Pattern:** Concurrency for the sake of it. Goroutines have overhead. Don't use them for trivial operations that execute faster sequentially.
*   **Anti-Pattern:** Unbounded concurrency. Always use worker pools or semaphores when iterating over massive datasets to prevent exhausting system resources.
*   **Performance:** Pre-allocate slice capacity if the size is known: `make([]int, 0, capacity)`.

---

## 12. API Design Checklist

Before releasing a Go API package or HTTP service, review this checklist:

- [ ] **Minimal Exported Surface:** Are all structs, fields, and functions that *can* be unexported actually unexported?
- [ ] **Context Aware:** Do all blocking operations accept a `context.Context`?
- [ ] **Documentation:** Does every exported identifier have a doc comment?
- [ ] **Zero Value Usability:** Can your structs be used usefully in their zero state? (e.g., `sync.Mutex` is ready to use immediately).
- [ ] **Interface Returns:** Are you returning concrete structs and accepting interfaces?
- [ ] **Errors:** Are errors values? Are they wrapped to provide context? Do you avoid panics?
- [ ] **Naming:** Do package names avoid stutter? (e.g., `sql.DB` not `sql.SQLDB`).
- [ ] **Testing:** Is there comprehensive test coverage using `testing` package? Are examples provided as tests (which `go doc` will render)?