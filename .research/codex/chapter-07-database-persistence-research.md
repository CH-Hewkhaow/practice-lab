# Chapter 7. Database & Persistence Research

## 1. Executive Summary

A Go API is only as reliable and performant as its persistence layer. In Go, the persistence strategy should follow the shape of the domain and the performance envelope, rather than relying on framework convenience (a stark contrast to ecosystems dominated by ActiveRecord or Hibernate). This chapter researches the database integration landscape in Go, evaluating the `database/sql` standard library, native drivers like `pgx`, and the spectrum of tooling from raw SQL generators (`sqlc`) to heavy Object-Relational Mappers (GORM, Ent). The ultimate recommendation for enterprise Go APIs is to favor explicit, type-safe SQL generation (PostgreSQL + `pgx` + `sqlc`) over reflection-heavy ORMs.

## 2. Definitions and Key Terms

- **ORM (Object-Relational Mapper):** A tool that maps database tables to Go structs and translates method calls into SQL queries automatically (e.g., GORM).
- **Query Builder:** A tool that allows developers to construct SQL queries using Go methods instead of raw strings, providing syntax validation without hiding the SQL (e.g., Squirrel, Bun).
- **Code Generator:** A tool that reads raw SQL schemas and queries, then generates type-safe Go structs and functions to execute them (e.g., sqlc).
- **Connection Pooling:** Maintaining a cache of database connections so they can be reused for future requests, avoiding the massive overhead of opening a new TCP connection per query.
- **N+1 Problem:** A performance anti-pattern where an application queries the database once for a list of $N$ parent records, and then executes $N$ additional queries to fetch child records, rather than using a single `JOIN`.

## 3. Background and Context

Go’s standard library provides `database/sql`, a generic interface around SQL (or SQL-like) databases. It requires a driver (like `lib/pq` or `pgx` for Postgres) to function. Historically, developers either wrote raw SQL strings using `database/sql` (which is prone to runtime errors and SQL injection if misused) or adopted GORM to mimic the Rails/Laravel development speed they were used to. 

However, as Go applications scale to handle high concurrency, the reflection overhead of GORM often becomes a bottleneck. The community has shifted heavily toward "schema-first" tools like `sqlc` which guarantee compile-time safety without runtime reflection costs.

## 4. Core Concepts

Before selecting a tool, a production API must master these database concepts in Go:
- **Context Propagation:** Every database call must take a `context.Context`. If an HTTP request is cancelled by the user, the database query must also be cancelled to prevent resource exhaustion.
- **Connection Tuning:** By default, `database/sql` has an unlimited connection pool. This is dangerous. `SetMaxOpenConns`, `SetMaxIdleConns`, and `SetConnMaxLifetime` must be explicitly configured.
- **Transactions (`Tx`):** Managing atomic operations across multiple tables requires explicitly passing a `sql.Tx` object down through the repository layer.

## 5. Detailed Explanation of Database Systems

### 5.1 PostgreSQL
The undisputed default relational database for most serious Go APIs. Its strong transactional model, advanced indexing, and native JSONB support make it versatile. In Go, the `jackc/pgx` driver is overwhelmingly preferred over the older `lib/pq` because `pgx` offers native Postgres features, better performance, and standard `database/sql` compatibility.

### 5.2 MySQL
A solid choice for legacy compatibility or when an organization already has deep MySQL DBA expertise. Use the `go-sql-driver/mysql` driver.

### 5.3 SQLite
Incredible for unit testing, local development, or small edge-deployed tools (e.g., IoT devices). With CGO-free drivers like `modernc.org/sqlite`, it is easier to compile than ever. However, it is not the default for heavily concurrent web APIs due to write-locking constraints.

### 5.4 MongoDB & Redis
- **MongoDB:** Useful when data is strictly document-shaped and schema-less. Requires the `go.mongodb.org/mongo-driver`.
- **Redis:** Mandatory for caching, rate-limiting, and distributed session management (using `go-redis/redis`), but rarely the primary source of truth.

## 6. Step-by-Step Process for Database Design

1. **Schema First:** Write raw SQL `CREATE TABLE` statements.
2. **Migration Management:** Use a tool like `goose` or `golang-migrate` to track schema changes.
3. **Query Definition:** Write the raw SQL queries you need (e.g., `SELECT * FROM users WHERE id = $1`).
4. **Code Generation:** Run `sqlc` to generate the Go structs and interface methods.
5. **Repository Implementation:** Wrap the generated `sqlc` code in a domain repository interface.

## 7. Practical Examples

**GORM (Code-First, Reflection-Based):**
```go
// Struct defines schema
type User struct {
    gorm.Model
    Name string
}
// Magic query execution
db.Where("name = ?", "jinzhu").First(&user)
```
*Pros:* Fast to write. *Cons:* Hidden queries, runtime panics on typos.

