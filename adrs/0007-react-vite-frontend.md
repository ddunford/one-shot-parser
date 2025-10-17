# ADR-0007: React with Vite Frontend

**Status**: Accepted
**Date**: 2025-10-16

## Context

Need modern frontend framework with excellent DX for SPA.

## Decision

Use **React + Vite** for frontend.

**React**: Component-based, mature, TypeScript support
**Vite**: Lightning-fast HMR, modern build tool

## Rationale

✅ React: Mature, large ecosystem, team familiarity
✅ Vite: Fastest HMR (instant feedback), ESM-based, modern
✅ TypeScript: Type safety across stack
✅ Simple state management: Context API sufficient (no Redux)

## Alternatives Considered

- **Next.js**: Overkill (no SSR needed, stateless)
- **Vue**: Team less familiar
- **Svelte**: Smaller ecosystem
- **CRA**: Slower than Vite, outdated

## Consequences

✅ **Fast Development**: Sub-second hot reload
✅ **Modern Tooling**: ESM, optimized builds
✅ **Great DX**: Clear errors, fast feedback
❌ **Build Complexity**: Vite config for production
❌ **No SSR**: Not needed, but can't add easily later

## Vite 5 Reverse Proxy Configuration

**CRITICAL**: Vite 5+ has DNS rebinding attack protection that validates Host headers. When running behind a reverse proxy (Traefik), you MUST configure `server.allowedHosts`:

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    host: '0.0.0.0',
    port: 3000,
    allowedHosts: [
      'cvparse.demosrv.uk',  // Production domain
      'localhost',            // Local development
      '.demosrv.uk',          // Allow all subdomains (optional)
    ],
  },
})
```

**Without this**: Traefik requests will be blocked with "This host is not allowed"

**Security Note**: Only add trusted domains to prevent DNS rebinding attacks.

## References

- ARCHITECTURE.md § Frontend Components

**Related ADRs**: ADR-0005 (Multi-Stage Docker)
