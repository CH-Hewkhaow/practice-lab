# Frontend Monorepo Integration (Vue, Nuxt, SSE, WebSockets)

This guide outlines how to integrate a **Vue 3 Backoffice** and a **Nuxt 3 Storefront** into your Go ecosystem using a monorepo structure. We use **PrimeVue + Tailwind CSS** for a flexible UI and a hybrid **SSE + WebSocket** strategy for real-time updates.

---

## 1. Monorepo Architecture (pnpm Workspaces)

A monorepo allows sharing logic and types while keeping the applications independent.

### Directory Structure
```text
.
├── apps/
│   ├── backoffice/          # Vue 3 + Vite + PrimeVue
│   └── storefront/          # Nuxt 3 (SSR) + PrimeVue
├── packages/
│   ├── ui/                  # Shared PrimeVue/Tailwind base configurations
│   └── core/                # Shared API clients, SSE/WS composables, & Types
├── pnpm-workspace.yaml
└── docker-compose.yml
```

---

## 2. Mini-Module Architecture

To keep the frontend maintainable as it grows, we use a **Feature-Based (Mini-Module)** structure. Each module is a self-contained unit of functionality.

### Internal App Structure (`apps/backoffice` & `apps/storefront`)
```text
src/
├── modules/ (or features/)
│   ├── auth/
│   │   ├── components/      # Feature-specific components
│   │   ├── store/           # Pinia store for this module
│   │   ├── styles/          # Module-specific styles
│   │   ├── composables/     # Logic specific to this feature
│   │   └── index.ts         # Public API for the module
│   └── dashboard/
│       ├── components/
│       └── ...
├── shared/                  # Common resources used across modules
│   ├── components/          # "Atoms" (Buttons, Inputs, Spinners)
│   ├── composables/         # Global logic (real-time, auth guards)
│   ├── assets/              # Global styles, fonts, images
│   └── types/               # Shared TypeScript interfaces
└── App.vue
```

### Key Principles:
1.  **Encapsulation:** A module should not reach deep into another module's `components` or `store`. Use the `index.ts` as a gatekeeper.
2.  **Shared vs. Module:** If a component is used by 3+ modules, move it to `shared/components`.
3.  **Atomic Design:** The `shared/components` folder should contain your "Atoms" (base elements) and "Molecules" (small groups), while `modules/*/components` contain "Organisms" (complex feature blocks).

---

## 3. Shared UI Strategy (PrimeVue + Tailwind)

To maintain flexibility while using the same library, we centralize the **Tailwind configuration** and **PrimeVue presets**, but allow each app to configure its own theme.

### Recommended Tooling
- **PrimeVue (Unstyled Mode):** Allows Tailwind to drive the styling of components.
- **Tailwind CSS:** Shared configuration for colors, spacing, and typography.

---

## 3. Real-time Hybrid System

We use both protocols to optimize for different use cases.

### A. Server-Sent Events (SSE)
- **Use Case:** One-way notifications (Order status, Stock alerts, Analytics).
- **Implementation:** Go backend streams events over HTTP/2.

### B. WebSockets
- **Use Case:** Bidirectional interaction (Live chat, Admin commands, Collaborative editing).
- **Implementation:** Using `gorilla/websocket` or `centrifuge` in Go, and a shared `useSocket` composable in the frontend.

### Shared Composable (`packages/core/composables/useRealtime.ts`)
```typescript
export const useRealtime = () => {
  // SSE for Notifications
  const initSSE = (url: string) => {
    const eventSource = new EventSource(url);
    eventSource.onmessage = (event) => console.log("SSE Update:", event.data);
    return eventSource;
  };

  // WebSocket for Commands
  const initWS = (url: string) => {
    const socket = new WebSocket(url);
    socket.onmessage = (event) => console.log("WS Message:", event.data);
    return socket;
  };

  return { initSSE, initWS };
};
```

---

## 4. Nuxt SSR Integration

The Nuxt Storefront uses SSR for SEO and initial load performance, while the Backoffice is a pure SPA for speed.

### Nuxt Configuration (`apps/storefront/nuxt.config.ts`)
```typescript
export default defineNuxtConfig({
  modules: ['@primevue/nuxt-module'],
  primevue: {
    usePrimeVue: true,
    options: { unstyled: true }, // Drive styling via Tailwind
  },
  css: ['~/assets/css/main.css', 'tailwind/base.css'],
  // Shared logic from monorepo
  alias: {
    '@core': '../../packages/core',
    '@ui': '../../packages/ui',
  }
})
```

---

## 5. Deployment with Docker

Each app is containerized independently but can share a base image.

### Orchestration (`docker-compose.yml`)
```yaml
services:
  api:
    build: ./go-app
    # ... existing backend config

  backoffice:
    build: ./apps/backoffice
    ports: ["3000:3000"]
    depends_on: [api]

  storefront:
    build: ./apps/storefront
    ports: ["3001:3000"]
    environment:
      - NUXT_PUBLIC_API_BASE=http://api:8080
    depends_on: [api]
```

---

## 6. Integration Roadmap

1.  **Init Workspace:** Run `pnpm init` and create `pnpm-workspace.yaml`.
2.  **Setup Packages:** Create `packages/core` for API logic and `packages/ui` for Tailwind configs.
3.  **Bootstrap Apps:**
    - `pnpm create vite apps/backoffice --template vue-ts`
    - `npx nuxi init apps/storefront`
4.  **Add PrimeVue:** Install `@primevue/nuxt-module` in Nuxt and `primevue` in the Vue app.
5.  **Hook up Real-time:** Implement the Go SSE/WS endpoints and connect the frontend composables.
