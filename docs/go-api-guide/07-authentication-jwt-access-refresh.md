# Authentication Guide: JWT Access & Refresh Tokens

This guide details how to implement a secure, professional authentication system using **JWT (JSON Web Tokens)** with an **Access + Refresh Token** pattern for a Go backend and a Vue/Nuxt monorepo.

---

## 1. Backend Implementation (Go)

We use the `golang-jwt/jwt` library for token management.

### Token Types
1.  **Access Token:** Short-lived (e.g., 15 mins). Used for API authorization.
2.  **Refresh Token:** Long-lived (e.g., 7 days). Used to obtain new access tokens.

### Go Middleware
The middleware intercepts requests to protected routes and verifies the `Authorization` header.

```go
func JWTMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		authHeader := r.Header.Get("Authorization")
		if authHeader == "" {
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}

		tokenString := strings.TrimPrefix(authHeader, "Bearer ")
		token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
			return []byte(os.Getenv("JWT_SECRET")), nil
		})

		if err != nil || !token.Valid {
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}

		next.ServeHTTP(w, r)
	})
}
```

---

## 2. Frontend Implementation (Shared Core)

Logic should live in `packages/core` to be shared by both the Backoffice and Storefront.

### Pinia Auth Store (`packages/core/store/auth.ts`)
```typescript
export const useAuthStore = defineStore('auth', {
  state: () => ({
    user: null,
    accessToken: localStorage.getItem('access_token'),
    refreshToken: localStorage.getItem('refresh_token'),
  }),
  actions: {
    setTokens(access: string, refresh: string) {
      this.accessToken = access;
      this.refreshToken = refresh;
      localStorage.setItem('access_token', access);
      localStorage.setItem('refresh_token', refresh);
    },
    logout() {
      this.user = null;
      this.accessToken = null;
      this.refreshToken = null;
      localStorage.clear();
    }
  }
});
```

### Axios Interceptor (Auto-Refresh)
This logic automatically handles token expiry without interrupting the user.

```typescript
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    const authStore = useAuthStore();

    if (error.response.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      try {
        const { data } = await axios.post('/auth/refresh', {
          refresh_token: authStore.refreshToken,
        });
        authStore.setTokens(data.access_token, data.refresh_token);
        originalRequest.headers['Authorization'] = `Bearer ${data.access_token}`;
        return api(originalRequest);
      } catch (refreshError) {
        authStore.logout();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    return Promise.reject(error);
  }
);
```

---

## 3. Nuxt SSR Specifics

Since Local Storage is not available on the server during the initial SSR load, we must sync the token.

### Handshake Strategy
1.  **Client-side:** On login, store the `access_token` in a **non-HttpOnly cookie** (e.g., `ssr_token`).
2.  **Server-side (Nuxt):** During `fetch` or `useAsyncData`, the Nuxt server reads the `ssr_token` cookie and adds it to the backend request header.

```typescript
// apps/storefront/plugins/auth-sync.ts
export default defineNuxtPlugin(() => {
  const token = useCookie('ssr_token');
  if (token.value) {
    // Inject token into your API client headers
  }
});
```

---

## 4. Integration Roadmap

1.  **Backend:** Implement `/login` and `/refresh` handlers.
2.  **Packages/Core:** Create the Pinia store and shared API client with interceptors.
3.  **Apps:** Import the shared auth store and protect routes using **Middlewares** (Vue Router or Nuxt Middleware).
4.  **Security:** Ensure `JWT_SECRET` is set as an environment variable in Staging, UAT, and Production.
