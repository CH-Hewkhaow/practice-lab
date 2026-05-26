````md
สร้างไฟล์ Markdown ชื่อ `golang-api-research-plan.md`

เป้าหมายของไฟล์นี้คือเป็น “แผนการ Research แบบครบถ้วน” สำหรับการศึกษาและออกแบบการทำ API ด้วย Go / Golang ตั้งแต่พื้นฐานจนถึง production-ready system

## เงื่อนไขสำคัญ

1. ก่อนเริ่มเขียนแผน ให้ตรวจสอบข้อมูล Research เดิมก่อน
   - ให้ค้นหาในโฟลเดอร์หลักชื่อ `document`
   - อ่านไฟล์ Research เดิมทั้งหมดที่เกี่ยวข้องกับ:
     - API
     - Backend
     - Golang
     - Integration
     - Architecture
     - Database
     - Security
     - Deployment
     - Testing
   - สรุปว่า Research เดิมมีอะไรอยู่แล้ว
   - ระบุว่ายังขาดหัวข้ออะไรบ้าง

2. หลังจากตรวจสอบเอกสารเดิมแล้ว ให้ Research ข้อมูล Go / Golang เพิ่มเติมให้ครอบคลุมทุกด้านของการทำ API และ Integration

3. ไฟล์ Markdown ต้องมี Research Source ชัดเจน
   - แยกเป็น Official Documentation
   - Community / Best Practice
   - Framework Documentation
   - Cloud / Deployment Reference
   - Security Reference
   - Testing Reference
   - Database / ORM Reference
   - Integration Reference

4. ต้องบังคับให้ Markdown ครอบคลุมทุก Topic สำคัญของ Go ที่เกี่ยวข้องกับ API จริง ๆ ห้ามข้ามหัวข้อสำคัญ

---

# โครงสร้าง Markdown ที่ต้องสร้าง

## 1. Research Objective

อธิบายเป้าหมายของ Research นี้ เช่น:

- ศึกษาการทำ REST API ด้วย Go
- ศึกษา API Architecture ที่เหมาะกับ Production
- ศึกษา Integration กับระบบภายนอก
- ศึกษา Best Practice ของ Go Backend
- ศึกษา Security, Testing, Logging, Observability, Deployment
- สร้าง Knowledge Base สำหรับใช้พัฒนา API จริง

---

## 2. Existing Document Review

ให้ตรวจสอบข้อมูลจากโฟลเดอร์ `document` ก่อน

ต้องมีตาราง:

| File | Related Topic | Key Finding | Reusable Knowledge | Missing Gap |
| ---- | ------------- | ----------- | ------------------ | ----------- |

จากนั้นสรุป:

### Existing Knowledge Summary

### Missing Research Gap

---

## 3. Golang API Research Scope

ต้องครอบคลุมหัวข้อต่อไปนี้ทั้งหมด:

### 3.1 Go Language Foundation

- Go syntax
- Package structure
- Module system
- `go.mod`
- Error handling
- Pointers
- Structs
- Interfaces
- Generics
- Context
- Goroutines
- Channels
- Mutex
- Worker pool
- Memory model
- Standard library
- File structure convention

### 3.2 API Fundamentals

- REST API
- HTTP method
- Status code
- Request / response lifecycle
- Header
- Query parameter
- Path parameter
- JSON body
- Multipart upload
- Pagination
- Filtering
- Sorting
- Versioning
- Idempotency
- Rate limit

### 3.3 Go HTTP Layer

- `net/http`
- Router / multiplexer
- Middleware
- Handler pattern
- Request validation
- Response formatter
- Error response format
- Graceful shutdown
- Timeout
- Context cancellation

### 3.4 Go API Frameworks

Research และเปรียบเทียบ:

- Standard `net/http`
- Gin
- Echo
- Fiber
- Chi
- Gorilla Mux

ต้องมีตาราง:

| Framework | Strength | Weakness | Best For | Production Suitability |
| --------- | -------- | -------- | -------- | ---------------------- |

### 3.5 Project Structure

ต้อง Research โครงสร้างแบบ:

- Simple structure
- Standard Go project layout
- Clean Architecture
- Hexagonal Architecture
- Layered Architecture
- Domain-driven structure

ต้องมีตัวอย่าง structure เช่น:

```txt
cmd/
internal/
pkg/
api/
handler/
service/
repository/
model/
config/
middleware/
migration/
test/
```
````

### 3.6 Configuration Management

- Environment variable
- `.env`
- Config file
- Secret management
- Viper
- Validation config
- Runtime config
- Separate config by environment

