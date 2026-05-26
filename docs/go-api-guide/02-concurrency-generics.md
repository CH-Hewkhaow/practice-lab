# Go API Reference Guide: Part 2 - Concurrency & Generics

This section covers Go's powerful concurrency primitives and the recently introduced type parameters (generics), both of which are critical for designing high-performance, reusable APIs.

---

## 3. Concurrency Model

Go's concurrency is built on the philosophy: *"Do not communicate by sharing memory; instead, share memory by communicating."*

### 3.1. Goroutines and Channels
A **goroutine** is a lightweight thread managed by the Go runtime.
A **channel** is a typed conduit through which you can send and receive values with the channel operator, `<-`.

```go
package main

import (
    "fmt"
    "time"
)

// API Design: Pass channels to functions rather than returning them
// when you want to stream results.
func processJobs(jobs <-chan int, results chan<- string) {
    for j := range jobs {
        fmt.Printf("Processing job %d...\n", j)
        time.Sleep(time.Millisecond * 100)
        results <- fmt.Sprintf("Result of job %d", j)
    }
}

func main() {
    jobs := make(chan int, 5)     // Buffered channel
    results := make(chan string, 5)

    // Start 3 worker goroutines
    for w := 1; w <= 3; w++ {
        go processJobs(jobs, results)
    }

    // Send 5 jobs
    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs) // Important: close channels when no more data will be sent

    // Collect results
    for a := 1; a <= 5; a++ {
        fmt.Println(<-results)
    }
}
```

### 3.2. The `context` Package
The `context` package is mandatory for any blocking or long-running API. It carries deadlines, cancellation signals, and request-scoped values across API boundaries.

**API Rule:** `context.Context` should always be the **first parameter** of any function that performs I/O or blocks.

```go
import (
    "context"
    "database/sql"
    "time"
)

// Good API Design: Accepts context
func GetUserByID(ctx context.Context, db *sql.DB, id string) (*User, error) {
    // Pass the context to the database query
    row := db.QueryRowContext(ctx, "SELECT id, name FROM users WHERE id = ?", id)
    // ...
}

// Caller usage:
func caller() {
    // Timeout after 2 seconds
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel() // Always defer cancel to prevent context leaks

    user, err := GetUserByID(ctx, db, "123")
}
```

### 3.3. The `sync` Package
While channels are preferred for communication, sometimes you just need to protect a shared resource or wait for a group of goroutines to finish.

#### `sync.Mutex`
Used to protect state from concurrent read/writes. Embed it in your structs.

```go
import "sync"

type SafeCounter struct {
    mu sync.Mutex
    v  map[string]int
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
    c.mu.Lock()
    defer c.mu.Unlock() // Use defer to ensure the lock is always released
    c.v[key]++
}
```

#### `sync.WaitGroup`
Used to wait for a collection of goroutines to finish executing.

```go
import "sync"

func fetchAll(urls []string) {
    var wg sync.WaitGroup

    for _, url := range urls {
        wg.Add(1) // Increment the counter before the goroutine starts
        go func(u string) {
            defer wg.Done() // Decrement when the goroutine completes
            fetch(u)
        }(url) // Pass the loop variable to prevent closure variable capture issues
    }

    wg.Wait() // Block until the counter goes back to 0
}
```

---

## 4. Generics (Type Parameters)

Introduced in Go 1.18, generics allow you to write functions and structs that can operate on multiple types, ensuring type safety without relying on `interface{}`.

### 4.1. Generic Functions

```go
// T is a type parameter constrained by the 'any' interface
func PrintSlice[T any](s []T) {
    for _, v := range s {
        fmt.Println(v)
    }
}

// Usage:
PrintSlice([]int{1, 2, 3})
PrintSlice([]string{"a", "b", "c"})
```

### 4.2. Type Constraints
You can restrict the types that a generic function or struct accepts. `comparable` is a built-in constraint for types that support `==` and `!=`.

```go
// K must be comparable, V can be anything
func GetKeys[K comparable, V any](m map[K]V) []K {
    keys := make([]K, 0, len(m))
    for k := range m {
        keys = append(keys, k)
    }
    return keys
}
```

### 4.3. API Design Implications with Generics
*   **Don't overuse them:** If an `interface` solves the problem elegantly, use an interface. Generics are best for data structures (like a custom Cache or Stack) or utility functions (like sorting or mapping arrays).
*   **Generic Structs:** Useful for creating container types.

```go
// A generic stack implementation
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}
```