# Go Production-Ready Setup & Integration

This guide provides a professional blueprint for initializing a Go project integrated with Docker, PostgreSQL, and Redis, specifically designed to handle **Staging, UAT, and Production** environments following 12-factor app principles.

---

## 1. Project Structure

A clean layout is essential for scaling. We follow the standard Go project layout.

```text
.
├── cmd/
│   └── api/
│       └── main.go          # Application entry point
├── internal/
│   ├── config/              # Env var parsing & validation
│   │   └── config.go
│   ├── database/            # Postgres (pgx) setup
│   │   └── postgres.go
│   └── cache/               # Redis setup
│       └── redis.go
├── migrations/              # SQL files for goose
│   └── 20260513_init.sql
├── .env.example             # Template for env vars
├── Dockerfile               # Multi-stage production build
├── docker-compose.yml       # Local & UAT orchestration
├── Makefile                 # Automation for common tasks
└── go.mod
```

---

## 2. Environment Configuration (12-Factor)

We use `caarlos0/env` for type-safe configuration. This allows us to use the same binary across all environments, changing only the environment variables.

### Recommended Tool: `caarlos0/env`
Install: `go get github.com/caarlos0/env/v11`

**internal/config/config.go**
```go
package config

import (
	"github.com/caarlos0/env/v11"
	"log"
)

type Config struct {
	AppEnv      string `env:"APP_ENV" envDefault:"development"`
	Port        int    `env:"PORT" envDefault:"8080"`
	DatabaseURL string `env:"DATABASE_URL,required"`
	RedisURL    string `env:"REDIS_URL,required"`
}

func New() *Config {
	cfg := &Config{}
	if err := env.Parse(cfg); err != nil {
		log.Fatalf("failed to parse config: %v", err)
	}
	return cfg
}
```

---

## 3. Database & Migrations

### Tools: `pgx` & `goose`
*   **Driver:** `github.com/jackc/pgx/v5` (Performance & Type-safety)
*   **Migrations:** `github.com/pressly/goose/v3` (Version control for schema)

**Migrations Example (`migrations/20260513_init.sql`):**
```sql
-- +goose Up
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT UNIQUE NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- +goose Down
DROP TABLE users;
```

---

## 4. Production-Ready Docker

We use a **multi-stage build** to ensure the production image is minimal and secure.

**Dockerfile**
```dockerfile
# --- Stage 1: Build ---
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main ./cmd/api

# --- Stage 2: Run ---
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
COPY --from=builder /app/migrations ./migrations

# Expose port and run
EXPOSE 8080
CMD ["./main"]
```

---

## 5. Multi-Environment Orchestration

Use `docker-compose.yml` with profiles or multiple `.env` files to manage environments.

**docker-compose.yml**
```yaml
services:
  api:
    build: .
    ports:
    - "${PORT:-8080}:8080"
    env_file:
      - .env.${APP_ENV:-development}
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5
```

---

## 6. Suggested Automation (Makefile)

Wrap complex commands to avoid errors during staging or UAT deployments.

```makefile
.PHONY: up-staging up-uat migrate-up

up-staging:
	APP_ENV=staging docker-compose up -d --build

up-uat:
	APP_ENV=uat docker-compose up -d --build

migrate-up:
	goose -dir migrations postgres "$(DATABASE_URL)" up
```

---

## Summary of Suggested Tools

1.  **Framework:** `Standard Library` + `Chi` or `Echo` (for routing).
2.  **Database:** `pgx` (Postgres driver) and `goose` (Migrations).
3.  **Cache:** `go-redis/v9`.
4.  **Config:** `caarlos0/env` (12-factor compliance).
5.  **Validation:** `go-playground/validator` (for request payloads).
6.  **CLI/Task Runner:** `Makefile` or `Taskfile`.
