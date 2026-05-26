# Go API Reference Guide: Part 3 - Building Go APIs

This section focuses on the structural and architectural patterns used to build specific kinds of Go APIs.

---

## 5. Building Library APIs

When building a library for other Go developers to import, the API must be clean, discoverable, and extensible.

### 5.1. The Options Pattern
When structs require many configuration parameters, avoid constructors with long parameter lists. Use the Functional Options Pattern.

```go
package server

import "time"

type Server struct {
    host    string
    port    int
    timeout time.Duration
}

// Option defines a functional configuration parameter
type Option func(*Server)

// WithPort configures the port
func WithPort(port int) Option {
    return func(s *Server) {
        s.port = port
    }
}

// WithTimeout configures the timeout
func WithTimeout(t time.Duration) Option {
    return func(s *Server) {
        s.timeout = t
    }
}

// NewServer applies all options dynamically
func NewServer(host string, opts ...Option) *Server {
    s := &Server{
        host:    host,
        port:    8080,               // default
        timeout: 30 * time.Second,   // default
    }
    for _, opt := range opts {
        opt(s)
    }
    return s
}

// Caller Usage:
// srv := server.NewServer("localhost", server.WithPort(9090))
```

### 5.2. Return Structs, Accept Interfaces
A core Go proverb: **Accept interfaces, return structs.**
If you return an interface, you force the caller to use exactly the abstraction you defined. If you return a struct, the caller can define their *own* interface that matches the subset of methods they actually need.

```go
// BAD: Returning an interface
func NewReader() io.Reader { return &myReader{} }

// GOOD: Returning the concrete struct
func NewReader() *MyReader { return &MyReader{} }
```

---

## 6. Building REST APIs & HTTP Handlers

Go's standard library `net/http` is robust enough that many developers build APIs without third-party frameworks.

### 6.1. The `http.Handler` Interface
Any struct that implements `ServeHTTP(w http.ResponseWriter, r *http.Request)` is an `http.Handler`.
Alternatively, `http.HandlerFunc` converts a regular function into a handler.

```go
package api

import (
    "encoding/json"
    "net/http"
)

type API struct {
    db Database
}

// JSON handler example
func (api *API) HandleGetUser(w http.ResponseWriter, r *http.Request) {
    id := r.URL.Query().Get("id")
    user, err := api.db.GetUser(id)
    if err != nil {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}
```

### 6.2. Middleware Pattern
Middleware in Go is simply a function that takes an `http.Handler` and returns an `http.Handler`.

```go
// Logging Middleware
func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("[%s] %s", r.Method, r.URL.Path)
        next.ServeHTTP(w, r) // Call the next handler in the chain
    })
}

// Setup
// mux := http.NewServeMux()
// mux.Handle("/user", LoggingMiddleware(http.HandlerFunc(api.HandleGetUser)))
```

### 6.3. JSON Encoding/Decoding (`encoding/json`)

```go
type RequestPayload struct {
    Name string `json:"name"`
    Age  int    `json:"age,omitempty"` // Omitted if 0
}

func HandlePost(w http.ResponseWriter, r *http.Request) {
    var payload RequestPayload
    
    // Decode from request body
    if err := json.NewDecoder(r.Body).Decode(&payload); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    defer r.Body.Close()

    // ... process payload
}
```

---

## 7. Command-Line APIs (CLI)

CLIs are a form of API where the human or shell script is the caller. The `flag` package handles argument parsing.

### 7.1. Using the `flag` Package

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    // Define flags
    port := flag.Int("port", 8080, "Port to listen on")
    configPath := flag.String("config", "config.yaml", "Path to config file")
    verbose := flag.Bool("v", false, "Enable verbose logging")

    // Parse flags
    flag.Parse()

    // Access remaining positional arguments
    args := flag.Args()

    if *verbose {
        fmt.Printf("Starting on port %d with config %s\n", *port, *configPath)
    }
    
    if len(args) > 0 {
        fmt.Printf("Positional args: %v\n", args)
    }
}
```