**sqlc (Schema-First, Generation-Based):**
```sql
-- query.sql
-- name: GetUser :one
SELECT * FROM users WHERE name = $1 LIMIT 1;
```
```go
// Generated Go Code (Compile-time safe)
user, err := queries.GetUser(ctx, "jinzhu")
```
*Pros:* Blazing fast, pure Go types, compile-time SQL validation. *Cons:* Must write SQL manually.

## 8. Real-World Use Cases

- **Fast Prototyping / MVP:** If you have 3 weeks to launch a basic CRUD app, **GORM** is a perfectly valid choice.
- **High-Performance Financial Service:** If you are processing ledgers where transaction boundaries and exact SQL locking (`SELECT ... FOR UPDATE`) matter, **sqlc** or pure `pgx` is mandatory.
- **Complex Graph Data:** If your domain involves heavy relationships (e.g., a social network), Facebook's **Ent** framework provides a graph-based ORM that handles complex traversals safely.

## 9. Comparisons and Alternatives

| Tool | Type | Strength | Weakness | Best Use Case |
| ---- | ---- | -------- | -------- | ------------- |
| `database/sql` | Stdlib | Portable, explicit | Massive boilerplate for row scanning | Simple queries |
| `pgx` (Native) | PG Driver | Fastest Postgres access | Locked to Postgres | High-load Postgres APIs |
| **SQLC** | Codegen | Type-safe, fast, pure SQL | Requires SQL knowledge | **Enterprise Default** |
| GORM | ORM | Dev speed, auto-migrations | Reflection overhead, hidden N+1s | Rapid MVPs |
| Ent | Graph ORM | Strong type safety, traversals | Steep learning curve | Complex domains |
| Bun | Builder | Flexible SQL ergonomics | Manual mapping still needed | Dynamic query apps |
| Squirrel | Builder | Dynamic WHERE clauses | No type-safe mapping | Complex filtering APIs |

## 10. Benefits and Advantages of `pgx` + `sqlc`

- **Performance:** `sqlc` generates exact `row.Scan` code. There is zero reflection overhead at runtime.
- **Transparency:** You know exactly what query is sent to the database. There is no ORM magic generating a 10-table JOIN behind your back.
- **Safety:** If you change a column name in your SQL schema, `sqlc` will fail to compile your Go code, catching the bug instantly.

## 11. Limitations, Risks, and Tradeoffs

- **Dynamic Queries:** `sqlc` struggles with highly dynamic queries (e.g., an admin dashboard with 20 optional filters). In these scenarios, a query builder like `Squirrel` or `Bun` must be used alongside `sqlc`.
- **Developer Friction:** Developers accustomed to Laravel Eloquent or Django ORM will find writing raw SQL migrations tedious at first.

## 12. Edge Cases

- **Read Replicas:** Native `database/sql` does not automatically route `SELECT` queries to a read replica and `UPDATE` queries to a master. This requires custom wrapper logic or middleware at the connection pool level.
- **Distributed Tracing:** Ensure that the database driver is wrapped with OpenTelemetry (e.g., `otelsql`) so every query shows up in your tracing dashboard.

## 13. Common Mistakes and Misconceptions

- **Mistake:** Failing to call `defer rows.Close()`. This permanently leaks a database connection from the pool.
- **Mistake:** Using `db.Query` for an INSERT or UPDATE and ignoring the result. Always use `db.Exec` if you don't need rows returned.
- **Misconception:** "Go ORMs are as advanced as Java's Hibernate." They are not, and because Go lacks deep inheritance, attempting to use them like Hibernate leads to terrible performance.

## 14. Best Practices

- **Strict Connection Limits:** 
  ```go
  db.SetMaxOpenConns(25)
  db.SetMaxIdleConns(25)
  db.SetConnMaxLifetime(5 * time.Minute)
  ```
- **Migrations in CI/CD:** Do not use GORM's `AutoMigrate` in production. Use explicit `.sql` files managed by `golang-migrate` and run them during the CI/CD pipeline deployment.

## 15. Open Questions and Further Research

- As Go generics mature, will a new wave of type-safe, generic ORMs emerge that bridge the gap between GORM's ease of use and `sqlc`'s performance?
- How to best handle multi-tenant database architectures (row-level security vs. separate schemas) within Go repositories.

## 16. References or Source Notes

- [SQLC Documentation](https://sqlc.dev/)
- [jackc/pgx GitHub](https://github.com/jackc/pgx)
- [Go `database/sql` Tutorial](https://go.dev/doc/database/)
- [GORM Documentation](https://gorm.io/)

## 17. Chapter Summary

The most resilient Go APIs rely on explicitly defined SQL and explicit error handling. While ORMs like GORM provide speed during prototyping, they obscure database interactions and introduce runtime overhead. The recommended production standard for enterprise Go APIs is a combination of **PostgreSQL**, the **pgx** driver, and **sqlc** for compile-time safe code generation. This stack ensures that database access is transparent, performant, and fully context-aware.
