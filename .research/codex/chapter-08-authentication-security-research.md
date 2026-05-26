# Chapter 8. Authentication & Security Research

## 1. Executive Summary

Authentication (who are you?) and Authorization (what can you do?) represent the perimeter defense of any Go API. Security cannot be bolted on at the end of the development cycle; it must be designed into the HTTP middleware and database architecture from day one. This chapter details the production standards for securing Go APIs, covering stateless JWTs, stateful sessions, OAuth2, role-based access controls, and mitigations against the OWASP API Security Top 10. The ultimate recommendation is to leverage Go's strong standard library features (like `crypto/rand` and `crypto/subtle`) combined with strict middleware boundaries to enforce defense in depth.

## 2. Definitions and Key Terms

- **Authentication (AuthN):** The process of verifying an entity's identity (e.g., verifying an email and password).
- **Authorization (AuthZ):** The process of verifying that an authenticated entity has permission to perform a specific action.
- **JWT (JSON Web Token):** A stateless, cryptographically signed token often used as a Bearer token in HTTP headers.
- **RBAC (Role-Based Access Control):** Granting permissions based on an assigned role (e.g., "Admin", "User").
- **ABAC (Attribute-Based Access Control):** Granting permissions based on specific attributes of the user, resource, or environment (e.g., "User can edit document if User.ID == Document.OwnerID").
- **CSRF (Cross-Site Request Forgery):** An attack where a malicious site tricks a user's browser into executing unwanted actions on a trusted site where the user is authenticated.

## 3. Background and Context

Go’s explicit nature makes it well-suited for security programming. Unlike some frameworks that attempt to automatically magically bind users to requests (often leading to bypass vulnerabilities like mass-assignment), Go forces developers to explicitly parse tokens, check signatures, and manually attach identity to the `context.Context`.

However, the Go ecosystem moves fast, and older advice regarding password hashing (like using MD5 or SHA1) is dangerously obsolete. Today, enterprise systems require Argon2 or Bcrypt, secure cookie flags, and robust token lifecycle management (access/refresh rotation).

## 4. Core Concepts

Effective API security in Go relies on three core concepts:
1. **Defense in Depth:** Do not rely solely on the network perimeter (VPCs or Firewalls). Every HTTP handler must verify permissions independently.
2. **Context-Driven Identity:** Once a token is validated in middleware, the user's identity must be injected into the `context.Context`. The business logic layer should extract the identity from the context, ensuring the core domain never parses HTTP headers.
3. **Least Privilege:** API keys and database connections should only have the permissions necessary to perform their specific tasks.

## 5. Detailed Explanation of Authentication Models

### 5.1 JSON Web Tokens (JWT)
The de facto standard for modern REST APIs.
- **Flow:** The user logs in, the API returns a short-lived (e.g., 15 minutes) Access Token and a long-lived (e.g., 7 days) Refresh Token.
- **Implementation in Go:** Use `github.com/golang-jwt/jwt/v5`.
- **The Catch:** JWTs cannot be trivially invalidated before expiration without implementing a centralized blacklist (usually in Redis), which defeats the "stateless" benefit.

### 5.2 Session-Based Auth (Stateful)
The traditional method where a Session ID is stored in a secure cookie, and the server looks up the user in a database/Redis on every request.
- **Pros:** Instant revocation. If a user's account is compromised, you delete their session row in Redis, and they are instantly logged out.
- **Cons:** Requires a database lookup on every single API request, adding latency and requiring a scalable session store.

### 5.3 OAuth2 & OpenID Connect (OIDC)
When you want to offload identity management to Google, Apple, Auth0, or Keycloak.
- Go's `golang.org/x/oauth2` package is the industry standard for implementing these flows.

### 5.4 API Keys
Used for Machine-to-Machine (M2M) communication.
- Keys must be generated using `crypto/rand`.
- **CRITICAL:** Never store plain-text API keys in the database. Store a fast cryptographic hash (like SHA-256) of the key. When a request comes in, hash the provided key and compare the hashes using `crypto/subtle.ConstantTimeCompare` to prevent timing attacks.

## 6. Step-by-Step Process for Securing an Endpoint

1. **Request Arrives:** The request hits a Rate Limiter middleware.
2. **Authentication Middleware:** Extracts the `Authorization: Bearer <token>` header.
3. **Validation:** Cryptographically verifies the JWT signature and checks the expiration (`exp`) claim.
4. **Context Injection:** Creates a custom `UserClaims` struct and attaches it to `r.Context()`.
5. **Authorization Middleware:** Checks if the injected user has the required Role (RBAC) for the route.
6. **Business Logic:** The handler extracts the user from the context and passes it to the Use Case layer to enforce ABAC (e.g., "Does this user own this resource?").

