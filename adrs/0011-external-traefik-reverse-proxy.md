# ADR-0011: External Traefik Reverse Proxy

**Status**: Accepted
**Date**: 2025-10-16

## Context

Server already runs Traefik in external Docker network (traefik_demosrv). Need to integrate CV Parser without creating duplicate proxy.

## Decision

**Use existing external Traefik** - DO NOT create Traefik container.

**Integration**:
- Services join `traefik_demosrv` external network
- Traefik labels on services for routing
- Frontend: `Host(\`cvparse.demosrv.uk\`)`
- Backend: `Host(\`cvparse.demosrv.uk\`) && PathPrefix(\`/api\`)`
- Let's Encrypt certs handled by existing Traefik

## Consequences

✅ **No Duplication**: Single Traefik for all projects
✅ **Automatic HTTPS**: Let's Encrypt already configured
✅ **Centralized Routing**: Easier to manage
✅ **Resource Efficient**: One proxy instead of many
❌ **External Dependency**: Traefik must be running
❌ **Shared Configuration**: Can't modify Traefik independently

## Configuration

```yaml
networks:
  traefik_demosrv:
    external: true  # CRITICAL: Don't create, use existing

services:
  backend:
    networks:
      - traefik_demosrv
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.cvparse-api.rule=Host(`cvparse.demosrv.uk`) && PathPrefix(`/api`)"
      - "traefik.http.services.cvparse-api.loadbalancer.server.port=3001"
      - "traefik.http.services.cvparse-api.loadbalancer.healthcheck.path=/api/health"
```

## Frontend Vite Configuration

**CRITICAL**: Vite 5+ requires explicit `allowedHosts` for reverse proxy:

```typescript
// src/frontend/vite.config.ts
export default defineConfig({
  server: {
    host: '0.0.0.0',
    port: 3000,
    allowedHosts: [
      'cvparse.demosrv.uk',  // REQUIRED for Traefik
    ],
  },
})
```

**Without this**: Requests will be blocked with "This host is not allowed"

## References

- DEPLOYMENT.md § Infrastructure Architecture
- BUILD_SPEC.md § Docker Configuration

**Related ADRs**: ADR-0005 (Multi-Stage Docker)
