# ADR-0005: Multi-Stage Docker with Hot Reload

**Status**: Accepted
**Date**: 2025-10-16

## Context

Development requires hot reload for fast iteration. Production requires optimized builds. Need single Dockerfile supporting both.

## Decision

Use multi-stage Dockerfiles with `development` and `production` targets:
- **Development**: Volume mounts, dev dependencies, nodemon/Vite HMR
- **Production**: Compiled code, minimal image, no volume mounts

**Build targets**:
```bash
# Development (default)
docker build --target development -t cvparse:dev .

# Production
docker build --target production -t cvparse:prod .
```

## Consequences

✅ **Fast Development**: Hot reload on file changes (<1s)
✅ **Optimized Production**: Small images, fast startup
✅ **Single Dockerfile**: Maintain one file for both environments
✅ **Docker Best Practices**: Non-root user, health checks
❌ **Build Complexity**: Multiple stages require understanding
❌ **Image Size**: Development images are larger

## References

- DOCKER_BEST_PRACTICES.md
- DEPLOYMENT.md § Docker Configuration

**Related ADRs**: ADR-0011 (External Traefik)