## 7. Practical Examples

**Context Injection Middleware:**
```go
type contextKey string
const userContextKey = contextKey("userClaims")

func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        tokenString := extractToken(r) // helper function
        claims, err := verifyJWT(tokenString)
        if err != nil {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        
        // Inject into context
        ctx := context.WithValue(r.Context(), userContextKey, claims)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

## 8. Real-World Use Cases

- **Consumer Mobile App:** Use JWTs with Refresh Token rotation. Mobile connections drop frequently, so checking a Redis session on every request adds too much latency.
- **Internal Financial Dashboard:** Use strict Session-Based Auth with `HttpOnly` cookies. If an employee is terminated, their access must be revokable in milliseconds.
- **B2B SaaS:** Issue scoped API Keys to partner companies, ensuring the keys have read-only access to specific resources.

## 9. Comparisons and Alternatives

| Model | Strengths | Weaknesses | Go Tooling |
| ----- | --------- | ---------- | ---------- |
| **JWT** | Stateless, fast validation | Hard to revoke instantly | `golang-jwt/jwt/v5` |
| **Sessions** | Instant revocation | Database hit per request | `gorilla/sessions`, Redis |
| **OIDC** | No passwords to store | Complex protocol dance | `golang.org/x/oauth2` |
| **API Keys** | Easy for developers | Prone to being leaked in git | `crypto/subtle` |

## 10. Benefits and Advantages of Go's Approach

Go forces you to handle errors explicitly. In security programming, unhandled exceptions are vulnerabilities. Go's lack of "magic" means developers are explicitly aware of when a token is being parsed and when a password is being hashed, reducing the surface area for logic-bypass exploits.

## 11. Limitations, Risks, and Tradeoffs

- **JWT Size:** JWTs can become massive if you stuff too many roles and permissions into the payload. This inflates HTTP header sizes, which can cause load balancers (like AWS ALB) to drop requests.
- **Refresh Token Theft:** If a refresh token is stolen, an attacker can generate infinite access tokens. Implementing "Refresh Token Rotation" (invalidating the token family if an old refresh token is reused) is necessary but complex to build.

## 12. Edge Cases

- **Timing Attacks:** Comparing strings using `==` for API keys or passwords allows an attacker to guess the string based on how many microseconds the comparison took. Always use `crypto/subtle.ConstantTimeCompare`.
- **CORS vs Credentials:** If your API requires credentials (cookies/Auth headers) from a browser frontend, you cannot use a wildcard `Access-Control-Allow-Origin: *`. You must echo back the specific exact origin of the request.

## 13. Common Mistakes and Misconceptions

- **Mistake:** Logging the raw `*http.Request` object, which accidentally writes Authorization headers and passwords in plain text to Datadog/CloudWatch.
- **Mistake:** Using `bcrypt` but setting the cost factor too low (e.g., default 10 instead of 12-14 in 2026), making passwords vulnerable to GPU cracking.
- **Misconception:** "JWTs are encrypted." No, they are base64 encoded and *signed*. Anyone who intercepts a JWT can read the payload. Never put PII (Personally Identifiable Information) inside a JWT.

## 14. Best Practices (OWASP Mitigation)

- **Limit Payload Sizes:** Use `http.MaxBytesReader` to prevent denial-of-service via massive JSON payloads.
- **Secure Headers:** Use middleware (like `unrolled/secure`) to inject `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, and `Content-Security-Policy`.
- **Rate Limiting:** Implement token-bucket rate limiting (e.g., `golang.org/x/time/rate`) keyed by User ID or IP address to prevent brute forcing.

## 15. Open Questions and Further Research

- What is the organization's threshold for transitioning from standard `bcrypt` to `argon2id` for password hashing, considering Argon2's memory-hardness provides better defense against specialized hardware attacks?
- How to implement a robust ABAC engine in Go (e.g., using Open Policy Agent/Rego) when RBAC rules become too complex?

## 16. References or Source Notes

- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [RFC 7519: JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519)
- Go `crypto/subtle` documentation.

## 17. Chapter Summary

Securing a Go API is a multi-layered discipline. It requires choosing the correct authentication model (JWT vs Sessions) based on revocation needs, strictly validating input sizes to prevent DoS, using constant-time comparisons for secrets, and injecting identity into the `context.Context` to keep the domain logic clean. By treating security as a foundational architectural requirement rather than an afterthought, Go APIs can easily meet the highest enterprise compliance standards.