### 3.7 Database Integration

ครอบคลุม:

- `database/sql`
- PostgreSQL
- MySQL
- SQLite
- MongoDB
- Redis
- Connection pooling
- Transaction
- Migration
- Indexing
- Query optimization
- N+1 problem
- Repository pattern

### 3.8 ORM / Query Builder

Research:

- GORM
- SQLC
- Ent
- Bun
- Squirrel
- Raw SQL

ต้องมีตารางเปรียบเทียบ:

| Tool | Type | Strength | Weakness | Best Use Case |
| ---- | ---- | -------- | -------- | ------------- |

### 3.9 Authentication & Authorization

- JWT
- OAuth2
- OpenID Connect
- Session-based auth
- API key
- Refresh token
- RBAC
- ABAC
- Permission model
- Password hashing
- bcrypt / argon2
- Token expiration
- Token revocation

### 3.10 API Security

ต้องครอบคลุม:

- OWASP API Security Top 10
- CORS
- CSRF
- XSS
- SQL Injection
- Request size limit
- Rate limiting
- Input validation
- Output encoding
- Secure headers
- HTTPS / TLS
- Secret leakage
- Logging sensitive data
- Dependency vulnerability scan
- Supply chain security

### 3.11 Validation

- Request validation
- Struct tag validation
- Custom validation
- Go validator package
- Sanitization
- Business rule validation
- Error message design

### 3.12 Error Handling

- Go error pattern
- Sentinel error
- Wrapped error
- Custom error type
- Error mapping to HTTP status
- Global error handler
- Error response schema
- Logging error
- Avoid panic in API

### 3.13 Logging

Research:

- Standard log
- Zap
- Zerolog
- Logrus
- Structured logging
- Request ID
- Correlation ID
- Log level
- Audit log
- Sensitive data masking

### 3.14 Observability

- Metrics
- Tracing
- Logging
- OpenTelemetry
- Prometheus
- Grafana
- Jaeger
- Health check
- Readiness probe
- Liveness probe
- Performance monitoring

### 3.15 Testing

ต้องครอบคลุม:

- Unit test
- Integration test
- End-to-end test
- Table-driven test
- Mocking
- Testify
- GoMock
- httptest
- Test database
- Test container
- Coverage
- Benchmark test
- Race condition test

### 3.16 API Documentation

- OpenAPI
- Swagger
- Redoc
- Postman collection
- API contract
- Code-first vs spec-first
- Versioned API docs

### 3.17 External Integration

ต้องครอบคลุม:

- Calling external API
- HTTP client
- Retry
- Timeout
- Circuit breaker
- Backoff
- Webhook
- Event-driven integration
- Message queue
- Kafka
- RabbitMQ
- NATS
- gRPC
- GraphQL
- SOAP integration
- Third-party SDK
- Payment gateway
- Email service
- File storage
- Cloud API

### 3.18 File Upload / Download

- Multipart upload
- File validation
- MIME type check
- File size limit
- Virus scan concept
- Local storage
- S3-compatible storage
- Signed URL
- Streaming download

### 3.19 Background Jobs

- Goroutine worker
- Job queue
- Cron job
- Retry failed job
- Dead-letter queue
- Scheduled task
- Distributed worker

### 3.20 Concurrency in API

- Goroutine safety
- Context cancellation
- Shared state
- Mutex
- Channel
- Worker pool
- Rate control
- Race detector
- Graceful shutdown

### 3.21 Performance

- Profiling
- pprof
- Benchmark
- Memory allocation
- Garbage collection
- Connection reuse
- JSON performance
- Caching
- Compression
- Load testing
- Horizontal scaling

### 3.22 Caching

- In-memory cache
- Redis
- HTTP cache
- ETag
- Cache-Control
- CDN
- Cache invalidation
- Distributed cache

### 3.23 Deployment

- Docker
- Docker Compose
- Kubernetes
- Helm
- CI/CD
- GitHub Actions
- GitLab CI
- Environment promotion
- Blue-green deployment
- Canary deployment
- Rollback strategy

### 3.24 Cloud Integration

Research:

- AWS
- GCP
- Azure
- S3 / Cloud Storage
- Secret Manager
- Cloud Run
- ECS
- EKS
- Lambda
- API Gateway
- Load Balancer
- Managed database

### 3.25 DevOps & Runtime

- Build binary
- Cross-compilation
- Static binary
- Runtime image
- Distroless image
- Environment variable injection
- Log shipping
- Monitoring
- Alerting
- Backup
- Disaster recovery

### 3.26 Code Quality

- gofmt
- go vet
- golangci-lint
- staticcheck
- govulncheck
- pre-commit hook
- code review checklist
- naming convention
- package design
- interface design
- dependency management

### 3.27 Dependency Injection

- Manual DI
- Wire
- Fx
- Dig
- Constructor pattern
- Interface-based injection

### 3.28 API Design Best Practice

- Resource naming
- Consistent response format
- Error schema
- Pagination standard
- Filtering convention
- Versioning strategy
- Backward compatibility
- Deprecation strategy
- API contract testing

### 3.29 Production Checklist

ต้องมี Checklist แยกหมวด:

- Security
- Testing
- Performance
- Observability
- Deployment
- Database
- API design
- Documentation
- Incident response

### 3.30 Common Mistakes

ต้องสรุปข้อผิดพลาดที่พบบ่อย เช่น:

- ใช้ global state เยอะเกินไป
- ไม่ใช้ context
- ไม่มี timeout
- ไม่ handle error
- log sensitive data
- query database ไม่มี index
- ไม่มี graceful shutdown
- ไม่มี validation
- ไม่มี integration test
- ใช้ goroutine โดยไม่ control lifecycle

---

## 4. Research Method

ต้องอธิบายวิธี Research:

1. Review existing files in `document`
2. Identify reusable knowledge
3. Identify missing gaps
4. Research official Go documentation
5. Research framework documentation
6. Research production best practices
7. Research integration patterns
8. Compare tools and approaches
9. Summarize recommendation
10. Create implementation checklist

---

## 5. Research Source Requirement

ต้องใส่แหล่งข้อมูลอย่างน้อยจากกลุ่มต่อไปนี้:

### Official Go Sources

- Go Documentation
- Effective Go
- Go Blog
- Go Packages
- Go Security
- Go Vulnerability Management

### API / HTTP Sources

- MDN HTTP
- REST API design references
- OpenAPI Specification

### Framework Sources

- Gin documentation
- Echo documentation
- Fiber documentation
- Chi documentation
- Gorilla documentation

### Database Sources

- PostgreSQL documentation
- MySQL documentation
- Redis documentation
- GORM documentation
- SQLC documentation
- Ent documentation

### Security Sources

- OWASP API Security Top 10
- OAuth2 specification
- OpenID Connect documentation
- JWT documentation

### Testing Sources

- Go testing package
- Testify
- GoMock
- httptest
- Testcontainers Go

### Observability Sources

- OpenTelemetry Go
- Prometheus
- Grafana
- Jaeger

### Deployment Sources

- Docker documentation
- Kubernetes documentation
- GitHub Actions documentation
- Cloud provider documentation

---

## 6. Output Format

ให้ Markdown สุดท้ายมีหัวข้อหลักดังนี้:

```md
# Golang API Research Plan

## 1. Executive Summary

## 2. Existing Document Review

## 3. Research Gap Analysis

## 4. Complete Golang API Topic Map

## 5. API Architecture Research

## 6. Framework Comparison

## 7. Database & Persistence Research

## 8. Authentication & Security Research

## 9. Integration Research

## 10. Testing Strategy Research

## 11. Observability Research

## 12. Performance Research

## 13. Deployment & DevOps Research

## 14. Best Practice Summary

## 15. Production Checklist

## 16. Common Mistakes

## 17. Recommended Learning Path

## 18. Research Sources

## 19. Final Recommendation
```

---

## 7. Quality Rules

- ห้ามเขียนกว้าง ๆ แบบผิวเผิน
- ทุกหัวข้อต้องมีรายละเอียดที่นำไปใช้จริงได้
- ต้องมีตารางเปรียบเทียบเมื่อมีหลายตัวเลือก
- ต้องมี checklist สำหรับ production
- ต้องมี source ประกอบ
- ต้องระบุ gap จากเอกสารเดิม
- ต้องแยกสิ่งที่ควรใช้สำหรับ:
  - Small API
  - Medium API
  - Enterprise API
  - Microservice
  - High-load API

- ต้องมี recommendation ชัดเจน
- ต้องมี TODO สำหรับ Research ต่อ
- ต้องมี implementation roadmap

---

## 8. Final Output

ให้สร้างไฟล์ Markdown ที่พร้อมใช้งานจริง สามารถดำเนินการขั้นตอนข่อไปได้ มีตาราง มี checklist มี source และครอบคลุมทุก Topic สำคัญของ Go API Development, Integration, Security, Testing, Observability และ Deployment

```

```
